<div align="center">

<img src="https://cdn.akamai.steamstatic.com/steam/apps/3036410/header.jpg" width="800"/>

# Sound of Slash

[![Steam](https://img.shields.io/badge/Steam-Sound_of_Slash-1b2838?style=flat-square&logo=steam&logoColor=white)](https://store.steampowered.com/app/3036410/Sound_of_Slash/)
[![Early Access](https://img.shields.io/badge/Early_Access-2024.08.31-f4a42a?style=flat-square)](https://store.steampowered.com/app/3036410/)
[![Rating](https://img.shields.io/badge/Steam_Rating-90%25_Positive-brightgreen?style=flat-square&logo=steam)](https://store.steampowered.com/app/3036410/)
[![Ussistant Studio](https://img.shields.io/badge/Ussistant_Studio-Website-blueviolet?style=flat-square)](https://www.ussistantstudio.com/)

**리듬 액션 게임 · PC (Steam) · Early Access**

> 8개의 지형으로 나뉜 세계. 방랑자 G가 음악에 맞춰 적을 베고,  
> 오염된 룬과 수호신을 정화하는 리듬 액션 게임.

</div>

---

## 📌 나의 역할

| 항목 | 내용 |
|------|------|
| **기간** | 2024.04 – 2026.04 (약 2년) |
| **역할** | 인게임 클라이언트 개발 · 채보 에디터 메인테이너 · Firebase 백엔드 연동 |
| **Tech Stack** | Unity C# · Firebase (Firestore / Cloud Functions / Auth) · Steamworks.NET · Node.js (TypeScript) |
| **커밋** | SoS-Refactoring 526커밋 · SOS_NoteEditor 61커밋 |

> 배경/사운드/프로필 씬 등 **초기 인게임 시스템**부터 시작해,
> Hold · Rolling · LongRange · EvilWood **4종 신규 노트 타입** 설계,
> **채보 에디터 전체 리팩토링(v0.4.0)** 메인테이너 인수,
> Firebase 기반 **서버 권위(Server-Authoritative) 경제 시스템** 구축까지 담당했습니다.

---

## 🎮 게임 소개

<table>
  <tr>
    <td width="50%">

**장르:** 리듬 액션 (싱글 / 로컬 Co-op / Remote Play)

**주요 콘텐츠:**
- 10곡 · 2/4트랙 구성
- 캐릭터 · 노트 스킨 · 배경 커스터마이징
- 유저 제작 악보 공유 시스템
- 글로벌 리더보드

    </td>
    <td width="50%">

**플랫폼:** Windows (Steam)  
**출시:** 2024.08.31 Early Access  
**개발사:** Ussistant Studio  
**스팀 평가:** 90% 긍정적

    </td>
  </tr>
</table>

---

## 📸 스크린샷

<!-- 스크린샷을 여기에 추가해주세요 -->
<table>
  <tr>
    <td align="center"><img src="" width="380"/><br/><sub>인게임 플레이</sub></td>
    <td align="center"><img src="" width="380"/><br/><sub>캐릭터 스킨 샵</sub></td>
  </tr>
  <tr>
    <td align="center"><img src="" width="380"/><br/><sub>배경 커스터마이징</sub></td>
    <td align="center"><img src="" width="380"/><br/><sub>노트 스킨 시스템</sub></td>
  </tr>
</table>

---

## 🛠 주요 구현 시스템

### 1. 커스터마이징 시스템 — Skin / BG / Note Skin

캐릭터 스킨 · 배경 · 노트 스킨을 상점에서 구매하고 인게임에 즉시 반영하는 시스템.

- `SkinData` / `BGData` / `NoteData` — ScriptableObject로 아이템 데이터 관리
- Steam Stats API(`SteamUserStats.GetStat`)로 소유권 실시간 검증
- 미소유 아이템 → 자물쇠 UI + 아이콘 회색 처리 자동 적용
- `Player3DViewPanel` — 상점에서 캐릭터 스킨 3D 미리보기

**BG 4레이어 패럴랙스 스크롤 (`BGManager`):**
```
Layer 1: Ground
Layer 2: ForeGround     ← 레이어별 독립 속도
Layer 3: MidGround      ← BgScrollTrigger 좌우 연동
Layer 4: BackGround
```
`Resources.LoadAll<BGData>("BGSO")`로 전체 BG 동적 로드,
인게임 캐릭터 이동에 따라 레이어가 서로 다른 속도로 스크롤.

---

### 2. Server-Authoritative 아키텍처 전환

**Before:** 클라이언트가 아이템 가격 계산 → 메모리 해킹으로 무료 구매 가능

**After:** 클라이언트는 "의도"만 전달, 모든 계산은 서버가 수행

```
Client: purchaseItem({ itemId, userId })
Cloud Function:
  ① GAME_ITEMS에서 가격 조회
  ② 잔액 검증
  ③ Firestore 트랜잭션으로 차감 + 지급 + 이력 기록
Client: 결과 수신
```

**Idempotency:** `transactionId` 기반 중복 요청 방지 → 아이템 이중 지급 0건

---

### 3. Steam IAP 파이프라인

```
① Steam Overlay 결제
② OnMicroTxnAuthorizationResponse → OrderID 수신
③ Cloud Function: Steam Web API로 주문 상태 서버 검증
④ "Approved" 확인 후 Gems 지급
⑤ Finalize 처리 + Firestore 이력 기록
```

**엣지 케이스 대응:** Finalize 후 네트워크 단절 → 로그인 시 자동 복구 (미지급 건 자동 처리)

---

### 4. 판정 시스템 개선

**치명적 버그 수정:** Miss 1번 발생 시 이후 전부 Miss되는 구조적 결함

| | Before | After |
|---|---|---|
| **구조** | 리스트 기반 — 리스트 최상단 노트를 판정 | Collider 기반 — 히트라인 내 Collider 노트를 판정 |
| **Miss 발생 시** | 리스트 상태 꼬임 → 연쇄 Miss | 해당 노트 Collider만 비활성화, 다음 노트 독립 동작 |

**정밀도 개선:** X좌표 기반 → DSP-time ms 차이 기반 판정
- 배속(1.0x ~ 6.0x)에 관계없이 동일한 판정 체감
- 30 / 60 / 144fps 환경 모두 동일하게 보장

---

### 5. Offline / Online 하이브리드 시스템

```
Online  → 결과 즉시 Firebase 저장 + 리더보드 업데이트
Offline → Local Cache에 임시 저장 + 서버 의존 기능(상점/배틀패스) 비활성화
재연결  → 캐시 일괄 업로드 후 초기화
```

---

### 6. 데이터 파이프라인 자동화

```
JSON 수정
  → MissionGenerator.cs (Editor Script) → ScriptableObject 자동 생성
  → ChartDataGenerator.cs → Firebase Firestore 자동 반영
```

디자이너가 JSON만 수정하면 개발자 개입 없이 서버까지 자동 반영. QA 사이클 약 **40% 단축**.

---

### 7. 서버 모듈화

1,245줄 God Object → 기능별 모듈 분리:

```
index.ts (진입점)
├── auth.ts       — 인증 / 유저 생성
├── economy.ts    — 재화 · 아이템 구매
├── battlepass.ts — 배틀패스 보상
├── missions.ts   — 미션 진행도
├── admin.ts      — 어드민 도구
└── helpers.ts / types.ts
```

---

### 8. Discord 운영 봇

```
/user <uid>         → 유저 정보 조회
/grant <uid> <item> → 아이템 직접 지급
/reset <uid>        → 계정 초기화
```

서버 에러 발생 시 `#server-alerts` 채널에 자동 알림 → Firebase 콘솔 없이 실시간 모니터링.

---

## 🎪 오프라인 전시 참여

| 전시명 | 기간 | 빌드 |
|--------|------|------|
| **BIC** (Busan Indie Connect) | 2024-08 | v0.1.32 |
| **CBF** (콘텐츠 비즈니스 포럼) | 2024-10-22~24 | v0.3.3 |
| **G-STAR 2024** | 2024-11-13~15 | v0.3.x |
| **G-STAR 2025** | 2025-11 | EvilWood 노트 포함 |

---

## 🎵 신규 노트 타입 구현 (4종)

| 노트 타입 | 도입 시점 | 설명 |
|-----------|-----------|------|
| **Hold Note** | 2024-09 (v0.2.x) | 길게 누르는 홀드 노트 |
| **Rolling Note** | 2024-10 (v0.3.0) | 연속 입력 롤링 노트 |
| **Long Range Note** | 2024-10 (v0.3.0) | 원거리 판정 노트 |
| **EvilWood Note** | 2025-11 | G-STAR 2025 전시용 신규 노트 |

---

## 📈 개발 흐름

| 시기 | 주요 작업 |
|------|-----------|
| 2024-04 | 팀 합류 · BGManager · 프로필 씬 · 사운드 시스템 |
| 2024-07~08 | BIC 전시 준비 · 판정/JSON 버그 수정 · 키 바인딩 |
| 2024-09~10 | Steam Early Access 출시 · Hold/Rolling/LongRange 노트 추가 |
| 2024-11~12 | G-STAR 2024 · 노트 판정 범위 버그 수정 |
| 2025-01~02 | 에디터 v0.4.0 리팩토링 · ms 기반 판정 전환 · BackToBack Monster |
| 2025-07~09 | Firebase 연동 · BattlePass 뼈대 · Mission 시스템 |
| 2025-11 | G-STAR 2025 · EvilWood 노트 추가 |
| 2025-12~2026-04 | Steam 소액결제 · BattlePass UX 개편 · 글로벌 리더보드 · Discord 봇 |

