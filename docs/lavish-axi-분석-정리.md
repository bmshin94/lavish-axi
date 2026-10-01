# lavish-axi 전수조사 분석 & 활용 전략 정리

> 이 문서는 `lavish-axi` 저장소를 전수조사하여 "무엇을 하는 도구인지, 어떻게 쓰는지, 어떻게 활용·수익화할 수 있는지"를 정리한 한국어 분석 노트입니다.
> 작성 시점: 2026-10-01 / 분석 대상 버전: `0.1.71`

## 관련 링크

| 구분                    | URL                                              |
| ----------------------- | ------------------------------------------------ |
| 원본(업스트림) 저장소   | https://github.com/kunchenguid/lavish-axi        |
| 포크 저장소 (이 저장소) | https://github.com/bmshin94/lavish-axi           |
| npm 패키지              | https://www.npmjs.com/package/lavish-axi         |
| 이슈 트래커             | https://github.com/kunchenguid/lavish-axi/issues |
| AXI 규격                | https://axi.md                                   |
| Agent Skills            | https://agentskills.io                           |
| Agent Plugins           | https://agent-plugins.org                        |
| 공유 호스트(서드파티)   | https://ht-ml.app                                |
| Discord                 | https://discord.gg/Wsy2NpnZDu                    |
| 제작자 X                | https://x.com/kunchenguid                        |

---

## 1. 프로젝트 정체

**lavish-axi (Lavish Editor)** 는 AI 에이전트가 생성한 HTML 결과물(artifact)을 로컬 브라우저에서 열어, 사람이 요소를 클릭하거나 텍스트를 선택하거나 다이어그램에 직접 그려서 피드백을 남기고, 그 피드백을 롱폴링 API로 다시 에이전트에게 전달하는 **로컬 CLI + 로컬 HTTP 서버** 도구다.

슬로건: **"HTML is the new markdown. Lavish is the new editor for your HTML artifacts."**

| 항목              | 값                                                                                                                             |
| ----------------- | ------------------------------------------------------------------------------------------------------------------------------ |
| 버전              | 0.1.71                                                                                                                         |
| 라이선스          | MIT (상업적 이용 가능)                                                                                                         |
| 런타임            | Node 22+ , ESM-only JavaScript (TypeScript 소스 없음, `checkJs`로 타입 검증)                                                   |
| 주요 의존성       | express 5, ws, chokidar, parse5, open, cross-spawn, axi-sdk-js, @tailwindcss/browser, daisyui                                  |
| 개발 의존성       | esbuild, eslint, prettier, typescript, mermaid(정확 고정), @excalidraw/excalidraw, @excalidraw/mermaid-to-excalidraw, react 18 |
| 제작자            | Kun Chen (kunchenguid)                                                                                                         |
| 포크 커스터마이징 | `CLAUDE.md`에 카리나 페르소나 가이드 추가 (커밋 `be56bef`)                                                                     |

### 해결하는 문제

AI에게 "여기 고쳐줘"를 전달할 때 스크린샷 + 장문 설명으로 회귀하면서 HTML의 최대 장점인 인터랙티브함을 잃는다. Lavish는 사람이 **직접 손가락으로 짚는** 방식으로 그 왕복을 줄인다.

### 작동 흐름

```
AI가 artifact.html 작성
        ↓
lavish-axi <file>        → 로컬 브라우저 UI 자동 오픈
        ↓
사람이 요소 클릭 / 텍스트 드래그 / 다이어그램 낙서 / 채팅 입력
        ↓
lavish-axi poll <file>   → AI가 롱폴링으로 대기, 피드백 수신
        ↓
AI가 파일 수정 → 라이브 리로드 → 반복
```

---

## 2. 폴더 전수조사 결과

### `src/` — 소스 29개 파일

| 파일                                    | 크기      | 역할                                                                             |
| --------------------------------------- | --------- | -------------------------------------------------------------------------------- |
| `chrome-client.js`                      | 170KB     | 브라우저 UI(크롬) 클라이언트. 대화 패널, 주석 카드, 모바일 바텀시트, 첨부 업로드 |
| `export-bundle.js`                      | 128KB     | HTML 자급자족 1파일 패키징 (로컬 자산만 인라인, 외부 요청 0회)                   |
| `server.js`                             | 124KB     | Express 로컬 서버. 세션, WebSocket, 라우팅, 보안 가드                            |
| `artifact-sdk.js`                       | 109KB     | artifact에 주입되는 SDK. 클릭 주석, 레이아웃 감사, Mermaid 팬/줌                 |
| `cli.js`                                | 105KB     | CLI 본체 (open/poll/end/export/share/design/playbook/setup/stop/server)          |
| `session-store.js`                      | 44KB      | `~/.lavish-axi/state.json` 상태 저장. 단일 AsyncMutex로 모든 변경 직렬화         |
| `export-bundle`, `chrome.css`           | 38KB      | 브라우저 크롬 스타일 (860px 모바일 분기 포함)                                    |
| `attachment-store.js`                   | 34KB      | 이미지 첨부. content-addressed (sha256이 곧 ID), TTL/디스크 쿼터                 |
| `whiteboard-core/frame/store.js`        | 50KB+     | Mermaid → Excalidraw 화이트보드 변환 및 저장                                     |
| `design-reference.js`                   | 25KB      | 에이전트용 디자인 가이드. `DESIGN_PRIORITY_RULE` 단일 출처                       |
| `layout-warnings.js`                    | 23KB      | 레이아웃 이슈 라이프사이클 (순수 함수, 브라우저/서버 없이 테스트 가능)           |
| `playbooks.js`                          | 22KB      | 8종 플레이북                                                                     |
| `chat-messages.js`                      | 21KB      | 대화 트랜스크립트 직렬화/마크다운 렌더 (서버 소유 표시 상태)                     |
| `html-app.js`                           | 15KB      | ht-ml.app 공유 발행                                                              |
| `plugin.js`                             | 13KB      | Agent Plugin 클라이언트 등록 (VS Code / Cursor / Copilot CLI)                    |
| `table-cell.js`                         | 8KB       | 테이블 셀의 의미적 행·열 이름 추출                                               |
| `skill.js`                              | 8KB       | 설치용 스킬 마크다운 생성 + 프론트매터 검증                                      |
| `telemetry.js`                          | 5KB       | Umami 익명 통계 (`LAVISH_AXI_TELEMETRY=0`으로 비활성)                            |
| `paths.js` / `tailscale.js`             | 5KB / 4KB | 호스트·포트·링크 호스트 해석, Tailscale 감지                                     |
| `mermaid-node.js` / `mermaid-source.js` | 4KB / 3KB | Mermaid 노드 식별, 소스 추출                                                     |
| `self-paint.js`                         | 3KB       | 렌더 없는 배경 누락 경고 (fail-open)                                             |
| `async-mutex.js`                        | 2KB       | 스토어 전역 뮤텍스                                                               |
| `share-password.js`                     | 1KB       | 공유 비밀번호 생성 단일 출처                                                     |
| `html-transform.js`                     | 1KB       | SDK 스크립트 태그 1줄만 주입                                                     |

### `test/` — 42개 테스트 파일

`node:test` 러너 기반. 실브라우저 E2E 5종은 `LAVISH_AXI_BROWSER_E2E=1`로 옵트인:
`layout-audit-browser`, `layout-warning-inbox.browser`, `event-transport.browser`, `whiteboard-render.browser`, `attachment-upload.browser`, `mobile-conversation-sheet.browser`.

### 기타 폴더

| 경로                            | 내용                                                                                                                           |
| ------------------------------- | ------------------------------------------------------------------------------------------------------------------------------ |
| `skills/lavish/SKILL.md`        | 공개 Agent Skill. 일부러 **얇은 스텁** — 상세 가이드는 CLI를 가리킴. `scripts/build-skill.js`로 자동 생성, 드리프트 시 CI 실패 |
| `.agents/skills/lavish-design/` | 내부 브랜드 스킬 (`metadata.internal: true`로 숨김). React JSX UI킷 다수 포함                                                  |
| `lavish-editor-marketing/`      | 마케팅 랜딩 페이지 + 데모 GIF/MP4 + hyperframes 애니메이션                                                                     |
| `task-evidence/`                | 버그 수정 증거 스크린샷 (Mermaid 테마, Excalidraw 라벨 클리핑)                                                                 |
| `.github/workflows/`            | `ci.yml`, `release-please.yml`, `guard-generated-files.yml`, `no-mistakes-required.yml`                                        |
| `docs/self-hosting-share.md`    | 공유 백엔드 자체 호스팅 시 구현해야 할 API 계약                                                                                |
| `AGENTS.md` (86KB)              | 아키텍처 내부사정·불변식·보안 근거. 모든 설계 결정의 "왜"가 담긴 보물                                                          |
| `VISION.md`                     | 수용 정책. 원칙과 **명시적 non-goals**                                                                                         |

---

## 3. 핵심 기능

1. **요소 주석** — HTML 요소 클릭 → 메모 작성 → CSS selector까지 함께 전달
2. **텍스트 범위 주석** — 드래그 선택 텍스트 + 범위 앵커(시작/끝 경계)
3. **테이블 셀 주석** — 좌표가 아니라 **의미적 행·열 이름**을 전달. 모호하면 **추측하지 않고 생략**
4. **Mermaid 화이트보드** — Mermaid 다이어그램을 Excalidraw로 변환해 직접 편집. 역변환 없음, 편집 결과는 "요약"으로 전달되고 Mermaid 소스가 권위
5. **이미지 첨부** — 붙여넣기/드래그/피커. 서버가 경로·mime·크기를 디스크에서 재도출
6. **라이브 리로드** — 파일 변경 감지 + 작성 중 메모·스크롤·폼 답변 보존
7. **레이아웃 이슈 인박스** — 글자 잘림, 뷰포트 넘침, 요소 겹침 등을 수동적으로 탐지. **AI를 절대 깨우지 않음**
8. **Export** — 1개 HTML로 자급자족 패키징. 외부 요청 0회, 심볼릭 링크 탈출 차단
9. **Share** — ht-ml.app 발행. 기본 공개, `--private`/`--password`로 비공개
10. **모바일 리뷰** — 860px 이하 바텀시트 UI, Tailscale MagicDNS로 폰 접속
11. **에이전트 프레즌스** — `waiting` / `listening` / `working` 상태를 브라우저에 표시
12. **8종 플레이북** — `diagram`, `table`, `comparison`, `plan`, `code`, `input`, `explanation`, `slides`

---

## 4. 설치 및 사용법

### 설치 방법 5가지

```sh
# 1) 스킬만 설치 (권장, 추가 설치 없음)
npx skills add kunchenguid/lavish-axi --skill lavish
npx skills add kunchenguid/lavish-axi --skill lavish -g   # 전역

# 2) 설치 0 — 에이전트에게 그냥 지시
#    "npx -y lavish-axi 써서 기획서 만들어줘"

# 3) 세션 훅 (라이브 세션 목록까지 주입)
npm install -g lavish-axi && lavish-axi setup hooks
#    → Claude Code / Codex / OpenCode / GitHub Copilot CLI, 설치 후 세션 재시작 필수

# 4) Agent Plugin 등록
npm install -g lavish-axi && lavish-axi setup plugin
#    → VS Code / Cursor / GitHub Copilot CLI, 설치 후 클라이언트 리로드

# 5) 소스 빌드 (포크 개발용)
git clone https://github.com/bmshin94/lavish-axi.git
cd lavish-axi && pnpm install --frozen-lockfile && pnpm run build && pnpm link
```

### CLI 명령어

| 명령어                              | 설명                                                          |
| ----------------------------------- | ------------------------------------------------------------- |
| `lavish-axi`                        | 현재 세션 + 사용 가이드 출력                                  |
| `lavish-axi <html>`                 | 세션 열기/재개. 사용자가 종료한 세션은 `--reopen` 없이는 거부 |
| `lavish-axi poll <html>`            | 피드백 롱폴링 대기 (에이전트 핵심 명령)                       |
| `lavish-axi end <html>`             | 에이전트가 세션 종료 (이후 평범한 재개 가능)                  |
| `lavish-axi export <html>`          | `<name>.export.html` 생성                                     |
| `lavish-axi share <html>`           | ht-ml.app 발행                                                |
| `lavish-axi stop`                   | 백그라운드 서버 종료                                          |
| `lavish-axi design`                 | 디자인 가이드 + CDN 스니펫                                    |
| `lavish-axi playbook [id]`          | 플레이북 목록/상세                                            |
| `lavish-axi setup hooks` / `plugin` | 훅/플러그인 설치                                              |
| `lavish-axi server`                 | 서버 직접 실행                                                |
| `lavish-axi update`                 | 자체 업데이트                                                 |

### 주요 플래그

`--no-open`, `--no-gate`, `--reopen`, `--out <path>`, `--private`, `--password <pw>`, `--site <id>`, `--update-key <key>`, `--unpublish`, `--token <t>`, `--agent-reply "..."`, `--agent-reply-file <path>`, `--timeout-ms <ms>`, `--port <port>`, `--verbose`

### 주요 환경변수

```sh
LAVISH_AXI_PORT=4387                 # 서버 포트
LAVISH_AXI_STATE_DIR=~/.lavish-axi   # 상태 저장 위치
LAVISH_AXI_IDLE_TIMEOUT_MS=1800000   # 유휴 종료 (0/off=비활성)
LAVISH_AXI_TELEMETRY=0               # 텔레메트리 끄기
LAVISH_AXI_HOST / LAVISH_AXI_LINK_HOST
LAVISH_AXI_ALLOWED_HOSTS             # 허용 호스트 (`*`로 가드 해제)
LAVISH_AXI_DIAGNOSTIC_VIEWPORTS      # mobile,compact,desktop
LAVISH_AXI_MAX_ATTACHMENT_BYTES      # 기본 10MiB
LAVISH_AXI_MAX_ATTACHMENTS_PER_PROMPT # 기본 4
LAVISH_AXI_MAX_PROMPT_ATTACHMENT_BYTES # 기본 25MiB
LAVISH_AXI_ATTACHMENT_TTL_MS         # 기본 7일
LAVISH_AXI_MAX_ATTACHMENT_DISK_MB    # 기본 512MiB
LAVISH_AXI_EXPORT_MAX_ASSET_BYTES    # 기본 10MB
LAVISH_AXI_EXPORT_MAX_BUNDLE_BYTES   # 기본 25MB
LAVISH_AXI_HTML_APP_API_URL          # 공유 백엔드 교체 지점
LAVISH_AXI_HTML_APP_TOKEN            # 선택적 bearer token
LAVISH_AXI_DEBUG=1                   # 디버그 로그
LAVISH_AXI_NO_OPEN=1                 # 브라우저 미실행
```

### 키보드 단축키

| 키               | 동작                                                   |
| ---------------- | ------------------------------------------------------ |
| `Enter`          | 전송 (주석 카드에서는 큐에 추가)                       |
| `Shift+Enter`    | 줄바꿈                                                 |
| `Ctrl/Cmd+Enter` | 주석 큐 추가 + 즉시 전송                               |
| `Ctrl/Cmd+I`     | 주석 모드 ↔ 탐색 모드 전환                             |
| `Esc`            | 주석 카드 닫기 (내용이 있으면 무시 — 데이터 손실 방지) |

---

## 5. 플러그인? 스킬? MCP?

**본질은 CLI (AXI)** 이고, 스킬과 플러그인은 **발견(discovery) 전용 래퍼**다. **MCP 서버는 명시적으로 아니다.**

| 형태         | 해당 | 근거                                                                                                           |
| ------------ | ---- | -------------------------------------------------------------------------------------------------------------- |
| CLI (AXI)    | 본질 | `bin/lavish-axi.js` → `src/cli.js`, `dist/cli.mjs`                                                             |
| Agent Skill  | 래퍼 | `skills/lavish/SKILL.md` (생성된 스텁)                                                                         |
| Agent Plugin | 래퍼 | 패키지 루트의 `plugin.json` + `skills/` — 마켓플레이스 불필요                                                  |
| MCP Server   | 아님 | `VISION.md`: "It is not an MCP server"; `AGENTS.md`: MCP를 추가하면 "유지해야 할 두 번째 에이전트 계약"이 생김 |

---

## 6. API 토큰 필요 여부

**핵심 기능은 토큰이 전혀 필요 없다.**

| 기능                                    | 토큰                                                                             |
| --------------------------------------- | -------------------------------------------------------------------------------- |
| 열기/주석/화이트보드/첨부/poll/export   | 불필요 (100% 로컬)                                                               |
| share (ht-ml.app 발행)                  | **계정·API 키 불필요**. 응답의 `update_key`가 유일한 자격증명이며 한 번만 반환됨 |
| `--token` / `LAVISH_AXI_HTML_APP_TOKEN` | **선택** (자체 백엔드용 bearer token). 재발행/비발행에서는 거부됨                |
| 텔레메트리                              | 불필요, 익명, 비활성 가능                                                        |

Lavish는 AI를 호출하지 않으므로 **추가 AI API 비용이 발생하지 않는다** (`VISION.md`: "Lavish does not launch, drive, or supervise the agent itself"). 비밀번호와 `update_key`는 어디에도 저장되지 않는다.

---

## 7. AI 에이전트 구축에 주는 도움

### 도구로서

- Human-In-The-Loop 레이어를 가장 쉽게 붙이는 방법
- 에이전트 산출물 정밀 검수 표준 UI
- 멀티 에이전트 파이프라인의 사람 검수 단계
- 클라이언트 데모 + 즉석 피드백 수집

### 교과서로서 (배울 설계 패턴)

1. **토큰 효율** — 점진적 공개(`--help`/`design`/`playbook` 분리), 롱폴링 + 하트비트, TOON 직렬화
2. **응답 키 순서가 계약** — `prompts` → `artifact_failures` → `next_step` → `dom_snapshot`. 에이전트가 응답을 잘라도 사용자의 말과 다음 지시에 반드시 도달. 테스트로 고정됨
3. **감지하되 깨우지 않는다** — 자동 수정 루프(auto-repair churn)의 함정 회피
4. **Fail-open 원칙** — "매번 뜨는 오탐이 미탐보다 비싸다"
5. **문서 단일 소유권** — README(사용자 계약) / CLI 출력(에이전트 지시) / VISION(수용 정책) / AGENTS(내부사정). 중복 금지, CI가 드리프트 차단
6. **동시성 안전** — 단일 AsyncMutex, 파괴적 take 후 `restore` 복구(PREPEND + 새 데이터 우선)
7. **신뢰 경계** — 클라이언트는 id + 표시이름만, 서버가 권위 필드 재도출, all-or-nothing 검증
8. **보안 패턴** — DNS 리바인딩(Host 허용목록), 클릭재킹(`frame-ancestors 'none'`), 심볼릭 링크 탈출(realpath), Confused Deputy 중재, 레이트리밋 3중 방어(횟수·누적바이트·동시성)

---

## 8. React / PHP 구현 가능성

### React — 부분적으로 가능 (이미 쓰고 있음)

- `src/whiteboard-frame.js`가 React 18로 Excalidraw 마운트 (이미 React)
- `.agents/skills/lavish-design/ui_kits/editor/`에 React JSX 목업 다수 (`Artifact.jsx`, `ChatPanel.jsx`, `TopBar.jsx`, `AnnotationCard.jsx`, `Bubbles.jsx` 등)
- 다만 메인 크롬 UI(`chrome-client.js`)는 **의도적으로 React 미사용** — "raw 파일로 서빙되어 모듈 import 불가" (그래서 표시 문자열을 서버에서 계산)

권장 전략: 전체 리라이트가 아니라 **React 레이어 점진 도입** (ui_kits 기반 → build.js에서 번들 산출물 서빙 → postMessage 프로토콜 유지).

### PHP — 기술적으론 가능하지만 비효율

| 필요 기능      | Node(현재)           | PHP                      |
| -------------- | -------------------- | ------------------------ |
| 분 단위 롱폴링 | 이벤트 루프 네이티브 | 워커 점유, FPM 고갈 위험 |
| WebSocket      | `ws` 사용            | Ratchet/Swoole 필요      |
| 파일 와처      | `chokidar`           | inotify 확장 필요        |
| 데몬 프로세스  | detached spawn       | 웹서버 모델과 충돌       |
| 배포 간편성    | `npx` 한 줄          | PHP + Composer 전제      |

**PHP가 적합한 자리**: 공유 백엔드(ht-ml.app 대체, `docs/self-hosting-share.md` 계약 구현), 팀 대시보드, 결제·인증·관리자 패널.

권장 조합: 코어는 Node 유지 / 브라우저 UI는 React 점진 도입 / SaaS 백엔드는 PHP·Next.js 자유 선택.

---

## 9. 유튜브 강의 제작 가능성

**가능하며 소재 가치가 높다.** 한국어 콘텐츠가 거의 없고, 클릭 → 피드백 → 자동 수정이 화면에 바로 보이며, MIT 라이선스라 코드 공개가 자유롭고, 마케팅 GIF/MP4가 저장소에 이미 포함되어 있다.

### 주의사항

1. 원작자(`kunchenguid`)와 저장소 링크 명시
2. MIT LICENSE 고지
3. "Lavish Editor" 명칭 사용 시 비공식임을 밝히기
4. 버전 자막 필수 (0.1.x, 변경 빈번)
5. ht-ml.app은 **서드파티이며 기본 공개**임을 경고
6. `LAVISH_AXI_HOST` 외부 바인딩의 보안 위험 언급

### 추천 시리즈 (10편)

| #   | 주제                                                 | 길이 |
| --- | ---------------------------------------------------- | ---- |
| 1   | AI에게 "여기 고쳐"를 손가락으로 가리키는 방법 (후킹) | 8분  |
| 2   | 5분 설치 완전정복 — 스킬/훅/플러그인 3가지 길        | 10분 |
| 3   | 실전 ①: 프로젝트 로드맵 만들고 피드백 주기           | 15분 |
| 4   | 실전 ②: Mermaid 다이어그램 손으로 그려서 수정        | 15분 |
| 5   | 실전 ③: 비교표 + 테이블 셀 주석의 마법               | 12분 |
| 6   | 폰에서 AI 결과물 리뷰하기 (Tailscale)                | 10분 |
| 7   | Export & Share — 1파일 패키징과 공유 보안            | 12분 |
| 8   | 아키텍처 해부 ①: 롱폴링과 토큰 효율                  | 20분 |
| 9   | 아키텍처 해부 ②: 샌드박스와 보안 설계                | 20분 |
| 10  | 포크해서 내 도구 만들기                              | 25분 |

보조 기획: Shorts 30초 클립, "Lavish vs 스크린샷 왕복 횟수 측정" 비교 영상, 라이브 코딩 스트림, AGENTS.md 리뷰 영상(개발자 타겟).

---

## 10. 수익화 아이디어

### 기회의 출처: 제작자가 명시한 non-goals

| 제작자의 non-goal                   | 파생 기회             |
| ----------------------------------- | --------------------- |
| 1명 1에이전트 1파일만               | 팀/멀티유저 협업 버전 |
| HTML만 (Markdown/PDF/이미지 미지원) | 포맷 확장 제품        |
| 호스팅 서비스 아님                  | 매니지드 호스팅 SaaS  |
| MCP 서버 아님                       | MCP 브릿지 제품       |
| artifact 자동 수정 안 함            | AI 자동수정 애드온    |
| 에이전트 구동·감독 안 함            | 오케스트레이션 플랫폼 |
| 디자인 시스템 미주입                | 프리미엄 템플릿 마켓  |

### 아이디어 11종 요약

| #   | 아이디어                            | 난이도 | 기간     | 초기비용 | 예상 월수익       | 추천도    |
| --- | ----------------------------------- | ------ | -------- | -------- | ----------------- | --------- |
| 1   | 교육 콘텐츠 (유튜브 → 유료 강의)    | 낮음   | 2-4주    | ~0       | 50만~500만        | 매우 높음 |
| 2   | 프리미엄 플레이북·템플릿 마켓       | 낮음   | 2-3주    | ~0       | 30만~300만        | 높음      |
| 3   | 셀프호스팅 공유 백엔드 SaaS         | 낮음   | 3-4주    | 50만     | 50만~1000만       | 매우 높음 |
| 4   | 도입 컨설팅                         | 낮음   | 즉시     | ~0       | 프로젝트당 수백만 | 매우 높음 |
| 5   | **팀 협업 버전**                    | 중간   | 2-3개월  | 500만    | 500만~5000만      | 매우 높음 |
| 6   | MCP 브릿지                          | 중간   | 3-4주    | ~0       | 간접(리드)        | 보통      |
| 7   | 포맷 확장 (PDF/이미지/Jupyter/영상) | 중간   | 2-3개월  | 300만    | 300만~2000만      | 높음      |
| 8   | AI 자동수정 애드온                  | 중간   | 1-2개월  | 200만    | 200만~1500만      | 보통      |
| 9   | 올인원 플랫폼                       | 높음   | 6-12개월 | 5000만+  | 수천만+           | 높음      |
| 10  | 버티컬 SaaS (의료/금융/법무/건설)   | 높음   | 4-8개월  | 2000만   | 1000만~5000만     | 높음      |
| 11  | 오픈코어 + 엔터프라이즈             | 높음   | 6-12개월 | 3000만   | 계약당 수천만     | 보통      |

※ 금액은 시장 규모와 실행력에 따라 크게 달라지는 **가정 기반 추정치**이며 보장치가 아니다.

### 핵심 아이디어 상세

#### #3 셀프호스팅 공유 백엔드 — 가장 빠른 SaaS

`docs/self-hosting-share.md`에 API 계약이 이미 문서화되어 있고 `LAVISH_AXI_HTML_APP_API_URL`만 교체하면 되므로 개발 기간이 짧다. ht-ml.app의 약점이 그대로 세일즈 포인트가 된다.

| ht-ml.app 한계                           | 대체 제품의 차별점      |
| ---------------------------------------- | ----------------------- |
| 기본 공개                                | 기본 비공개             |
| 삭제 엔드포인트 없음                     | 실제 삭제 지원          |
| 비밀번호 해제 불가                       | 자유 변경/해제          |
| 공개→비공개 전환이 CDN 캐시로 수 분 지연 | 즉시 반영               |
| 서드파티 (데이터 통제 불가)              | 자체 호스팅 / 리전 선택 |
| 접근 로그 없음                           | 조회 분석 + 감사 로그   |

추가 기능: 커스텀 도메인, SSO/SAML, 만료일, 워터마크, PDF 변환, 뷰어 분석.

#### #5 팀 협업 버전 — 가장 큰 시장

`VISION.md`가 "it is not a place for two people to collaborate with each other"라고 명시적으로 범위 밖으로 선언했다. MIT 라이선스이므로 포크해서 구현하는 데 법적 문제가 없고 업스트림과 경쟁하지 않는다.

추가 기능: 멀티유저 주석(작성자 표시), 주석 스레드, 승인 워크플로우, @멘션 + 슬랙/팀즈 알림, 버전 히스토리/diff, 역할 권한, Jira/Linear/Notion 연동, 사용 분석.

구현 방향: `state.json` → PostgreSQL + Redis + 실시간 동기화, UI는 `ui_kits/editor/*.jsx` 기반 React 리라이트, 인증은 Auth0/Clerk + SSO.

#### #10 버티컬 SaaS 후보

의료/임상(규제 대응), 금융(IR·투자보고서 컴플라이언스), 법무(계약서 조항 단위 주석), 건설/엔지니어링(도면·공정표), 게임(기획서·레벨디자인), 교육·출판(교재 검수). 버티컬은 경쟁이 적고 단가가 높으며 고객 충성도가 높다.

### 실행 로드맵

```
1~2개월  씨앗 뿌리기 (리스크 0)
         유튜브 입문 3편 + 템플릿 팩 1~2종 + 컨설팅 포트폴리오
         목표: 월 100만 + 리드 10건

3~4개월  첫 SaaS (현금흐름)
         셀프호스팅 공유 백엔드 출시 + 심화 시리즈 + 유료 강의
         목표: MRR 300만

5~8개월  본게임 (스케일)
         팀 협업 버전 MVP + React UI 리라이트 + 베타 10팀
         목표: MRR 1,500만

9~18개월 확장
         포맷 확장 또는 버티컬 선택 + 엔터프라이즈 영업 + 투자 검토
         목표: ARR 3억+
```

### 법적·윤리 체크리스트

| 항목              | 조치                                              |
| ----------------- | ------------------------------------------------- |
| MIT 라이선스 고지 | LICENSE 파일 유지 및 표시 (필수)                  |
| 원작자 크레딧     | `kunchenguid` 명시 (매너이자 신뢰 자산)           |
| 서브 의존성 고지  | `THIRD-PARTY-NOTICES.md` 유지                     |
| 상표              | "Lavish" 명칭 그대로 쓰지 않고 별도 브랜드명 권장 |
| 업스트림 기여     | 범용 개선은 PR로 환원 (커뮤니티 평판)             |
| 의존성 라이선스   | Excalidraw, DaisyUI, mermaid 등 재확인            |
| 텔레메트리        | 상용화 시 자체 엔드포인트로 교체 + 사용자 고지    |

### 종합 추천

**아이디어 1(유튜브) + 4(컨설팅)로 시작 → 3(공유 백엔드)으로 현금흐름 확보 → 5(팀 협업)에 집중.**

- 1 + 4는 자본 리스크가 없고 신뢰와 리드를 쌓는다.
- 3은 API 계약이 이미 문서화되어 개발 기간이 가장 짧은 SaaS다.
- 5는 제작자가 명시적으로 포기한 영역이며 기업의 지불 의사가 높다.

---

## 부록: 개발 명령어

```sh
pnpm run check          # build + lint + format:check + typecheck + test + 스킬/플러그인 드리프트 검사
pnpm run build          # dist/cli.mjs 번들 + chrome/design 자산 복사
pnpm run build:skill    # skills/lavish/SKILL.md 재생성
pnpm run build:plugin   # plugin.json 재생성
pnpm test               # node:test 러너
pnpm run lint           # ESLint (bin src test scripts)
pnpm run format:check   # Prettier 검사
pnpm run typecheck      # tsc --noEmit (checkJs)

node --test test/server.test.js                      # 단일 테스트 파일
LAVISH_AXI_BROWSER_E2E=1 node --test test/layout-audit-browser.test.js   # 실브라우저 E2E
```

주의: `CHANGELOG.md`, `.release-please-manifest.json`은 release-please가 소유하므로 직접 편집하지 않는다. `skills/lavish/SKILL.md`와 `plugin.json`은 생성 파일이므로 생성기를 수정해야 한다.
