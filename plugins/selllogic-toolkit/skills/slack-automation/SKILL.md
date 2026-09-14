---
name: slack-automation
description: 슬랙을 코드로 읽고 쓸 때 사용. 봇 토큰 발급, 메시지 전송·조회, 스레드·파일 첨부, 실패 판정과 흔한 함정을 담는다. "슬랙에 자동으로 올려줘", "채널 내용 읽어서 정리해줘", "슬랙 봇 만들기", "알림 자동화" 요청 시 사용.
---

# 슬랙 자동화

**DOM 을 긁지 않는다.** 슬랙은 공식 API 가 충분히 열려 있다. 화면을 긁으면 UI 가 바뀔 때마다 깨진다.

## 1. 봇 토큰 받기

api.slack.com → Your Apps → Create New App → From scratch → 워크스페이스 선택.

OAuth & Permissions 에서 **Bot Token Scopes** 를 추가한 뒤 Install to Workspace.

| 스코프 | 무엇이 가능해지나 |
|---|---|
| `chat:write` | 메시지 보내기 |
| `channels:history` | 공개 채널 읽기 |
| `groups:history` | 비공개 채널 읽기(초대된 곳만) |
| `files:write` | 파일 올리기 |
| `reactions:write` | 이모지 달기 |
| `users:read` | 사용자 이름 조회 |

설치하면 `xoxb-` 로 시작하는 봇 토큰이 나온다. **이게 전부다.** 스코프를 나중에 추가하면
반드시 재설치해야 적용된다.

> 스코프는 필요한 것만 준다. `channels:history` 하나로도 그 워크스페이스의 모든 공개 대화를 읽는다.

## 2. 기본 호출

```python
import os, requests

TOKEN = os.environ["SLACK_BOT_TOKEN"]      # 코드에 박지 않는다
H = {"Authorization": f"Bearer {TOKEN}"}

def api(method, **params):
    r = requests.post(f"https://slack.com/api/{method}", headers=H, data=params).json()
    if not r.get("ok"):
        raise RuntimeError(f"{method} 실패: {r.get('error')}")   # 조용히 넘기지 않는다
    return r

api("chat.postMessage", channel="C0123456789", text="배포 완료")
```

`ok: false` 를 무시하면 **에러 없이 아무것도 안 보내진 채로 성공처럼 보인다.** 슬랙 API 는
HTTP 200 에 실패를 담아 돌려준다. 반드시 `ok` 를 본다.

## 3. 자주 쓰는 것

| 하려는 일 | 메서드 | 비고 |
|---|---|---|
| 메시지 보내기 | `chat.postMessage` | `thread_ts` 를 주면 스레드 답글 |
| 채널 읽기 | `conversations.history` | `oldest`/`latest` 는 유닉스 초 |
| 스레드 읽기 | `conversations.replies` | 봇 답변은 대개 여기 달린다 |
| 이모지 달기 | `reactions.add` | `name` 은 콜론 없이 (`white_check_mark`) |
| 사람 이름 | `users.info` | 표시명과 실명이 다를 수 있다 |
| 링크 만들기 | `chat.getPermalink` | 기록에 남길 땐 링크로 |

### 파일 올리기 (3단계)

옛 `files.upload` 는 폐기됐다. 지금은 세 번 호출한다.

1. `files.getUploadURLExternal` — 업로드 URL 과 파일 id 를 받는다
2. 그 URL 로 파일 바이트를 POST
3. `files.completeUploadExternal` — 채널에 게시

## 4. 함정

| 증상 | 원인·해법 |
|---|---|
| `channel_not_found` (비공개 채널) | 봇이 초대되지 않았다. 사람이 `/invite @봇` |
| `not_in_channel` | 공개 채널이어도 초대가 필요한 경우가 있다 |
| `conversations.history` 가 **에러 없이 0건** | `oldest` 에 소수점 붙은 문자열을 넣었다 → `str(int(ts))` |
| `ratelimited` | 채널 여러 개를 짧은 간격으로 돌면 분당 한도를 넘는다. 간격을 늘리거나 이벤트 방식으로 |
| 90일 전 메시지가 안 보임 | 무료 플랜의 보관 한도. **로그로 퍼널을 판정하면 틀린다** |
| 멘션이 글자로만 나감 | `<@U0123456789>` 형식이어야 한다. `@이름` 은 그냥 텍스트 |
| 이모지 반응이 안 옴 | 이벤트 구독 목록에 `reaction_added` 가 없으면 아예 발화하지 않는다 |

## 5. 성공 판정

**반환값이 아니라 채널 상태로 판정한다.**

```python
ts = api("chat.postMessage", channel=CH, text=body)["ts"]
back = api("conversations.history", channel=CH, latest=ts, inclusive="true", limit=1)
assert back["messages"][0]["text"] == body
```

"보냈다"는 응답을 받고도 실제로는 안 올라간 경우가 있다. 중요한 발송일수록 되읽어 확인한다.

## 6. 규칙

| 규칙 | 이유 |
|---|---|
| 토큰은 환경변수로 | 커밋되면 워크스페이스 전체가 열린다. 유출 시 즉시 재발급 |
| 테스트는 테스트 채널로 | 운영 알림 채널에 쏘면 사람들이 알림을 끈다 |
| 자동 삭제는 **내가 만든 ts** 로만 | 제목·표식으로 지우면 남의 글이 지워진다 |
| 실패는 반드시 남긴다 | 로그 + 알림 1회. `except: pass` 는 사고의 씨앗 |
| 반복 발송에는 중복 가드 | 같은 건을 두 번 처리하지 않게 (채널, ts) 를 기억한다 |
