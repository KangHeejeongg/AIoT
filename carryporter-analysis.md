# CarryPorter 개인 기여 분석 (강희정)

- 분석 기준: `master` HEAD `1253d73` (2026-02-18 머지), 전체 브랜치 히스토리(unshallow)
- 대상 author: `강희정 <he5298@naver.com>`, `KangHeejeongg <96936403+KangHeejeongg@users.noreply.github.com>`
- 표기 규칙
  - **[사실]**: diff, 커밋 메시지, 현재 코드, 명령 출력으로 확인한 내용
  - **[추측]**: 확인한 사실에서 추론한 내용으로, 실행하거나 로그로 검증하지 않음
  - 커밋 해시는 모두 7자리 short hash

---

## 1. 작업 범위

### 1.1 기여자 목록 (`git shortlog -sne --all`)

| 커밋 | Author |
|---:|---|
| 290 | 김정훈 <kik1232198@gmail.com> |
| **189** | **강희정 <he5298@naver.com>** |
| 84 | 전준완 <joownan1024@naver.com> |
| 78 | joonwan1024 <joownan1024@naver.com> |
| 55 | umaicode <park.seungchan99@gmail.com> |
| 34 | 전준완 <joonwan1024@naver.com> |
| 4 | DESKTOP-6MRTL1B\SSAFY <kik1232198@gmail.com> |
| **3** | **KangHeejeongg <96936403+KangHeejeongg@users.noreply.github.com>** |
| 3 | Seokihyeon <kint_myu_yoi@naver.com> |
| 3 | 박승찬 <sc_park99@naver.com> |
| 2 | joonwan <joonwan1024@naver.com> |
| 1 | JungHoonKim-KR <100742423+JungHoonKim-KR@users.noreply.github.com> |
| 1 | 정승현 <shjung2506@naver.com> |

본인 author는 위 굵게 표시한 2개입니다. author/committer에서 `he5298`, `heejeong`, `희정`, `kang`을 검색했을 때 다른 이름이나 이메일은 나오지 않았습니다.

### 1.2 담당 영역 요약

| 영역 | 담당 정도 (현재 HEAD blame 기준) |
|---|---|
| 인증 (JWT, Refresh Token, Redis, 필터) | `AuthService` 221/223줄, `AuthController` 139/140줄, `JwtUtils` 114/126줄, `JwtAuthenticationFilter` 75/93줄 |
| SSE 알림 | `SseService` 88/177줄, `SseEmitterRepository` 51/67줄, `UserSseNotificationHandler` 112/128줄, `AdminSseNotificationHandler` 36/133줄 |
| MQTT 연동 | `MqttSubscriberService` 92/334줄. `MqttConfig`, `MqttPublisherService`는 거의 팀원 코드 |
| 사물함 잠금/해제, 실패 카운트 | `FailureCountHandler` 68/68줄, 잠금·해제 API |
| 관리자 프론트 (`admin-frontend`) | 현재 코드의 97% (6,999/7,191줄). `RobotStage.tsx` 1014/1038, `RobotsPage.tsx` 572/575 |

### 1.3 변경 디렉터리 상위 10개 (merge 제외, 2단계 경로 기준 파일 변경 수)

| 순위 | 디렉터리 | 변경 파일 수 |
|---:|---|---:|
| 1 | `backend/src` | 236 |
| 2 | `admin-frontend/src` | 115 |
| 3 | `admin-fronted/public` | 86 |
| 4 | `admin-fronted/src` | 72 |
| 5 | `admin-frontend/public` | 44 |
| 6 | `admin-frontend/` (루트 파일) | 18 |
| 7 | `admin-fronted/` (루트 파일) | 16 |
| 8 | `admin-fronted/docs` | 14 |
| 9 | `00Document/회고` | 10 |
| 10 | `admin-frontend/docs` | 7 |

`admin-fronted`(오타)와 `admin-frontend`는 같은 코드입니다. `dc236f5`에서 추가하고, `4a9c0de`에서 복사본을 추가하고, `02ace91`에서 오타 폴더를 삭제하면서 이중으로 집계됐습니다.

### 1.4 기능 묶음 요약

| # | 묶음 | 커밋 수 | 기간 |
|---|---|---:|---|
| 1 | 인증 (JWT/Redis/필터) | 15 | 01-27 ~ 02-01 |
| 2 | SSE 알림 (백엔드) | 17 | 01-26 ~ 02-08 |
| 3 | MQTT 로봇 통신 | 6 | 01-29 ~ 02-08 |
| 4 | 사물함 API (조회·잠금/해제) | 4 | 01-29 ~ 01-30 |
| 5 | 관리자 로봇 제어·보관/반납 API | 7 | 02-02 ~ 02-08 |
| 6a | 관리자 프론트: 레이아웃/화면/API 연동 | 17 | 02-02 ~ 02-08 |
| 6b | 관리자 프론트: 3D 맵 | 10 | 02-03 ~ 02-08 |
| 6c | 관리자 프론트: SSE 훅/알림 화면 | 10 | 02-03 ~ 02-08 |
| 7 | 테스트 (테스트만 바꾼 커밋) | 6 (테스트를 건드린 커밋은 30) | 01-28 ~ 01-31 |
| 0 | 초기 문서/세팅 | 7 | 01-12 ~ 01-15 |

전체 흐름은 1월 말 백엔드(인증 → SSE → MQTT/사물함), 2월 2일부터 관리자 프론트입니다.

---

## 2. 기능별 타임라인

날짜는 KST입니다. 하나의 커밋이 여러 묶음에 걸치면 주된 묶음에 넣고, 다른 묶음에서는 *기울임*으로 표시했습니다.

### 2.0 초기 문서 / 세팅 (01-12 ~ 01-15)
`449cef9` README · `030bb80` 팀원별 폴더 · `9345421` 더미 파일 · `93ef05d` 문서 폴더 · `e6a83d6` 회고 형식 · `bb41b98` 폴더 구조 정리 · `ab3c70d` 회고 문서

### 2.1 인증 (JWT / Redis / 필터) — 01-27 14:13 ~ 02-01 23:25

| 커밋 | 날짜 | 내용 |
|---|---|---|
| `bec12cb` | 01-27 | 필터 구현 (`JwtUtils`, `JwtAuthenticationFilter`, `CorsFilter`, `WebConfig`) |
| *`7fc5958`* | 01-27 | 필터 수정 (SSE와 공통) |
| `e68d076` | 01-28 | 로그인/인증번호 (`AuthController`, `AuthService`, `EmailCodeRedisRepository`, `TempPasswordRedisRepository`, `MattermostClient`) |
| `e309899`, `262005c` | 01-28 | 입력값 검증, DTO, refresh token |
| `08c51ef` | 01-28 | 코드리뷰 반영, DTO 패키지 정리 |
| `e177f7a` / `17c5885` | 01-28 | 배포 테스트용 / 로컬 실행용 경로 수정 |
| `a275b04` | 01-28 | 응답에 인증번호 추가 |
| `2eb4cf0` | 01-29 | 중복 이메일 분기 (find-or-create) |
| `478b717` | 01-29 | refresh token을 쿠키로 |
| `d458618` | 01-29 | reissue를 화이트리스트에 추가 |
| `6c6f09a` | 01-31 | refresh token 반환값 수정 (Rotation 버그) |
| *`702a584`* | 02-01 | 로컬 개발용 필터/CORS |
| *`0cca96a`* | 01-28 | 로그인 실패 횟수 Redis (`LoginFailCountRedisRepository`) |

### 2.2 SSE 알림 (백엔드) — 01-26 22:51 ~ 02-08 15:39

| 커밋 | 날짜 | 내용 |
|---|---|---|
| `6711109` | 01-26 | SSE 구현 (관리자/사용자): `SseController`, `SseService`, `SseEmitterRepository` |
| `7fc5958` | 01-27 | 필터 연동 수정 |
| `8c9e166` | 01-28 | SSE enum 삭제 |
| `ab34b69` | 01-28 | 이벤트 뼈대 (`RobotArrivalEvent`, `UserAuthFailed/SuccessEvent`, `MissionLocked/AbortedEvent`) |
| `6d6b9d2` | 01-28 | 이벤트별 필드 결정 |
| `0cca96a` | 01-28 | Listener 구현 |
| `2d13c43`, `fb3e02f` | 01-28 | 연결 종료를 프론트에 맡김, `complete()` 삭제 |
| `9cbcb00` | 01-29 | Admin/User 핸들러 분리 |
| `b1c69a4` | 01-29 | 이벤트 변경 작업 |
| `6583f36` | 01-31 | 로봇 열림 완료 메시지 |
| `84eacd3` | 01-31 | 구독 신원을 토큰 기반으로 변경 |
| `2d498c7` | 02-01 | 하트비트 추가 |
| `03d5ddf`, `ec985c7` | 02-01 | 배정 이벤트, 핸들러 수정 |
| `1817d77` | 02-03 | SSE 테스트 컨트롤러, `broadcastToUsers` |
| *`0c7a460`* | 02-08 | 긴급정지 알림 |

### 2.3 MQTT 로봇 통신 — 01-29 21:27 ~ 02-08 17:13

| 커밋 | 날짜 | 내용 |
|---|---|---|
| `fead195` | 01-29 | RobotService와 MQTT 테스트 |
| `dce3d3e`, `6ac2547` | 01-30 | 로봇 할당 수신 처리와 테스트 |
| `15ce27c` | 01-30 | 트랜잭션 위치 수정 |
| `22fc20e` | 02-01 | 로봇 복귀 이벤트 |
| `7b12e6e` | 02-03 | 복귀 이벤트에 lockerId 추가 |
| (`0c7a460`, `94c4d63`) | 02-08 | 긴급정지 수신, 언락 명령 |

### 2.4 사물함 API — 01-29 12:36 ~ 01-30 21:35

| 커밋 | 날짜 | 내용 |
|---|---|---|
| `bd1b002` | 01-29 | 비밀번호 검증 후 잠금 해제 요청 |
| `1e62133` | 01-30 | 잠금 요청/응답 (MQTT 연동, `MissionLockRequestEvent`) |
| `f19ac73` | 01-30 | 상태별 잠금 로직 (`MissionUnlockedEvent`) |
| `54e09c8` | 01-30 | 사물함 조회 API (`UserLockerController`), Mattermost 채널 변경 |

### 2.5 관리자 로봇 제어 · 보관/반납 API — 02-02 12:11 ~ 02-08 17:13

`435d851` missionId만 전달 · `2c95cf7` 관리자 권한 열기 이벤트 · `89eaada` 임시 · `d193d90` 보관 API 수정 중 · `0d3f9df` 보관/반납 로직 (`MissionStoredEvent`, `MissionStatusHandler`, `RobotStatusHandler`) · `58102b6` 배정 이벤트에 MAC 주소 추가 · `94c4d63` 어드민 언락

### 2.6 관리자 프론트 (`admin-frontend`)

**6a. 레이아웃 / 화면 / API 연동 (02-02 19:45 ~ 02-08 16:53)**
`dc236f5` 초기 생성 · `4a9c0de`, `02ace91` 폴더명 변경 · `fd7882d` 릴리스 업데이트 · `752d8cc` 로고 · `231d6ac` API 연결 (`useRobotFetch`) · `ce8a921`, `59cc4f6` vite 프록시 · `c4553d6` · `20c2b2d` 대시보드 삭제 · `1f8f433` 모니터링 CSS/API · `40cd7f3` 충돌 수정 · `4d22b39` 보관 처리 요청 · `183d78c` 전송 중 닫힘 · `a1c520f` 모달 · `7b901f8`, `0d14dfc` 잠금 해제 요청/원복

**6b. 3D 맵 (02-03 00:23 ~ 02-08 21:47)**
`f43ddd8` glb 모델 · `7a69004` 로봇 상세/3D · `c316c64` 캐리포터 모델 · `44d10a1` 맵 수정 · `b2c776c` 활동 터미널 · `8a14ac9` 좌표 순간이동 · `18e60b5` 경로 임시 구현 (`navigationPaths.ts`) · `d7a5b75` 로봇 이동 임시 · `6ad0bc4` 경로 이동 · `88ecba9` 스테이션 변경

**6c. SSE 훅 / 알림 화면 (02-03 01:04 ~ 02-08 16:14)**
`a6c1f42` 복귀 이벤트 lockerId · `8ce839a` SSE 모니터링 (`RealtimeActivityFeed`) · `6309b9d` SSE JSON · `b8c873e` 관리자 알림 개선 · `d392e4e` 이벤트 핸들러 · `30b5c15` 로봇 디스플레이 (`AlertPage`, `sseStore`) · `be5165d` 롤백 · `61c5710` SSE 이벤트 처리 분리 · `e5f9aab` 알림·헤더 · `2a28830` 호출 장소 변경 알림

### 2.7 테스트

- 테스트만 바꾼 커밋: `e6bece3` · `6a590f8` · `2464d56` · `ad36d3d` · `75058bc` · `dcfecea`
- 기능 개발과 함께 테스트도 바꾼 커밋: `6711109` `bec12cb` `7fc5958` `e68d076` `262005c` `08c51ef` `8c9e166` `17c5885` `a275b04` `2eb4cf0` `bd1b002` `9cbcb00` `478b717` `fead195` `dce3d3e` `6ac2547` `1e62133` `f19ac73` `54e09c8` `84eacd3` `435d851` `2c95cf7` `7b12e6e` `0d3f9df`
- 프론트엔드 테스트 커밋은 없음

### 2.8 히스토리 위생 이슈
- [사실] `backend/env` 파일이 커밋됨 (`1817d77`, `d193d90`). 비밀값이 들어 있는지 확인 필요
- [사실] MQTT 클라이언트 `.lck` 파일이 커밋됨 (`54e09c8`, `2c95cf7`, `f43ddd8`). `admin-frontend/git` 파일도 커밋됨 (`c4553d6`)
- [사실] 일부 커밋 메시지 인코딩 깨짐 (`e309899`, `08c51ef` 등)

---

## 3. 트러블슈팅

### 한눈에 보기

| # | 건 | 성격 | 근거 커밋 | 간격 |
|---|---|---|---|---|
| 1 | Refresh Token Rotation에서 옛 토큰을 내려줌 | **실제 버그 수정** | `478b717` → `6c6f09a` | 약 28시간 |
| 2 | 화이트리스트 경로 `/api` 접두사 왕복 | 환경 차이 대응 | `262005c` `e177f7a` `17c5885` `a275b04` `d458618` (+팀원 `0fe86e7`) | 같은 날 4시간 안 |
| 3 | 같은 이메일로 다시 로그인하면 실패 | **실제 버그 수정** | `262005c` → `2eb4cf0` | 약 20시간 |
| 4 | `private` 메서드에 붙인 `@Transactional` | **실제 버그 수정** | `dce3d3e` → `15ce27c` | 38분 |
| 5 | SSE 구독자 신원을 쿼리 파라미터로 받음 | **보안 성격 리팩터** + 테스트 연쇄 수정 | `b1c69a4` `fead195` `ad36d3d` `75058bc` `84eacd3` `dcfecea` | 약 2일 |
| 6 | SSE 연결 종료 방식과 하트비트 | 설계 변경 / 끊김 대응 | `2d13c43` `fb3e02f` `2d498c7` | 4분, 이후 3일 |
| 7 | 프론트 SSE 시스템 메시지 필터링 | 리팩터 + 노이즈 제거 | `61c5710` | — |
| 8 | 관리자 잠금 해제 요청 왕복 | 시도 → 원복 → 백엔드 수정 | `7b901f8` → `0d14dfc` → `94c4d63` | 47분 |
| 9 | vite 프록시 대상 왕복 | 로컬/배포 전환 | `231d6ac` `ce8a921` `59cc4f6` | 75분 |
| 10 | 폴더 이름 오타 | 정리 | `dc236f5` `4a9c0de` `02ace91` | 69분 |
| 11 | 3D 맵 이동 로직 5번 개편 | 반복 개발 | `8a14ac9` `18e60b5` `d7a5b75` `6ad0bc4` `88ecba9` | 2.5일 |
| 12 | "롤백" 커밋 | 메시지와 diff 불일치 | `be5165d` | — |

### 3.1 Refresh Token Rotation에서 옛 토큰을 내려줌 ⭐
- **문제**
  - [사실] `478b717`(01-29)에서 `reissue()`가 새 refresh token을 만들어 Redis에 저장했지만, 응답에는 `.refreshToken(refreshToken) // 기존 리프레시 토큰 유지`로 예전 토큰을 담았습니다.
  - [사실] 같은 커밋에서 응답의 refresh token을 HttpOnly 쿠키로 내려주도록 바뀌어, 쿠키에 예전 토큰이 남았습니다.
- **원인**
  - [사실] Redis 갱신은 Rotation으로 바뀌었는데 응답을 만드는 줄은 그대로였습니다.
  - [추측] 두 번째 재발급부터 `AUTH_005: 토큰 정보가 일치하지 않습니다`로 실패해 강제 로그아웃처럼 보였을 것입니다.
- **수정** — [사실] `6c6f09a`(01-31): `.refreshToken(newRefreshToken) // ★ 수정완료: 반드시 '새 토큰'을 내려줘야 함!`
- **수정 후** — [사실] 발급, Redis, 쿠키가 같은 새 토큰을 씁니다. 테스트 변경은 없습니다.

### 3.2 화이트리스트 경로 `/api` 접두사 왕복
- **문제** — [사실] `JwtAuthenticationFilter`의 WHITELIST가 `262005c`(13:31) `/api/auth/*` → `e177f7a`(15:17, "배포테스트용") `/auth/*` → `17c5885`(15:58, "로컬에서 돌아가도록") `/api/auth/*` → `a275b04`(17:41) `/auth/*`로 바뀌었습니다.
- **원인**
  - [사실] 필터가 `getRequestURI()`로 비교했습니다. 팀원 커밋 `0fe86e7`(17:18)이 `context-path: ${CONTEXT_PATH:/api}`를 추가하고, 비교를 `getServletPath()`로 바꾸고, 두 형태를 모두 등록했습니다.
  - [추측] 배포 환경(프록시)과 로컬에서 실제 들어오는 경로가 달라 한쪽에 맞추면 다른 쪽이 깨졌을 것입니다.
- **수정 후** — [사실] `d458618`(01-29) 시점에는 `/auth/*`와 `/api/auth/*`가 모두 등록돼 있고 `reissue`도 추가됐습니다.
- [추측] reissue를 화이트리스트에 넣은 것은 3.1의 쿠키 방식 재발급(`478b717`, 20분 전) 때문으로 보입니다.

### 3.3 같은 이메일로 다시 로그인하면 실패
- **문제** — [사실] `262005c`에서 `requestAuth()`의 "이미 가입된 이메일" 검사를 지웠지만, `verifyAuth()`는 여전히 항상 `User.createUser(email)`을 저장했습니다.
- **원인** — [사실] `User.mmEmail`이 `@Column(unique = true)`입니다. [추측] 두 번째 인증에서 unique 제약 위반이 났을 것입니다.
- **수정** — [사실] `2eb4cf0`: `findByMmEmail(email).orElseGet(...)`로 바꾸고, "이미 가입된 유저가 다시 인증하면 새로운 유저를 생성하지 않고 기존 유저 정보를 사용한다." 테스트를 추가했습니다.
- **수정 후** — [사실] 가입과 재로그인이 같은 find-or-create 흐름을 탑니다.

### 3.4 `private` 메서드에 붙인 `@Transactional` ⭐
- **문제**
  - [사실] `dce3d3e`에서 `MqttSubscriberService`의 `private void handleArrived()`에 `@Transactional(readOnly = true) // DB 조회를 위해 트랜잭션 필요`가 붙어 있었습니다.
  - [사실] 메서드 안에서 `mission.getRobot().getMacAddress()`를 호출하는데, `Mission.robot`과 `Mission.user`는 `FetchType.LAZY`입니다.
- **원인**
  - [사실, Spring 동작] 프록시 기반 `@Transactional`은 private 메서드와 같은 클래스 안의 호출에는 적용되지 않습니다.
  - [추측] 지연 로딩에서 `LazyInitializationException`이 났지만 catch 블록이 로그만 남겨 "도착 이벤트가 안 나간다"는 증상으로 보였을 것입니다. 직전 커밋 메시지 "테스트 하고 싶은데 로봇을 미션에 넣을 수 없다 ..."도 이 무렵 디버깅 상황을 보여줍니다.
- **수정** — [사실] `15ce27c`에서 클래스 레벨 `@Transactional(readOnly = true)`로 옮겼습니다.
- **수정 후** — [사실] 이후 팀원 커밋 `b16bd92`(02-02)에서 `TransactionTemplate` 방식으로 다시 바뀌었습니다. 주석: "Paho 스레드는 Spring 관리 스레드가 아니므로 TransactionTemplate 사용".

### 3.5 SSE 구독자 신원을 쿼리 파라미터로 받음 ⭐
- **문제**
  - [사실] `84eacd3` 이전 `SseController.subscribe`는 `@RequestParam userId`, `@RequestParam role`을 클라이언트에게서 직접 받았습니다.
  - [사실] `b1c69a4`(01-29 19:09)에서 `/sse/subscribe`, `/sse/send`를 JWT 화이트리스트에 넣었고, 2시간 뒤 `fead195`에서 뺐습니다.
- **원인** — [추측] 개발 편의용 구조였지만, 이대로면 `?userId=다른사람&role=ADMIN`으로 다른 사용자나 관리자 채널을 구독할 수 있었습니다.
- **수정**
  - [사실] `84eacd3`(01-31): `@RequestAttribute("userId")`, `@RequestAttribute("userRole")`로 바꿔 필터가 넣은 값만 씁니다.
  - [사실] 테스트 연쇄 수정: `ad36d3d`(context-path 반영), `75058bc`(불필요한 mock 제거), `84eacd3`(`requestAttr` 사용), `dcfecea`(`"ROLE_USER"` → `"BASIC"`, 실제 enum 이름과 맞춤)
- **수정 후** — [사실] 신원은 토큰에서만 나옵니다.

### 3.6 SSE 연결 종료 방식과 하트비트
- **종료 방식** — [사실] `2d13c43`에서 "LOCKED" 전송 후 `Thread.sleep(1000)` → `complete()` 하던 코드를 지웠고, 4분 뒤 `fb3e02f`에서 `complete()` 메서드 자체를 지웠습니다. [추측] 리스너 스레드를 막는 sleep과 메시지 도착 전 끊김을 피하려는 의도입니다.
- **하트비트**
  - [사실] `2d498c7`(02-01): 연결마다 `newSingleThreadScheduledExecutor()`로 45초마다 heartbeat를 보냈습니다. 주석은 "Nginx 기본 타임아웃 60초보다 짧아야 함"입니다.
  - [사실] 그런데 당시 `frontend/nginx/default.conf`의 `/api`에는 이미 `proxy_read_timeout 86400s`가 있었습니다. 주석이 말하는 원인과 실제 설정이 맞지 않습니다.
  - [추측] 실제 끊김 원인은 다른 중간 계층이었거나 배포 설정이 리포지토리와 달랐을 수 있습니다.
- **이후** — [사실] 팀원 커밋 `66cd88b`(02-08)에서 중앙 `@Scheduled(fixedRate = 15000)` 하트비트와 `onError`·자기 자신일 때만 삭제(`remove(id, emitter)`) 방식으로 교체됐습니다. 연결별 스레드와 `onError` 누락 문제는 현재 해소된 상태입니다.

### 3.7 프론트 SSE 시스템 메시지 필터링 (`61c5710`)
- **문제** — [사실] 수정 전 `UserRobotSSE.ts`는 모든 메시지를 `setLastMessage`로 넘기고 heartbeat 때 `setAlertEvent("heartbeat")`를 호출했습니다. `AlertPage`에는 `heartbeat: { text: 'ping!', sound: 'ping.mp3' }` 테마가 있었습니다. `RobotsPage`도 `'ping'` 등을 여러 곳에서 따로 걸렀습니다.
- **원인** — [추측] 3.6의 45초 하트비트가 로봇 디스플레이에 주기적으로 "ping!"과 소리를 띄웠을 것입니다.
- **수정** — [사실] `ping`/`heartbeat`/`keep-alive`/`Connected...` 필터링을 hook 한 곳으로 모으고, `RobotsPage`의 중복 필터를 지웠습니다.

### 3.8 관리자 잠금 해제 요청 왕복
- **경과**
  - [사실] `7b901f8`: 요청을 `/api/admin/missions/{id}/unlock`에서 `/api/auth/unlock`(본문 `{}`)으로 바꿨습니다. 이 엔드포인트는 `@Valid VerifyPasswordRequestDto`(missionId, password)를 요구하는 사용자용입니다.
  - [사실] `0d14dfc`: 원복했습니다.
  - [사실] `94c4d63`: 백엔드 `unlockByAdmin()`의 이벤트를 `AdminUnlockRequestEvent`에서 `UserAuthSuccessEvent`로 바꿨습니다.
- **원인**
  - [사실] 두 이벤트의 MQTT 핸들러는 같은 `sendCommand(mac, "unlock", "{}")`를 보냅니다. 차이는 `UserAuthSuccessEvent`에만 SSE 리스너(사용자 알림, 관리자 broadcast)가 있다는 점입니다.
  - [추측] 원래 문제는 "관리자가 열었을 때 화면에 상태 알림이 안 뜸"이었을 가능성이 큽니다.
- **수정 후** — [사실] `AdminUnlockRequestEvent` 리스너는 남아 있지만 발행하는 곳이 없습니다(dead code).

### 3.9 vite 프록시 대상 왕복
[사실] `231d6ac` `localhost:8080` → `ce8a921` `https://i14e101.p.ssafy.io/` → `59cc4f6` 끝 `/` 제거. [추측] 마지막 슬래시 때문에 `//api/...` 경로가 만들어졌을 것입니다.

### 3.10 폴더 이름 오타
[사실] `dc236f5`가 `admin-fronted/`에 94개 파일 추가 → `4a9c0de`가 `admin-frontend/`에 같은 94개를 새로 추가(이름 변경이 아닌 복사) → `02ace91`이 `admin-fronted/` 94개 삭제.

### 3.11 3D 맵 이동 로직 반복 개편
- [사실] `8a14ac9`(lerp 0.05, `test.tsx` 539줄) → `18e60b5`(경로 50개 저장, lerp 0.02, `navigationPaths.ts`) → `d7a5b75`(Main station 이동, `activePath` 웨이포인트) → `6ad0bc4`(`robotPaths` Map, `onMoveEnd`, localStorage) → `88ecba9`(색상)
- [사실] `18e60b5`의 `useEffect` 의존성에 `mainGroupRef.current?.position.x`가 있습니다. [추측] ref 변경은 리렌더를 일으키지 않아 의도대로 실행되지 않았을 수 있습니다.

### 3.12 "롤백" 커밋 (`be5165d`)
[사실] diff는 `AlertPage.tsx`에 빈 줄 1개 추가뿐입니다. [추측] 실제 되돌리기는 작업 트리에서 직접 했을 것입니다.

---

## 4. 설계 의사결정

> 현재 HEAD `1253d73` 기준입니다. 일부 코드는 팀원이 이후 수정했으며, 해당 부분은 따로 표시했습니다.

### 4.1 SSE

**현재 구조**
- [사실] `SseEmitterRepository.java:15,19`에 `userEmitters`, `adminEmitters` 두 `ConcurrentHashMap`. 사용자는 `sendToUser`(1:1), 관리자는 `broadcastToAdmins`
- [사실] 역할은 JWT의 `role` 값을 `JwtAuthenticationFilter`가 request 속성에 넣고, 컨트롤러가 `@RequestAttribute("role")`로 받음(`SseController.java:25`)
- [사실] 재구독 시 이전 emitter를 `complete()`(`SseService.java:53-69`), 종료 콜백은 자기 자신일 때만 삭제(`:74-86`) — 팀원 `66cd88b`
- [사실] 하트비트는 `@Scheduled(fixedRate = 15000)` 하나(`SseService.java:91-113`) — 팀원 `66cd88b`
- [사실] 서비스/MQTT 수신부는 도메인 이벤트만 발행 → `UserSseNotificationHandler`/`AdminSseNotificationHandler`가 `@Async @TransactionalEventListener`로 받아 SSE 전송. 같은 이벤트를 `MqttCommandHandler`도 구독

**근거**
| 근거 | 출처 |
|---|---|
| 관리자는 관제 목적이라 전체 전송이 잦다 | [사실] 저장소 주석 "관리자는 '관제' 목적이므로 전체 브로드캐스트가 자주 일어납니다." |
| 하나의 이벤트에 여러 반응을 붙이기 위해 | [사실] `FailureCountHandler` 주석 "-> MqttCommandHandler (로봇 복귀) -> UserSseNotificationHandler (사용자 알림)" |
| DB 커밋 후에만 알림 | [사실] `MqttCommandHandler` 주석 "AFTER_COMMIT : 트랜잭션 COMMIT이 된 경우에만 동작" |
| 신원 위조 방지 | [사실] `84eacd3` (3.5) |
| nginx 버퍼링 방지 | [사실] `X-Accel-Buffering: no`(`SseController.java:30`), nginx `proxy_buffering off` |
| 하트비트 중앙화 | [사실] `66cd88b` 메시지. [추측] 연결별 스레드 증가와 정리 경합을 함께 해결 |

**선택 / 대안 / 한계**
- **선택한 방식**: 서버 메모리 emitter 맵 2개 + 도메인 이벤트 → 리스너 → SSE, 중앙 하트비트
- **대안**: WebSocket/STOMP, Redis Pub/Sub이나 브로커로 이벤트 전달(수평 확장), 폴링
- **한계**
  - [사실] emitter가 JVM 메모리에만 있음. [추측] 서버가 2대 이상이면 알림 누락, 재배포 시 전체 끊김
  - [사실] `Last-Event-ID` 미처리, 이벤트 `id`가 사용자 ID. 끊긴 동안의 이벤트 유실
  - [사실] `SseController.java:35` `userRole.name()`에 null 검사 없음
  - [사실] `AdminSseNotificationHandler.handleRobotEmergencyEvent`(`:125`)에 `@EventListener`가 없어 호출되지 않음 (`0c7a460`, 본인)
  - [사실] 관리자 리스너 대부분 `fallbackExecution` 없음. `handleRobotAssignedEvent`만 동기 `@EventListener`
  - [사실] `AsyncConfig`가 비어 있어 기본 실행기 사용
  - [사실] 프론트 훅(`UserRobotSSE.ts:16`)이 토큰을 마운트 때 한 번만 읽음. [추측] access token 만료 후 401 재연결 무한 반복 가능

### 4.2 MQTT

**현재 구조**
- [사실] Paho `MqttClient`를 직접 빈으로 구성(`MqttConfig.java`). 구독 토픽 7개 `robot/+/{register|arrived|locked|unlocked|returned|IDLE|error}`, QoS 1(`:40`, `:131`)
- [사실] `messageArrived` → `MqttSubscriberService.handleMqttMessage` → 토픽 세 번째 부분으로 `switch`
- [사실] 각 처리 함수는 `TransactionTemplate.executeWithoutResult` 안에서 미션 조회 + 도메인 이벤트 발행
- [사실] payload에 missionId가 없어 MAC + 미션 상태로 조회(`findByMacAddressAndStatus(mac, MOVING)` 등)
- [사실] 명령은 `MqttCommandHandler`가 `@Async @TransactionalEventListener(AFTER_COMMIT, fallbackExecution = true)`로 받아 `robot/{mac}/command/{action}`으로 비동기 발행
- [사실] `cleanSession(true)`, `automaticReconnect(true)`, keepAlive 60초. 테스트 프로필은 clientId에 UUID

**근거**
| 근거 | 출처 |
|---|---|
| Paho 스레드는 Spring 트랜잭션 밖 | [사실] `MqttConfig.java:115`, `MqttSubscriberService.java:41` 주석. 3.4의 교훈 반영 |
| 로봇은 missionId를 모름 | [사실] 처리 함수 주석 "Payload: {} (빈 페이로드 - 로봇은 missionId를 모름)" |
| 커밋 후에만 명령 | [사실] `MqttCommandHandler` 공통 주석. [추측] DB 롤백 시 로봇만 움직이는 불일치 방지 |
| 테스트 clientId 충돌 방지 | [사실] `MqttConfig` 주석 |
| 직접 Paho로 전환 | [사실] 팀원 `b16bd92`(02-02). [추측] 스레드·트랜잭션 경계를 명확히 하려는 의도 |

**선택 / 대안 / 한계**
- **선택한 방식**: Paho 직접 구독 + `TransactionTemplate` + 도메인 이벤트 / `AFTER_COMMIT` 리스너 → 비동기 발행
- **대안**: Spring Integration `MqttPahoMessageDrivenChannelAdapter`, payload에 상관관계 ID 추가, MQTT v5 클라이언트
- **한계**
  - [사실] clean session + 자동 재연결인데 `MqttCallbackExtended.connectComplete`가 없고, `connectionLost`는 로그만, 구독은 최초 1회. [추측, Paho 동작] **재연결 후 로봇 메시지를 받지 못할 가능성이 큼**
  - [사실] `"emergency"` case(`MqttSubscriberService.java:78`)에 `break`가 없고, `robot/+/emergency`가 `SUBSCRIBE_TOPICS`에 없어 도달 불가 (`0c7a460`)
  - [사실] 예외는 로그만 남김. 재처리·실패 저장·관리자 알림 없음
  - [사실] MAC + 상태 조회라 같은 상태 미션이 2개면 모호. [추측] QoS 1 중복 전달 시 오처리 가능
  - [사실] 발행 실패 처리와 `setDefaultQos` 없음. [추측] 기본 QoS 0으로 알고 있음(확인 필요)
  - [사실] 명령 ACK/타임아웃 없음
  - [사실] 로봇 상태 토픽(`robot/+/status`) 미구독

### 4.3 인증

**현재 구조**
- [사실] 이메일 + 4자리 비밀번호 → 2자리 인증번호(`AuthService.java:55`)를 Mattermost로 전송 → 인증번호 확인 → find-or-create → 임시 비밀번호 승격

| Redis 키 | TTL |
|---|---|
| `auth:email:code:{email}` | 5분 |
| 임시 비밀번호 | 5분 |
| `user:password:{userId}` | 24시간 |
| `refresh:{userId}` | 7일 |
| `auth:fail:count:{userId}` | 1시간 |

- [사실] Access token: `sub`=이메일, `userId`, `role` HS256 JWT. `JwtAuthenticationFilter`가 검증 후 request 속성 설정. 관리자 경로는 `AuthorizationFilter`가 DB로 역할 재확인
- [사실] Refresh token: 응답 본문에서 제외(`refreshToken(null)`, `AuthController.java:53,81`), `HttpOnly; Secure; SameSite=None` 쿠키. `/reissue`는 Redis 값 비교 후 Rotation, 불일치 시 `AUTH_005`
- [사실] 실패 잠금: 이름은 "로그인 실패"지만 실제로는 로봇 문 열기 비밀번호 실패. `unlockRequest` 실패 → `UserAuthFailedEvent` 발행 후 예외 → `FailureCountHandler`가 `@Async @EventListener`로 `INCR`(첫 실패 시 TTL 1시간) → 3회면 `MissionAbortedEvent`(로봇 복귀 + 사용자 알림) 후 초기화

**근거**
| 근거 | 출처 |
|---|---|
| refresh token을 JS에서 숨김 | [사실] 주석 "★ 보안: Body에는 Refresh Token을 주지 않음", `httpOnly(true)` |
| 탈취 감지와 Rotation | [사실] `reissue` 주석 "토큰 일치 여부 확인 (탈취 감지)", "새 토큰 생성 (Rotation)". 3.1 수정 결과 |
| 실패 처리가 롤백되면 안 됨 | [사실] `FailureCountHandler.java:25-26` "@EventListener: 트랜잭션 롤백 여부와 상관없이 무조건 실행되어야 함". `dce3d3e`에서 `@TransactionalEventListener` → `@EventListener` |
| Redis 장애가 서비스를 멈추면 안 됨 | [사실] `LoginFailCountRedisRepository.java:40` "Redis 죽었을 때 로직을 멈출 순 없으니, 0을 리턴" |
| TTL로 자동 만료 | [사실] 각 저장소 TTL 상수 |
| context-path 차이 흡수 | [사실] `getServletPath()` + `/auth/*`, `/api/auth/*` 이중 등록 (3.2) |

**선택 / 대안 / 한계**
- **선택한 방식**: 직접 만든 서블릿 필터 2개 + 무상태 Access JWT + Redis Refresh(Rotation, HttpOnly 쿠키) + 이벤트 기반 실패 카운트(fail-open)
- **대안**: Spring Security `SecurityFilterChain`, 서버 세션 + Redis Session, 동기 실패 카운트와 사전 차단, Redis 장애 시 fail-closed
- **한계**
  - [사실] 응답에 인증번호를 그대로 담음 (`AuthService.java:65`, `a275b04`). 인증번호가 2자리(90가지)이고 시도 제한 없음. [추측] 개발 편의 코드가 남은 것
  - [사실] `application.yml`에서 access 24시간, refresh도 24시간. 쿠키 `maxAge`와 Redis TTL은 7일. [추측] refresh token을 따로 두는 의미가 약함
  - [사실] 토큰 종류 구분 없음. refresh token도 `validateToken` 통과 → `Role.valueOf(null)`(`JwtUtils.java:86`) NPE
  - [사실] `/logout`은 쿠키만 삭제하고 Redis refresh token은 유지
  - [사실] `unlockRequest`에 소유권 검증이 주석만 있음(`AuthService.java:172`)
  - [사실] 3회 판정이 비동기이고 `get()` 사용처가 없어 사전 차단 없음. 키가 `userId` 기준, Redis 장애 시 fail-open
  - [사실] 화이트리스트가 두 필터에 따로 있고 `startsWith` 비교

### 4.4 3D 모니터링

**현재 구조**
- [사실] `navigationPaths.ts`: 반지름 70 원 위에 `MAIN STATION`, `STOP1`, `GATE`를 120° 간격으로 두고, 모든 조합의 시계 방향 원호 경로를 미리 계산(구간당 60개 지점)
- [사실] 실제 좌표가 아닌 SSE 이벤트로 이동 시작
  - `MissionStartedEvent`: `MAIN STATION` → 목적지 (`'a'→STOP1`, `'c'→GATE` 하드코딩, `RobotStage.tsx:693`)
  - `ReturnStartedEvent`: 가장 가까운 정거장 → `MAIN STATION`
  - `MOVE` 처리 코드는 있으나 백엔드에서 보내지 않음
- [사실] `useFrame`에서 `MOVE_SPEED(1.25) × delta` 등속 이동, `atan2` 회전 보간, 출발 전 3.5초 대기(`:73`, 주석은 "2초"), 종료 시 `onMoveEnd`가 마지막 지점 저장

| 상태 위치 | 내용 |
|---|---|
| zustand `sseStore` | `isConnected`, `lastMessage`(원문), `alertEvent` |
| `RobotsPage` 로컬 | `activeMissions`, `sseMovements` |
| `RobotStage` 로컬 | `robotPaths`, `robotCurrentCoords` |
| `localStorage` | `robotCurrentCoords`(`:644`) |
| REST `useRobotFetch` | 로봇 목록 |

**근거**
| 근거 | 출처 |
|---|---|
| 서버가 좌표를 주지 않아 이벤트로 이동 재연 | [사실] `status` 토픽 미구독, 백엔드 `MOVE` 발행 없음. [추측] 그래서 클라이언트 시뮬레이션 선택 |
| 순간이동 → 경로 이동 개선 | [사실] `8a14ac9` → `18e60b5` → `d7a5b75`·`6ad0bc4` (3.11) |
| 실제 주행 경로(좌회전 순환) 흉내 | [사실] 파일 주석 "삼각형 네비게이션 경로 시스템 (원형 호 이동 & 좌회전)", `CW_ORDER` |
| 프레임 속도와 무관한 이동 | [사실] `delta` 곱 등속. [추측] lerp의 가변 속도 회피 |
| 새로고침해도 위치 유지 | [사실] 주석 "localStorage에서 저장된 위치 복원" |
| 공유 상태만 전역 | [사실] `sseStore`를 `MainLayout`, `AlertPage`, `RealtimeActivityFeed`, `RobotsPage`가 공유 |

**선택 / 대안 / 한계**
- **선택한 방식**: 미리 계산한 원호 웨이포인트 + 이벤트 트리거 + 클라이언트 등속 이동, 전역은 zustand(원문 메시지)
- **대안**: MQTT `status` 실측 좌표를 SSE로 전달해 보간만 하는 방식, 서버가 시작 시각·경로를 보내 모든 클라이언트가 같은 기준으로 재생, 파싱된 이벤트를 store에서 리듀서로 처리
- **한계**
  - [사실] 화면 위치는 실측이 아닌 추정값. 긴급정지·지연·다른 경로 미반영
  - [사실] 위치를 브라우저 `localStorage`에 저장. [추측] 관리자·브라우저마다 다를 수 있음
  - [사실] 호출 장소 `'b'` 매핑 없음, 정거장 3개 고정
  - [사실] `sseMovements`는 추가만 되고 비워지지 않으며, 새 이벤트마다 전체를 다시 처리. [추측] 끝난 이동 재생, 메모리 증가
  - [사실] `MissionStartedEvent`는 메모리 `activeMissions`에 있어야 처리. [추측] 새로고침 후 무시됨
  - [추측] `lastMessage` 하나로 전달해 같은 틈의 여러 이벤트가 React 배칭으로 유실될 수 있음
  - [사실] `useFrame`에서 지점마다 setState → 구간당 60번 리렌더
  - [사실] `sseStore.robots`는 `setRobots` 호출처가 없어 항상 빈 배열. `robotStore`(더미 6대)는 import하는 곳 없음

### 4.5 요약

| 영역 | 설계 의도가 잘 드러난 점 | 우선순위 높은 한계 |
|---|---|---|
| SSE | 사용자/관리자 분리, 이벤트 기반 분리, emitter 정리 경합 방어 | 단일 인스턴스 메모리, 이벤트 유실, **긴급정지 리스너 어노테이션 누락** |
| MQTT | 수동 트랜잭션, 커밋 후 명령, MAC 기반 상관관계 | **재연결 후 재구독 없음**, emergency 토픽 미구독, ACK/재시도 없음 |
| 인증 | HttpOnly + Rotation, 롤백과 무관한 실패 카운트, fail-open | **인증번호 응답 노출 + 2자리**, 토큰 종류 미구분, 로그아웃 시 Redis 토큰 유지 |
| 3D | 프레임 독립 등속 이동, 미리 계산한 경로 | 실측이 아닌 **시뮬레이션 위치**, 이벤트 배열 누적, 상태 분산 |

---

## 5. 정량 근거

모든 숫자는 명령으로 직접 센 값입니다. 기준 HEAD는 `1253d73`입니다.
- **최초 작성**: 파일이나 줄을 처음 추가한 커밋의 작성자
- **현재 소유(blame)**: HEAD에 남은 줄을 마지막으로 고친 사람 (`git blame -w`)

### 5.1 요약표

| 항목 | 숫자 | 기준 |
|---|---|---|
| 커밋 | **192개** (merge 아닌 커밋 97 + merge 95) | `git log --all --author=…` |
| 활동 기간 | 2026-01-12 ~ 2026-02-08, 커밋한 날 17일 | 월별 01월 73 / 02월 119 |
| 추가/삭제 (전체) | +45,532 / −20,484 | merge 제외. 폴더 이중 추가·lockfile·에셋 포함이라 부풀려짐 |
| 추가/삭제 (소스만) | **+15,571 / −5,340** | lockfile, `public/`, `admin-fronted/`, `.lck`, `.md` 제외 |
| └ 백엔드 main | +2,603 / −656 | `backend/src/main/` |
| └ 백엔드 test | +1,740 / −767 | `backend/src/test/` |
| └ 관리자 프론트 | +11,013 / −3,889 | `admin-frontend/src/` (실험용 `test.tsx` 538줄 포함) |
| HEAD에 남은 내 코드 | 백엔드 main **1,751줄**(6,311의 28%), test **1,190줄**, 관리자 프론트 **6,999줄**(7,191의 97%) | blame |
| 테스트 클래스 (최초 작성) | **11개** HEAD에 남음 (+3개 이후 삭제) | 파일 최초 추가 |
| 테스트 메서드 | **35개** (위 11개 클래스의 `@Test`) / 현재 소유 기준 **36개**, 14개 파일 | `@Test` 개수, blame |
| API 엔드포인트 (최초 작성) | **14개** = 운영 8 + 테스트·디버그 6 | `git log -S`. blame과 결과 동일 |
| 도메인 이벤트 클래스 (최초 작성) | **10개** / 전체 20개 | 파일 이름 기준 최초 추가 |
| SSE 이벤트 타입 | 서버 전송 **15종** + 시스템 2종(CONNECT, heartbeat). 처리 메서드가 내 코드인 것 **8종** (동작하는 것 7종) | 리스너 메서드 blame |
| MQTT | 수신 토픽 7개 + 명령 4종 모두 팀원 추가 / 내 코드는 수신 처리 3개, 명령 리스너 2개 | blame, `git log -S` |
| 관리자 프론트 컴포넌트 | `.tsx` 컴포넌트 **20개** + 페이지 **5개** 모두 최초 작성. 실제 import되는 것 **15개** (컴포넌트 10 + 페이지 5) | 파일 최초 추가, import 검색 |

### 5.2 테스트 상세

| 클래스 (최초 작성) | `@Test` |
|---|---:|
| UserSseNotificationHandlerTest | 7 |
| AuthControllerTest | 5 |
| AuthServiceTest | 5 |
| SseServiceTest | 5 |
| JwtUtilsTest | 3 |
| SseEmitterRepositoryTest | 3 |
| AuthVerifyControllerTest | 2 |
| AuthVerifyServiceTest | 2 |
| MattermostClientTest | 2 |
| SseControllerTest | 1 |
| FailureCountHandlerTest | 0 (`@Test` 없음) |

- 팀원 파일에 추가한 테스트: MqttSubscriberServiceTest 3개, AdminNotificationHandlerTest 1개
- 삭제된 내 테스트 파일: SseNotificationHandlerTest, UserLockerControllerTest, UserLockerServiceTest
- 저장소 전체: 테스트 파일 44개, `@Test` 260개

### 5.3 API 엔드포인트 상세 (14개, 최초 작성 커밋)

| 구분 | 엔드포인트 | 최초 커밋 |
|---|---|---|
| 운영 | `POST /auth/request` | `e68d076` |
| 운영 | `POST /auth/verify` | `e68d076` |
| 운영 | `POST /auth/reissue` | `e68d076` |
| 운영 | `POST /auth/logout` | `478b717` |
| 운영 | `POST /auth/unlock` | `1e62133` |
| 운영 | `POST /auth/lock` | `1e62133` |
| 운영 | `GET /sse/subscribe` | `6711109` |
| 운영 | `POST /admin/missions/{missionId}/store` | `d193d90` |
| 테스트 | `GET /sse/send/{userId}` | `b1c69a4` |
| 테스트 | `GET /test/sse/start` | `702a584` |
| 테스트 | `GET /test/sse/return` | `702a584` |
| 테스트 | `GET /test/sse/robot-assigned/all` | `1817d77` |
| 테스트 | `GET /test/sse/assign/first` | (blame 본인) |
| 테스트 | `GET /test/sse/assign/recall` | (blame 본인) |

- 전체 37개 중 나머지 23개는 팀원 코드
- `GET /me/lockers`는 본인이 처음 만들었지만(`54e09c8`) 팀원이 삭제(`ea21060`) 후 재구현(`53aa6bc`)해서 제외

### 5.4 SSE 이벤트 / 도메인 이벤트 상세
- 처리 메서드가 내 코드인 SSE 이벤트 8종: RobotAssigned, MissionStarted, RobotArrival, UserAuthSuccess, MissionUnlocked, MissionAborted, MissionLocked, RobotEmergency(미동작)
  - 사용자 핸들러 9개 중 7개, 관리자 핸들러 11개 중 2개
- 최초 작성한 이벤트 클래스 10개

| 이벤트 | 최초 커밋 |
|---|---|
| RobotArrivalEvent, UserAuthSuccessEvent, UserAuthFailedEvent, MissionLockedEvent, MissionAbortedEvent | `ab34b69` |
| MissionLockRequestEvent | `1e62133` |
| MissionUnlockedEvent | `f19ac73` |
| RobotReturnedAdminEvent | `22fc20e` |
| MissionStoredEvent | `0d3f9df` |
| RobotEmergencyEvent | `0c7a460` |

### 5.5 MQTT 상세
- 수신 토픽 7개 최초 추가: `register`/`arrived`/`error` = `07d35ab`, `locked`/`unlocked`/`returned`/`IDLE` = `b16bd92` (모두 김정훈 님)
- 실제 쓰이는 명령 4종: `dispatch`, `lock`, `unlock`, `return`. `stop`은 정의만 있고 호출처 없음
- 내 코드 (blame): 수신 처리 `handleLocked`, `handleUnlocked`, `handleEmergency`(미도달) / 명령 리스너 `handleMissionLocked`, `handleAdminUnlockRequest`

### 5.6 프론트 컴포넌트 상세
- 관리자 프론트 `.ts`/`.tsx` 42개 중 41개가 최초 작성 본인 (예외: `sseStore.ts`, 김정훈 님 `7cfe1c3`)
- import되지 않는 파일 11개: `dashboard/` 8개 전부, `Sidebar`, `RobotReturnModal`, `test.tsx`
- 첫 커밋 `dc236f5`가 src 파일 36개 포함 94개 파일, +15,124줄을 한 번에 추가

### 5.7 경험 정리 문장 예시 (위 숫자만 사용)
- "4주간 커밋 192회. 백엔드 약 4.3천 줄(main +2.6k, test +1.7k), 관리자 프론트 +11k줄 작성"
- "인증·SSE·잠금 API 8개 구현, 테스트 클래스 11개·테스트 메서드 35개 작성"
- "도메인 이벤트 10종을 설계해 SSE 알림(사용자 7종)과 MQTT 로봇 명령에 연결"
- "관리자 관제 콘솔을 단독 개발(현재 코드의 97%): 3D 맵과 실시간 SSE 모니터링"

### 5.8 재현용 명령 (주요)

```bash
A='--author=he5298@naver.com --author=KangHeejeongg'
git log --all $A --format=%H | sort -u | wc -l                 # 커밋 수
git log --all --no-merges $A --numstat --format=                # 라인 수 원본
git blame -w --line-porcelain HEAD -- <file> | grep '^author-mail' | sort | uniq -c   # 현재 소유
git log --all --diff-filter=A --reverse --format='%ae %h' --name-only                # 최초 작성
git log --reverse --format='%ae %h' -S'@PostMapping("/request")' -- backend/src/main # 엔드포인트 최초 커밋
```
