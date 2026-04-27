<div align="center">

# Sound of Slash — 포트폴리오

[![Steam](https://img.shields.io/badge/Steam-Sound_of_Slash-1b2838?style=flat-square&logo=steam&logoColor=white)](https://store.steampowered.com/app/3036410/Sound_of_Slash/)
[![Studio](https://img.shields.io/badge/Ussistant_Studio-Website-blueviolet?style=flat-square)](https://www.ussistantstudio.com/)

← [게임 소개로 돌아가기](./README.md)

<br>

| 참여 기간 | 총 커밋 | 구현 기능 | 해결 버그 | 오프라인 전시 |
|:---------:|:-------:|:---------:|:---------:|:-------------:|
| **2024.04 – 2026.04** | **587커밋** | **61개** | **12건** | **4회** |

</div>

---

## 📌 내 역할

| 항목 | 내용 |
|------|------|
| **역할** | 인게임 클라이언트 개발 · 채보 에디터 메인테이너 · Firebase 백엔드 연동 |
| **Tech Stack** | Unity C# · Firebase (Firestore / Cloud Functions / Auth) · Steamworks.NET · Node.js (TypeScript) |

---

## 🔄 Before / After — 2년간 내가 만든 변화

<table width="100%">
  <tr>
    <td align="center" width="50%">
      <img src="https://raw.githubusercontent.com/Kkackit02/Ussistant-Studio-Archive/main/SoundOfSlash/images/ingame_early.png" width="100%"/><br/>
      <sub>2024-05 합류 직후 — UI 없는 프로토타입</sub>
    </td>
    <td align="center" width="50%">
      <img src="https://raw.githubusercontent.com/Kkackit02/Ussistant-Studio-Archive/main/SoundOfSlash/images/ingame_1.png" width="100%"/><br/>
      <sub>2026-04 현재 — BeatHelper · 몬스터 노트 · FEVER 시스템</sub>
    </td>
  </tr>
</table>

| | **2024.04 합류 시점** | **2026.04 현재** |
|---|---|---|
| **노트 타입** | 일반 · 몬스터 2종 | Hold · Rolling · LongRange · EvilWood 포함 **6종** |
| **판정 시스템** | x좌표 픽셀 기반 → 해상도/배속에 따라 판정 불일치 | ms 기반 → 배속 1x~6x, 모든 해상도에서 동일한 체감 |
| **경제 시스템** | 없음 (로컬 저장) | Firebase 서버 권위 모델 (BattlePass · Mission · 리더보드) |
| **결제** | 없음 | Steamworks SDK 소액결제 · Gem 구매 시스템 |
| **채보 제작** | 수작업 100% | MIDI + DNN v3.0 자동 생성 파이프라인 |
| **보안** | 클라이언트 연산 → 메모리 해킹 가능 | 서버 권위 모델 → 클라이언트 조작 불가 |
| **오프라인** | 미지원 | Local Cache → 재연결 시 자동 동기화 |

---

## 🛠 핵심 구현

### 1. 판정 시스템 재설계 — x좌표 → ms 기반 (v0.3.13)

리듬 게임에서 판정의 일관성은 핵심 품질 지표입니다.

**문제:** x좌표 픽셀 거리로 판정 → 배속 변경 시 같은 타이밍도 다른 판정, 해상도마다 체감 불일치

**해결:** 노트 도달 예정 시간 vs 실제 입력 시간의 차이를 ms 단위로 계산

| | Before | After |
|---|---|---|
| **판정 기준** | 화면 x좌표 픽셀 거리 | 시간 차이 (ms) |
| **배속 1x → 6x** | 판정 범위가 6배 달라짐 | 동일한 판정 체감 유지 |
| **30fps / 144fps** | 프레임마다 좌표 오차 발생 | 프레임레이트 독립적 |

추가 구조 수정: 리스트 기반 판정 → Collider 기반으로 전환.
Miss 1회 발생 시 리스트 상태가 꼬여 이후 전부 Miss되는 치명적 버그 근본 해결.

<table width="100%">
  <tr>
    <td align="center" width="50%">
      <img src="https://raw.githubusercontent.com/Kkackit02/Ussistant-Studio-Archive/main/SoundOfSlash/images/ingame_1.png" width="100%"/><br/>
      <sub>인게임 — BeatHelper 링 · 몬스터 노트 · 28 콤보</sub>
    </td>
    <td align="center" width="50%">
      <img src="https://raw.githubusercontent.com/Kkackit02/Ussistant-Studio-Archive/main/SoundOfSlash/images/scoreboard.png" width="100%"/><br/>
      <sub>결과 화면 — PERFECT · GREAT · GOOD · MISS 판정 분류</sub>
    </td>
  </tr>
</table>

---

### 2. 신규 노트 타입 4종 — 독립 설계

Steam Early Access 출시 후 지속적인 게임플레이 확장. 각 노트 타입은 **판정 로직 · 애니메이션 · JSON 포맷**을 독립 설계해 기존 시스템과 충돌 없이 추가.

| 노트 타입 | 도입 | 입력 방식 |
|-----------|------|-----------|
| **Hold Note** | 2024-09 | 키 유지 · 홀드 길이를 채보 `length` 필드로 관리 |
| **Rolling Note** | 2024-10 | 연속 입력 · 타이밍 누적 판정 |
| **Long Range Note** (EvilWood) | 2024-10 | 좌우에서 EvilWood 캐릭터가 등장해 포물선 노트 발사 · 프로젝트 내 명칭 혼용 |
| **Missile Note** | 2025-10 | 부엉이 몬스터가 화면에 나타나 플레이어를 향해 노트 발사 |

<table width="100%">
  <tr>
    <td align="center" width="33%">
      <img src="https://raw.githubusercontent.com/Kkackit02/Ussistant-Studio-Archive/main/SoundOfSlash/images/tutorial.png" width="100%"/><br/>
      <sub>튜토리얼 — 각 노트 타입 안내</sub>
    </td>
    <td align="center" width="33%">
      <img src="https://raw.githubusercontent.com/Kkackit02/Ussistant-Studio-Archive/main/SoundOfSlash/images/tutorial_2.png" width="100%"/><br/>
      <sub>"Hit Q and P to SLASH"</sub>
    </td>
    <td align="center" width="33%">
      <img src="https://raw.githubusercontent.com/Kkackit02/Ussistant-Studio-Archive/main/SoundOfSlash/images/tutorial_3.png" width="100%"/><br/>
      <sub>1 COMBO — 첫 노트 처치 연출</sub>
    </td>
  </tr>
</table>

---

### 3. Server-Authoritative 경제 시스템 구축

**문제:** 클라이언트에서 아이템 가격 계산 → 메모리 해킹으로 무료 구매 가능한 구조

**해결:** 클라이언트는 "의도"만 전달, 서버가 모든 계산·검증·지급을 처리

```
Client: purchaseItem({ itemId, userId })
  Cloud Function:
    ① 서버 GAME_ITEMS에서 실제 가격 조회  (클라이언트 가격 신뢰 X)
    ② 잔액 검증
    ③ Firestore 트랜잭션 — 차감 + 지급 + 이력 기록 (원자적 처리)
  Client: 성공/실패 수신
```

`transactionId` 기반 Idempotency → 네트워크 재시도 시 이중 지급 완전 방지.

<table width="100%">
  <tr>
    <td align="center" width="33%">
      <img src="https://raw.githubusercontent.com/Kkackit02/Ussistant-Studio-Archive/main/SoundOfSlash/images/shop_skin.png" width="100%"/><br/>
      <sub>상점 — 스킨 탭 (Rune · Gem 재화)</sub>
    </td>
    <td align="center" width="33%">
      <img src="https://raw.githubusercontent.com/Kkackit02/Ussistant-Studio-Archive/main/SoundOfSlash/images/shop_note.png" width="100%"/><br/>
      <sub>상점 — 노트 스킨 탭</sub>
    </td>
    <td align="center" width="33%">
      <img src="https://raw.githubusercontent.com/Kkackit02/Ussistant-Studio-Archive/main/SoundOfSlash/images/shop_bg.png" width="100%"/><br/>
      <sub>상점 — 배경 탭 (Desert_Stage4, 500 Runes)</sub>
    </td>
  </tr>
</table>

---

### 4. BattlePass 시스템 — 설계부터 UX 개편까지

| 시점 | 작업 |
|------|------|
| 2025-07 | 시즌 구조 · 티어 설계 · 뼈대 구현 |
| 2025-09 | 티어별 보상 지급 · Firebase 연동 |
| 2025-12 | Steam IAP 연동 프리미엄 패스 구매 |
| 2026-02 | 진행도 UI · 보상 애니메이션 **전면 UX 개편** |

서버 권위 모델 적용 — 보상 지급은 Cloud Function에서만 처리, 클라이언트 조작 불가.

<table width="100%">
  <tr>
    <td align="center">
      <img src="https://raw.githubusercontent.com/Kkackit02/Ussistant-Studio-Archive/main/SoundOfSlash/images/battlepass.png" width="100%"/><br/>
      <sub>BattlePass — Lv.3 · 프리미엄/무료 2트랙 · Rune·Gem 보상</sub>
    </td>
  </tr>
</table>

---

### 5. Steam IAP 파이프라인 (2025-12)

```
① Steam Overlay 결제
② OnMicroTxnAuthorizationResponse 콜백 → OrderID 수신
③ Cloud Function → Steam Web API 서버 검증
④ "Approved" 확인 → Gems 지급
⑤ Finalize + Firestore 이력 기록
```

**엣지 케이스:** Finalize 후 네트워크 단절 → 재로그인 시 미지급 건 자동 복구.

---

### 6. Mission 시스템 — 3단계 진화

| 단계 | 구현 | 시점 |
|------|------|------|
| 1차 | 로컬 미션 트래킹 (클라이언트 전용) | 2024 하반기 |
| 2차 | Firebase 연동 · 서버 동기화 추가 | 2025-09 |
| 3차 | Server-Authoritative 모델로 **완전 재설계** | 2026-02 |

---

### 7. Offline / Online 하이브리드 시스템 (2025-10)

```
Online  → 결과 즉시 Firebase 저장 + 리더보드 업데이트
Offline → Local Cache 임시 저장 + 서버 기능(상점/BattlePass) 비활성화
재연결  → 캐시 일괄 업로드 후 초기화
```

---

### 8. 서버 모듈화 — God Object 해체

1,245줄 단일 파일 → 기능별 분리:

```
index.ts
├── auth.ts       — Steam 인증 / 신규 유저 생성
├── economy.ts    — 재화 · 아이템 구매
├── battlepass.ts — BattlePass 보상 지급
├── missions.ts   — 미션 진행도 검증
├── admin.ts      — 어드민 도구
└── helpers.ts / types.ts
```

---

### 9. Discord 운영 봇 (2026-02)

```
/user <uid>         → 유저 정보 조회
/grant <uid> <item> → 아이템 직접 지급
/reset <uid>        → 계정 초기화
```

서버 에러 → `#server-alerts` 자동 알림. Firebase 콘솔 없이 실시간 모니터링.

---

### 10. 배경(BG) 시스템 — ScriptableObject + 패럴랙스

7종 스테이지(Forest · Desert · Ocean · Horror · Iceland · Toyland · Volcano)를 ScriptableObject 기반으로 관리.

```
Layer 1: Ground
Layer 2: ForeGround   ← 레이어별 독립 속도
Layer 3: MidGround    ← BgScrollTrigger 좌우 연동
Layer 4: BackGround
```

`Resources.LoadAll<BGData>("BGSO")`로 런타임 동적 로드. 프로필에서 선택한 배경이 인게임에 즉시 동기화.

<table width="100%">
  <tr>
    <td align="center" width="33%">
      <img src="https://raw.githubusercontent.com/Kkackit02/Ussistant-Studio-Archive/main/SoundOfSlash/images/loading.png" width="100%"/><br/>
      <sub>로딩 화면 — 스테이지별 아트워크 배경</sub>
    </td>
    <td align="center" width="33%">
      <img src="https://raw.githubusercontent.com/Kkackit02/Ussistant-Studio-Archive/main/SoundOfSlash/images/profile.png" width="100%"/><br/>
      <sub>프로필 — 배경(Background) 스테이지 선택</sub>
    </td>
    <td align="center" width="33%">
      <img src="https://raw.githubusercontent.com/Kkackit02/Ussistant-Studio-Archive/main/SoundOfSlash/images/ingame_2.png" width="100%"/><br/>
      <sub>인게임 — Horror 스테이지 패럴랙스 배경</sub>
    </td>
  </tr>
</table>

---

### 11. BeatHelper — 박자 가이드 시각화 (v0.3.22)

osu! 스타일 비트 링으로 노트 타이밍을 시각화. 플레이어 진입 장벽 완화.

<table width="100%">
  <tr>
    <td align="center" width="50%">
      <img src="https://raw.githubusercontent.com/Kkackit02/Ussistant-Studio-Archive/main/SoundOfSlash/images/ingame_1.png" width="100%"/><br/>
      <sub>BeatHelper — 흰 원형 링이 노트 타이밍 가이드 (Forest 스테이지)</sub>
    </td>
    <td align="center" width="50%">
      <img src="https://raw.githubusercontent.com/Kkackit02/Ussistant-Studio-Archive/main/SoundOfSlash/images/music_selection.png" width="100%"/><br/>
      <sub>음악 선택 — 2Track / 4Track · 난이도 · 모드 선택</sub>
    </td>
  </tr>
</table>

---

### 12. BackToBack Monster 시스템 (v0.3.14)

특수 몬스터 연속 등장 메카닉. 에디터에서 BackToBack 노트 타입으로 채보에 직접 배치 가능.

<table width="100%">
  <tr>
    <td align="center" width="50%">
      <img src="https://raw.githubusercontent.com/Kkackit02/Ussistant-Studio-Archive/main/SoundOfSlash/images/music_selection_2.png" width="100%"/><br/>
      <sub>음악 선택 — HopeYouShine · 4Track · EASY LOCKED</sub>
    </td>
    <td align="center" width="50%">
      <img src="https://raw.githubusercontent.com/Kkackit02/Ussistant-Studio-Archive/main/SoundOfSlash/images/mode_selection.png" width="100%"/><br/>
      <sub>모드 선택 — Training · Story · Challenge</sub>
    </td>
  </tr>
</table>

---

### 13. 글로벌 리더보드 + 플레이어 아이콘 시스템 (2026-02)

- Firebase Firestore 기반 전 세계 점수 순위. 오프라인 결과는 재연결 후 자동 반영.
- 플레이어 아이콘 획득/장착 상태 서버 동기화.

<table width="100%">
  <tr>
    <td align="center" width="50%">
      <img src="https://raw.githubusercontent.com/Kkackit02/Ussistant-Studio-Archive/main/SoundOfSlash/images/profile_skin.png" width="100%"/><br/>
      <sub>프로필 — 캐릭터 스킨 선택 (1P Infernal · 2P Normal)</sub>
    </td>
    <td align="center" width="50%">
      <img src="https://raw.githubusercontent.com/Kkackit02/Ussistant-Studio-Archive/main/SoundOfSlash/images/lobby.png" width="100%"/><br/>
      <sub>로비 메인 화면</sub>
    </td>
  </tr>
  <tr>
    <td align="center" width="33%">
      <img src="https://raw.githubusercontent.com/Kkackit02/Ussistant-Studio-Archive/main/SoundOfSlash/images/player_normal.png" width="100%"/><br/>
      <sub>Normal 스킨</sub>
    </td>
    <td align="center" width="33%">
      <img src="https://raw.githubusercontent.com/Kkackit02/Ussistant-Studio-Archive/main/SoundOfSlash/images/player_desert.png" width="100%"/><br/>
      <sub>Desert 스킨</sub>
    </td>
    <td align="center" width="33%">
      <img src="https://raw.githubusercontent.com/Kkackit02/Ussistant-Studio-Archive/main/SoundOfSlash/images/player_infernal.png" width="100%"/><br/>
      <sub>Infernal 스킨</sub>
    </td>
  </tr>
</table>

---

### 14. MIDI + DNN v3.0 채보 자동 생성 (2026-04)

```
MIDI 파일 → MIDI Parser(음표 추출) → DNN v3.0(리듬 패턴 예측)
→ SOS JSON 포맷 변환 → 에디터에서 검토·수정
```

---

## 🐛 주요 버그 해결 (12건)

| # | 버그 | 발생 시점 | 원인 | 해결 |
|---|------|-----------|------|------|
| 1 | BGList NullReference | 2024-06 | 인스펙터 참조 끊김 | `Resources.Load()` 동적 로드 전환 |
| 2 | 에디터-인게임 타이밍 불일치 | 2024-07 | ms/seconds 단위 혼용 | `× 0.001` 변환 처리 추가 |
| 3 | 모든 노트가 몬스터로 스폰 | 2024-07 | JSON 파싱 타입 분기 오류 | 타입 분기 로직 수정 |
| 4 | ESC 중 Fever 발동 | 2024-07 | 일시정지 상태 미처리 | Pause 상태 확인 조건 추가 |
| 5 | Combo 애니메이션 2개 동시 실행 | 2024-08 | 인스턴스 2개 존재 | 단일 인스턴스 관리 |
| 6 | Grade FX/SFX 미재생 | 2024-08 | 사운드 초기화 순서 문제 | 초기화 완료 후 연출 실행 |
| 7 | Fade 무한루프 | 2024-08 | 종료 조건 누락 | 반복 방지 플래그 추가 |
| 8 | MISS 후 Fatal Error | 2024-12 | 판정 범위 경계값 오류 | 상태 처리 + 범위 수정 |
| 9 | 해상도·배속별 판정 불일치 | 2025-02 | x좌표 기반 판정 구조적 한계 | ms 기반 판정으로 전환 |
| 10 | Firebase 직렬화 오류 | 2026-03 | 익명 타입 직렬화 불가 | 명시적 클래스 정의 |
| 11 | Fever 중 네트워크 팝업 | 2026-03 | Fever 상태 미처리 | Fever 활성 여부 조건 추가 |
| 12 | BattlePass 동기화 오류 | 2026-02 | 서버-클라이언트 매핑 불일치 | 데이터 매핑 로직 수정 |

---

## 🎪 오프라인 전시 (4회)

| 전시명 | 기간 | 빌드 |
|--------|------|------|
| **BIC** (Busan Indie Connect) | 2024-08 | v0.1.32 |
| **CBF** (콘텐츠 비즈니스 포럼) | 2024-10-22~24 | v0.3.3 |
| **G-STAR 2024** | 2024-11-13~15 | v0.3.x |
| **G-STAR 2025** | 2025-11 | EvilWood 노트 포함 |

---

## 📈 빌드 이력

### 주요 마일스톤

| 버전 | 날짜 | 주요 내용 |
|------|------|-----------|
| v0.1.2 | 2024-06-16 | 미션 · Retry · Credit 씬 · 효과음 |
| v0.1.9 | 2024-07-08 | Calibration · Key Setting · Easy 모드 |
| v0.1.18 | 2024-08-02 | Offset 적용 방식 전면 개선 |
| v0.1.24 | 2024-08-11 | 스킨 선택 기능 구현 |
| v0.1.31 | 2024-08-19 | Menu Tab UX 개편 · 피버 시스템 개편 |
| **v0.1.32** | **2024-08-22** | **⭐ BIC 전시 최종 빌드** |
| **v0.2.0** | **2024-09-02** | **⭐ Steam Early Access 정식 출시** |
| **v0.3.0** | **2024-10-14** | **Hold · Rolling · Long Range(EvilWood) 노트 · 에디터 JSON 호환** |
| v0.3.4 | 2024-10-22 | ⭐ CBF 전시 빌드 · Horror FX · 전 콘텐츠 해금 |
| v0.3.11 | 2025-01-04 | Long Range 연동 · Profile 배경/스킨/노트 선택 |
| **v0.3.13** | **2025-02-15** | **ms 기반 판정 시스템 전환** |
| v0.3.14 | 2025-03-01 | 콤보 UI 개편 · 피버 배수 결과 표시 |
| v0.3.21 | 2025-04-03 | 튜토리얼 완성 |
| v0.3.23 | 2025-05-05 | 안정화 · Tutorial 스킵/Task 시스템 |
| v0.3.27 | 2025-10-18 | 상점 · 배틀패스 첫 적용 |
| v0.3.31 | 2025-11-10 | ⭐ G-STAR 2025 준비 · 노트 버그 수정 |
| v0.3.36 | 2026-01-05 | 2P 모드 · 포물선 노트 · 배틀패스 |
| **v0.4.0** | **2025-02-01** | **에디터 전체 리팩토링 · Grid 엔진 재설계** |
| v0.4.5 | 2025-04-14 | BeatSnap Divisor · BPM 변경 JSON 반영 |
| v0.4.7 | 2025-10-21 | Missile Note 추가 |

<details>
<summary>전체 빌드 이력 펼치기 (65개)</summary>

#### 인게임

| 버전 | 날짜 | 내용 |
|------|------|------|
| v0.1.1 | 2024-06-13 || AUTO Mode 빌드 |
| v0.1.2 | 2024-06-16 || 미션 기능 · Retry · Credit 씬 · 효과음 추가 |
| v0.1.5 | 2024-06-20 || Horror 배경 적용 |
| v0.1.6 | 2024-06-24 || 버그 수정 |
| v0.1.7 | 2024-06-26 || 로비 키보드 조작 · BG/NOTE 선택 저장 |
| v0.1.8 | 2024-07-02 || Setting UI (BGM/SFX) · Key 설정 추가 |
| v0.1.9 | 2024-07-08 || Calibration · Key Setting · Easy 모드 3곡 |
| v0.1.10 | 2024-07-13 || Ghost FX 버그 수정 · ToyLand BG · 이전 곡 기억 |
| v0.1.12 | 2024-07-19 || Offset 적용 · 콤보골렘 모션 수정 |
| v0.1.13 | 2024-07-21 || (에디터 offset 단위 불일치 임시 빌드) |
| v0.1.14 | 2024-07-22 || Key binding 초기값 · 키 설정창 개선 |
| v0.1.18 | 2024-08-02 || Offset 적용 방식 전면 개선 · Note Line offset |
| v0.1.19 | 2024-08-07 || 새 곡 추가 (HEAVYgiant 시리즈) |
| v0.1.20 | 2024-08-07 || 콤보 애니메이션 개선 · 배속 0.5 단위 |
| v0.1.21 | 2024-08-09 || 새 곡 추가 (BraveHeart 등) · anim 버그 수정 |
| v0.1.22 | 2024-08-09 || 콤보 애니메이션 버그 수정 |
| v0.1.23 | 2024-08-10 || 콤보 애니메이션 안정화 |
| v0.1.24 | 2024-08-11 || 스킨 선택 기능 구현 |
| v0.1.28 | 2024-08-15 || SFX 최신 파일 교체 |
| v0.1.31 | 2024-08-19 || Menu Tab UX 변경 · 피버 활성 조건 개편 · 4key 곡 추가 |
| v0.1.32 | 2024-08-22 || 콤보골렘 애니메이션 버그 수정 **(BIC 전시 빌드)** |
| v0.2.0 | 2024-09-02 || ⭐ **Steam Early Access 정식 출시** |
| v0.2.1 | 2024-09-07 || 출시 후 핫픽스 |
| v0.2.6 | 2024-09-29 || HP Bar 레이어 · 배속 버그 수정 · 곡 추가 |
| v0.3.0 | 2024-10-14 || Hold · Rolling · Long Range 노트 · 에디터 JSON 호환 |
| v0.3.4 | 2024-10-22 || Horror FX · 누락 곡 추가 · 전 콘텐츠 해금 **(CBF 전시)** |
| v0.3.7 | 2024-11-10 || MusicSelection 이미지 개선 |
| v0.3.11 | 2025-01-04 || Long Range Note 연동 · Profile 배경/스킨/노트 선택 |
| v0.3.12 | 2025-02-04 || new_Profile 완성 · 폰트 변경 · 캐릭터 애니메이션 |
| v0.3.13 | 2025-02-15 || ms 기반 판정 전환 · 노트 이미지 수정 · UI 버그 수정 |
| v0.3.14 | 2025-03-01 || 콤보 UI 개편 · 피버 배수 결과 표시 |
| v0.3.16 | 2025-03-09 || 연타 노트 패치 · Output 파일 정리 |
| v0.3.17 | 2025-03-15 || 점수 시스템 수정 · 타격 이펙트 · Chain 이미지 버그 수정 |
| v0.3.18 | 2025-03-17 || 정확도 수치 조정 · 애니메이션 자연스럽게 |
| v0.3.19 | 2025-03-19 || JSON 버그 수정 · 전 곡 정상 로드 |
| v0.3.20 | 2025-03-24 || Calibration 수정 · 전 곡 앨범아트 추가 |
| v0.3.21 | 2025-04-03 || 튜토리얼 완성 · Profile Click Sound |
| v0.3.22 | 2025-04-14 || 튜토리얼 버그 수정 · Hold 노트 버그 수정 |
| v0.3.23 | 2025-05-05 || 안정화 · Tutorial 스킵/Task 시스템 추가 |
| v0.3.27 | 2025-10-18 || 상점 · 배틀패스 첫 적용 |
| v0.3.31 | 2025-11-10 || 일반 노트 몬스터 변환 버그 수정 · Hold 체인 길이 버그 수정 |
| v0.3.33 | 2025-11-13 || Slider Handle 개선 · Breath Effect |
| v0.3.34 | 2025-11-25 || 박쥐 텍스처 변경 · Missile/포물선 노트 스킨 변경 · 속도 고정 |
| v0.3.35 | 2025-12-05 || 콤보 몬스터 Idle 애니메이션 · Missile 속도 = 일반 × 2 |
| v0.3.36 | 2026-01-05 || 2P 모드 추가 · 포물선 노트 개선 · 배틀패스 |

#### 에디터

| 버전 | 날짜 | 내용 |
|------|------|------|
| v0.1.3 | 2024-06-30 || 배속 그리드 반영 · 단축키 (50%) |
| v0.1.4 | 2024-07-03 || Offset 음수 반영 |
| v0.1.11 | 2024-07-20 || BPM 값 JSON 오류 재수정 |
| v0.2.4 | 2024-08-06 || 노트 이동속도 BPM 연동 오류 수정 |
| v0.2.8 | 2024-08-09 || 그리드 이동속도 수정 · BPM 수정 UI |
| v0.2.13 | 2024-08-28 || 이전 노래 데이터 추가 |
| v0.2.14 | 2024-09-07 || 곡 15곡 추가 |
| v0.2.15 | 2024-09-08 || MultiSelection 범위 선택 · Delete 기능 |
| v0.2.16 | 2024-09-11 || 노래 파일 불러오기 구조 개선 |
| v0.2.17 | 2024-09-16 || 비트맵 복제 추가 |
| v0.3.1 | 2024-10-13 || M2(원거리) · M3(난타) 노트 추가 |
| v0.3.2 | 2024-10-21 || 신규 노트 타 비트맵 영향 버그 수정 |
| v0.3.3 | 2024-10-28 || 연타 노트 JSON 오류 수정 |
| v0.3.5 | 2024-11-21 || 난타 노트 오류 수정 |
| **v0.4.0** | **2025-02-01** || **전체 리팩토링 · BPM/OFFSET 소수점 · Grid 엔진 재설계** |
| v0.4.1 | 2025-02-01 || 에디터 아이콘 · 사운드 버그 수정 · Holding/BackToBack JSON 연동 |
| v0.4.3 | 2025-03-10 || 전 곡 Output 파일 교체 |
| v0.4.4 | 2025-03-20 || 튜토리얼 곡 적용 |
| v0.4.5 | 2025-04-14 || BeatSnap Divisor 버그 수정 · BPM 변경 JSON 반영 · combo 노트 수정 |
| v0.4.7 | 2025-10-21 || Missile Note 추가 |

</details>

---

## 📊 개발 흐름

| 시기 | 주요 작업 |
|------|-----------|
| 2024-04~05 | 팀 합류 · BGManager · 프로필 씬 · 플레이어 애니메이션 |
| 2024-06 | Steam 첫 심사 제출 (v0.1.0) |
| 2024-07~08 | BIC 전시 · 판정/JSON 버그 수정 · 사운드/키 바인딩 시스템 |
| **2024-09-02** | **⭐ Steam Early Access 정식 출시 (v0.2.0)** |
| 2024-09~10 | Hold · Rolling · LongRange 노트 설계·구현 |
| **2024-11** | **⭐ G-STAR 2024 전시** |
| 2024-12 | 노트 판정 범위 버그 수정 (v0.3.10) |
| 2025-01~02 | **에디터 v0.4.0 메인테이너 인수 · ms 기반 판정 전환 · BackToBack Monster** |
| 2025-05 | BeatHelper 구현 |
| 2025-07~09 | Firebase 연동 · BattlePass · Mission 시스템 |
| **2025-11** | **⭐ G-STAR 2025 전시** · EvilWood 노트 추가 |
| 2025-12~2026-01 | Steam IAP · Gem 구매 · 오프라인 모드 |
| 2026-02~03 | BattlePass UX 개편 · Mission 재설계 · 글로벌 리더보드 · Discord 봇 |
| 2026-04 | **MIDI + DNN v3.0 채보 자동 생성** |
