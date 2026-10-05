---
layout: post
title:  "Orca 업데이트 후 터미널에서 로컬 네트워크(LAN) 접속이 끊기는 문제 (macOS)"
categories: orca macos
---

> **요약**
> macOS에서 Orca를 업데이트하면, Orca 터미널에서 실행한 Codex CLI 같은 도구가 LAN 장비에 접속하지 못하는 문제가 생깁니다. 시스템 설정의 로컬 네트워크 권한은 여전히 켜져 있는데도 그렇습니다.
> 원인은 **업데이트 후에도 살아남은 이전 버전의 터미널 데몬**으로 보입니다. 열어 둔 터미널은 이 옛 데몬 아래에서 돌기 때문에, macOS가 새 Orca의 권한을 적용하지 못합니다.
> 대부분은 **Settings → Terminal → Manage Sessions → Restart**로 터미널 데몬을 다시 띄우면 해결됩니다. 이 글에서는 GitHub 이슈에 올라온 내용을 바탕으로 증상, 원인, 해결 방법을 정리합니다.

---

## 1. 어떤 문제였나

Orca에서 터미널을 열고 Codex CLI 같은 에이전트를 돌리면서, 같은 네트워크에 있는 서버(예: `192.168.x.x`)에 접속하는 일이 많습니다. 그런데 저도 Orca를 업데이트한 뒤 이 접속이 갑자기 막혔습니다.

증상은 다음과 같습니다.

- Orca 터미널에서 실행한 도구가 LAN 주소에 접속하지 못합니다. 보통 `No route to host`(`EHOSTUNREACH`, `[Errno 65]`) 에러가 납니다.
- 막히는 것은 LAN 주소뿐이고, 인터넷 주소는 정상입니다.
- `System Settings → Privacy & Security → Local Network`에서 Orca는 여전히 **켜져 있습니다.**
- 같은 명령을 Terminal.app이나 iTerm2에서 실행하면 잘 됩니다.
- Orca를 업데이트할 때마다 반복됩니다.

<!-- 📷 스크린샷: 접속 실패 에러 메시지 -->

권한은 켜져 있고 다른 터미널에서는 잘 되니, 증상만 보고는 원인을 짐작하기 어렵습니다.

---

## 2. 나만 겪은 문제가 아니었다: Orca 이슈 #21988

GitHub에 같은 증상의 이슈가 올라와 있습니다.

- [stablyai/orca#21988](https://github.com/stablyai/orca/issues/21988): *Local Network access breaks after every Orca update on macOS 27 until permission is toggled off/on*

원 작성자는 macOS 27, Orca 1.4.206에서 이 문제를 겪었고, 업데이트할 때마다 로컬 네트워크 권한을 껐다 켜는 방법으로 버텼다고 합니다. 이후 댓글로 비슷한 사례가 이어졌는데, 해결 방법이 사람마다 조금씩 다릅니다.

| 보고 | 환경 | 권한 껐다 켜기 | 실제로 해결된 방법 |
| --- | --- | --- | --- |
| 원 작성자 | macOS 27, Orca 1.4.206 | 효과 있음 | 권한 껐다 켜기 |
| 댓글 A | macOS 27.0, 1.4.205 → 1.4.206 | 효과 없음 | Manage Sessions → Restart |
| 댓글 B | macOS 27.0, Orca 1.4.212 | - | Orca 본체와 터미널 데몬 재시작 |
| 댓글 C | macOS 26.6.2, Orca 1.4.218 | 효과 없음 | 앱을 다시 실행해도 안 됨 (우회만 가능) |

Orca 개발자는 댓글에서 원인을 이렇게 추정했습니다. 터미널 데몬은 업데이트 후에도 살아남는데(그래서 터미널도 유지되는데), 업데이터가 옛 앱 번들을 다른 곳으로 옮겨 두기 때문에 데몬과 그 데몬이 띄운 Codex가 옛 사본에서 계속 실행된다는 것입니다. 업데이트 후 이 상황을 감지하고 새 터미널을 새 데몬으로 보내는 수정도 준비 중이라고 합니다.

비슷한 문제를 다룬 [stablyai/orca#20007](https://github.com/stablyai/orca/issues/20007) 이슈와 [stablyai/orca#21826](https://github.com/stablyai/orca/pull/21826) PR도 있는데, 2026년 10월 5일 기준으로 둘 다 아직 열려 있습니다.

---

## 3. 원인: 업데이트 후에도 살아남은 터미널 데몬

Orca는 터미널 세션을 앱 본체가 아니라 별도의 **터미널 데몬**(`daemon-entry.js`를 실행하는 `Orca Helper` 프로세스)에서 관리합니다. 덕분에 Orca를 재시작하거나 업데이트해도 열어 둔 터미널과 에이전트 세션이 끊기지 않습니다.

문제는 업데이트 과정에서 생깁니다. macOS용 Orca는 Squirrel(ShipIt)로 업데이트되는데, 이때 기존 `Orca.app`을 임시 폴더(`.../T/com.stablyai.orca.ShipIt.XXXXXXXX/`)로 옮기고 새 번들을 `/Applications`에 설치합니다. 임시 폴더는 나중에 지워지지만, 이미 실행 중이던 데몬은 **지워진 옛 실행 파일을 그대로 붙잡은 채** 계속 돌아갑니다.

macOS의 로컬 네트워크 권한은 연결을 시도한 프로세스의 **책임 프로세스(responsible process)** 를 기준으로 판단합니다. 스레드의 댓글 B가 이 부분을 직접 확인했습니다.

- 새 데몬이 띄운 셸: 책임 프로세스가 **현재 Orca 본체**라서 Orca의 권한이 적용됩니다.
- 옛 데몬이 띄운 셸: 책임 프로세스가 **자기 자신**이라서 Orca의 권한이 적용되지 않습니다.

그림으로 정리하면 이렇습니다.

```text
업데이트 전
Orca (이전 버전) ─ 터미널 데몬 (이전 버전) ─ zsh ─ codex        → LAN 접속 ✅

업데이트 후
Orca (새 버전, /Applications 에 새로 설치된 번들)
터미널 데몬 (이전 버전, 지워진 옛 번들에서 실행 중) ─ zsh ─ codex  → LAN 접속 ❌
  ↑ 업데이트 전에 열어 둔 터미널은 전부 여기에 붙어 있음
```

일부 보고에서는 같은 셸 안에서도 `/usr/bin/nc`, `ping` 같은 Apple 기본 도구는 LAN에 접속되고 `kubectl`, `node`, Homebrew `python3`만 막혔다고 합니다. 그래서 "어떤 명령은 되는데 어떤 명령은 안 되는" 헷갈리는 상황이 생길 수 있습니다.

권한을 껐다 켜는 방법이 어떤 사람에게는 통하고 어떤 사람에게는 통하지 않는 이유는 아직 명확하지 않습니다.

---

## 4. 해결 방법

### 먼저 확인하기

이슈에서 Orca 개발자가 권한을 건드리기 전에 실행해 보라고 요청한 진단 명령입니다.

```bash
lsof -p "$(pgrep -f daemon-entry.js | head -1)" | awk '$4=="txt"' | head -2
```

- `pgrep -f daemon-entry.js`로 터미널 데몬의 PID를 찾고, `lsof`로 그 데몬이 실제로 실행 중인 파일(`txt`)을 보여 줍니다.
- 경로가 `/Applications/Orca.app/...`이면 정상입니다.
- 경로가 `.../T/com.stablyai.orca.ShipIt.XXXXXXXX/Orca.app/...` 같은 임시 폴더라면, 업데이트 전 옛 번들에서 돌고 있는 데몬입니다. 댓글 A의 결과가 이 경우였습니다.

```text
Orca\x20H 66608 <user>  txt  REG  1,15  223248  111784381 /private/var/folders/zc/.../T/com.stablyai.orca.ShipIt.E8KqbPIH/Orca.app/Contents/Frameworks/Orca Helper.app/Contents/MacOS/Orca Helper
```

참고할 점이 두 가지 있습니다.

- 댓글 A의 환경에서는 `pgrep -f daemon-entry.js`가 아무것도 찾지 못해서, PID를 따로 찾아 넣었다고 합니다.
- `head -1`은 첫 번째 데몬 하나만 확인합니다. `pgrep -f daemon-entry.js`만 실행했을 때 PID가 여러 개 나오면 각각 확인해 보세요.

### 방법 1. 터미널 세션 재시작 (권장)

Orca에서 **Settings → Terminal → Manage Sessions → Restart**를 누릅니다.

- 옛 데몬이 정리되고, 터미널이 새 버전의 데몬에서 다시 뜹니다.
- ⚠️ **열려 있는 터미널이 모두 닫힙니다.** 돌고 있는 에이전트 작업이 있다면 끝난 뒤에 하세요.
- 스레드에서는 권한을 껐다 켜도 안 고쳐졌던 사람(댓글 A)이 이 방법으로 바로 해결했습니다.

### 방법 2. 로컬 네트워크 권한 껐다 켜기

`System Settings → Privacy & Security → Local Network`에서 Orca를 껐다가 다시 켭니다. 원 작성자는 이 방법으로 바로 복구됐지만, 효과가 없었다는 댓글도 있습니다. 목록에 Orca가 두 개 이상 보인다면 모두 껐다 켜 보세요.

### 방법 3. 그래도 안 될 때

- Orca를 완전히 종료(`⌘ + Q`)했다가 다시 실행해도, 터미널 데몬은 앱과 별개로 살아남을 수 있어서 효과가 없을 수 있습니다.
- 댓글 C처럼 위 방법이 모두 통하지 않는 경우도 있습니다. 스레드에서는 Apple 기술 문서 TN3179를 근거로 복구 모드에서 시스템 설정 파일을 지우는 방법까지 언급되지만, 시스템 파일을 건드리는 일이라 권하지 않습니다. 그럴 때는 LAN 작업만 Terminal.app에서 실행하면서 Orca의 수정 버전을 기다리는 편이 안전합니다.

<!-- ✍️ 직접 적용한 방법과 결과 -->

---

## 5. 마치며

업데이트할 때마다 LAN 접속이 끊기는 건 사소해 보여도, 터미널에 에이전트를 띄워 두고 일하는 입장에서는 꽤 번거로운 문제입니다. 원인을 알고 나면 대처는 간단합니다. 업데이트 후 LAN 접속이 안 되면 터미널 세션부터 재시작해 보면 됩니다. Orca 쪽에서도 업데이트 후 오래된 데몬을 감지하는 수정을 준비 중이라고 하니, 그 전까지는 이 방법으로 대응하면 됩니다.

### 참고 링크

- 이슈 #21988: <https://github.com/stablyai/orca/issues/21988>
- 관련 이슈 #20007: <https://github.com/stablyai/orca/issues/20007>
- 관련 PR #21826: <https://github.com/stablyai/orca/pull/21826>
- Apple TN3179, Understanding local network privacy: <https://developer.apple.com/documentation/technotes/tn3179-understanding-local-network-privacy>

---

> ✨ **Crafted with Claude Opus 5.5**
>
> *직접 겪은 불편함에서 출발해, AI와 함께 정리한 기록입니다.*
> 이 글은 Anthropic의 **Claude Opus 5.5**의 도움을 받아 작성되었습니다.
> 오류는 제가 직접 겪은 것이고, Claude는 이슈 스레드 조사와 글 정리를 도왔습니다.
