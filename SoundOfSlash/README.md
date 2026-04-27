<div align="center">

<img src="https://cdn.akamai.steamstatic.com/steam/apps/3036410/header.jpg" width="800"/>

# Sound of Slash

[![Steam](https://img.shields.io/badge/Steam-Sound_of_Slash-1b2838?style=flat-square&logo=steam&logoColor=white)](https://store.steampowered.com/app/3036410/Sound_of_Slash/)
[![Early Access](https://img.shields.io/badge/Early_Access-2024.09.02-f4a42a?style=flat-square)](https://store.steampowered.com/app/3036410/)
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
> Hold · Rolling · LongRange · EvilWood **4종 신규 노트 타입** 설계·구현,
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
**출시:** 2024.09.02 Early Access
**개발사:** Ussistant Studio
**스팀 평가:** 90% 긍정적

  </td>
  </tr>
</table>

---

## 🛠 주요 구현 시스템

### 1. 노트 타입 시스템 확장 (4종 신규)

Steam Early Access 출시 이후 기존 일반/몬스터 2종 → 6종으로 확장. 각 노트 타입은 판정 로직 · 애니메이션 연동 · JSON 포맷을 독립적으로 설계.

| 노트 타입 | 도입 | 설명 |
|-----------|------|------|
| **Hold Note** | 2024-09 | 키 유지형 · 홀드 길이 = 채보 `length` 필드 |
| **Rolling Note** | 2024-10 | 연속 입력 · 타이밍 누적 판정 |
| **Long Range Note** | 2024-10 | 원거리 판정 · 히트라인 도달 전 조기 입력 허용 |
| **EvilWood Note** | 2025-11 | G-STAR 2025 전시용 특수 패턴 노트 |

---

### 2. 판정 시스템 재설계 — x좌표 → ms 기반 (v0.3.13)

**문제:** x좌표 픽셀 차이 기반 판정 → 배속·해상도 변경 시 판정 체감 불일치

| | Before | After |
|---|---|---|
| **판정 기준** | 화면상 x좌표 픽셀 거리 | 노트 도달 예정 시간 vs 입력 시간 차이 (ms) |
| **배속 변화 시** | 판정 범위가 배속에 비례해 변함 | 배속 1.0x ~ 6.0x 모두 동일한 ms 체감 |
| **해상도 변화 시** | 같은 타이밍이 다른 좌표로 계산됨 | 해상도 완전 독립 · 30/60/144fps 동일 보장 |

추가 구조 버그 수정: Miss 1회 → 이후 전부 Miss되는 리스트 상태 꼬임 → Collider 기반 구조로 전환. Miss 발생 시 해당 노트 Collider만 비활성화, 다음 노트 독립 동작.

---

### 3. BackToBack Monster 시스템 (v0.3.14)

특수 몬스터가 연속 등장하는 메카닉. 특정 구간에서 긴장감을 높이는 게임플레이 디자인. 에디터에서 BackToBack 노트 타입으로 채보 배치 가능.

---

### 4. BeatHelper — 박자 가이드 시각화 (v0.3.22, 2025-05)

osu! 스타일의 비트 링 형태로 박자를 시각화. 플레이어가 노트 타이밍을 직관적으로 인지할 수 있도록 지원.

---

### 5. 배경(BG) 시스템 — ScriptableObject + 패럴랙스 스크롤

7종 스테이지 배경(Forest · Desert · Ocean · Horror · Iceland · Toyland · Volcano)을 ScriptableObject 기반으로 관리.

**BG 4레이어 패럴랙스 스크롤 (`BGManager`):**
```
Layer 1: Ground
Layer 2: ForeGround     ← 레이어별 독립 속도
Layer 3: MidGround      ← BgScrollTrigger 좌우 연동
Layer 4: BackGround
```
`Resources.LoadAll<BGData>("BGSO")`로 전체 BG 동적 로드. 프로필 씬에서 선택한 배경이 인게임에 즉시 동기화.

---

### 6. 커스터마이징 시스템 — Skin / BG / Note Skin

캐릭터 스킨 · 배경 · 노트 스킨을 상점에서 구매하고 인게임에 즉시 반영.

- `SkinData` / `BGData` / `NoteData` — ScriptableObject로 아이템 데이터 관리
- Steam Stats API(`SteamUserStats.GetStat`)로 소유권 실시간 검증
- 미소유 아이템 → 자물쇠 UI + 아이콘 회색 처리 자동 적용
- `Player3DViewPanel` — 상점에서 캐릭터 스킨 3D 미리보기

---

### 7. Server-Authoritative 경제 시스템

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

### 8. BattlePass 시스템

| 단계 | 구현 내용 | 시점 |
|------|-----------|------|
| 뼈대 구현 | 시즌 구조 · 티어 설계 | 2025-07 |
| 보상 시스템 | 티어별 보상 지급 로직 | 2025-09 |
| 프리미엄 패스 | Steam IAP 연동 프리미엄 구매 | 2025-12 |
| UX 전면 개편 | 진행도 UI · 보상 애니메이션 재설계 | 2026-02 |

서버 권위 모델 적용: 클라이언트에서 진행도 조작 불가. 모든 보상 지급은 Cloud Function에서 검증 후 처리.

---

### 9. Mission 시스템

| 단계 | 구현 내용 | 시점 |
|------|-----------|------|
| 초기 구현 | 로컬 미션 트래킹 | 2024 하반기 |
| Firebase 연동 | 서버 동기화 추가 | 2025-09 |
| 완전 재설계 | Server-Authoritative 모델로 전환 | 2026-02 |

---

### 10. Steam IAP 파이프라인 (2025-12)

```
① Steam Overlay 결제 실행
② OnMicroTxnAuthorizationResponse → OrderID 수신
③ Cloud Function: Steam Web API로 주문 상태 서버 검증
④ "Approved" 확인 후 Gems 지급
⑤ Finalize 처리 + Firestore 이력 기록
```

**엣지 케이스 대응:** Finalize 후 네트워크 단절 → 재로그인 시 미지급 건 자동 복구 처리

---

### 11. Offline / Online 하이브리드 시스템 (2025-10)

```
Online  → 결과 즉시 Firebase 저장 + 리더보드 업데이트
Offline → Local Cache에 임시 저장 + 서버 의존 기능(상점/배틀패스) 비활성화
재연결  → 캐시 일괄 업로드 후 초기화
```

---

### 12. 서버 모듈화

1,245줄 God Object → 기능별 모듈 분리:

```
index.ts (진입점)
├── auth.ts       — Steam 인증 / 신규 유저 생성
├── economy.ts    — 재화 · 아이템 구매
├── battlepass.ts — 배틀패스 보상 지급
├── missions.ts   — 미션 진행도 검증
├── admin.ts      — 어드민 도구
└── helpers.ts / types.ts
```

---

### 13. Discord 운영 봇 (2026-02)

```
/user <uid>         → 유저 정보 조회
/grant <uid> <item> → 아이템 직접 지급
/reset <uid>        → 계정 초기화
```

서버 에러 발생 시 `#server-alerts` 채널에 자동 알림 → Firebase 콘솔 없이 실시간 모니터링.

---

### 14. 글로벌 리더보드 (2026-02)

Firebase Firestore 기반 전 세계 점수 순위 시스템. 온라인 플레이 결과가 자동 반영되며, 오프라인 플레이는 재연결 후 일괄 업로드.

---

### 15. 채보 에디터 MIDI + DNN v3.0 자동 생성 (2026-04)

MIDI 파일을 입력받아 DNN(Deep Neural Network) v3.0으로 채보 데이터를 자동 생성. 기획자가 수동으로 노트를 배치하는 시간을 대폭 단축.

---

## 🐛 주요 버그 해결

| # | 버그 | 발생 시점 | 핵심 원인 | 해결 |
|---|------|-----------|-----------|------|
| 1 | BGList NullReference | 2024-06 | 인스펙터 참조 끊김 | `Resources.Load()` 동적 로드 전환 |
| 2 | 에디터-인게임 타이밍 불일치 | 2024-07 | ms/seconds 단위 불일치 | `× 0.001` 변환 처리 |
| 3 | 모든 노트가 몬스터로 스폰 | 2024-07 | JSON 파싱 타입 분기 오류 | 타입 분기 로직 수정 |
| 4 | ESC 중 Fever 발동 | 2024-07 | 일시정지 상태 미처리 | 일시정지 상태 확인 조건 추가 |
| 5 | Combo 애니메이션 2개 동시 실행 | 2024-08 | 인스턴스 2개 동시 존재 | 단일 인스턴스 관리 |
| 6 | Grade FX/SFX 미재생 | 2024-08 | 사운드 초기화 순서 문제 | 초기화 완료 후 연출 실행 |
| 7 | Fade 무한루프 | 2024-08 | 종료 조건 누락 | 반복 방지 플래그 추가 |
| 8 | MISS 후 Fatal Error | 2024-12 | 판정 범위 경계값 오류 | 판정 범위 + 상태 처리 수정 |
| 9 | 해상도별 판정 불일치 | 2025-02 | x좌표 기반 판정 한계 | ms 기반 판정 시스템으로 전환 |
| 10 | Firebase 직렬화 오류 | 2026-03 | 익명 타입 직렬화 불가 | 명시적 클래스로 데이터 정의 |
| 11 | Fever 중 네트워크 팝업 노출 | 2026-03 | Fever 상태 미처리 | Fever 활성 여부 확인 조건 추가 |
| 12 | BattlePass 동기화 오류 | 2026-02 | 서버-클라이언트 매핑 불일치 | 데이터 매핑 로직 수정 |

---

## 🎪 오프라인 전시 참여

| 전시명 | 기간 | 빌드 |
|--------|------|------|
| **BIC** (Busan Indie Connect) | 2024-08 | v0.1.32 |
| **CBF** (콘텐츠 비즈니스 포럼) | 2024-10-22~24 | v0.3.3 |
| **G-STAR 2024** | 2024-11-13~15 | v0.3.x |
| **G-STAR 2025** | 2025-11 | EvilWood 노트 포함 |

---

## 📈 빌드 이력

| 버전 | 날짜 | 주요 내용 |
|------|------|-----------|
| v0.1.0 | 2024-06-04 | Steam 심사 준비 첫 빌드 |
| v0.1.32 | 2024-08-21 | BIC 전시 최종 빌드 |
| **v0.2.0** | **2024-09-02** | **Steam Early Access 정식 출시** |
| v0.3.0 | 2024-10-14 | Hold / Rolling / LongRange 노트 추가 |
| v0.3.3 | 2024-10-22 | CBF 전시 빌드 |
| v0.3.10 | 2024-12-27 | 노트 판정 범위 조정 · Fatal Error 수정 |
| v0.3.13 | 2025-02-15 | ms 기반 판정 시스템 전환 |
| v0.3.14 | 2025-02-27 | BackToBack Monster 시스템 |
| v0.3.21 | 2025-03-21 | Accuracy 이펙트 · 해상도 수정 |
| v0.3.24 | 2025-07 | BeatHelper · Firebase 초기 연동 |

---

## 📊 개발 흐름

| 시기 | 주요 작업 |
|------|-----------|
| 2024-04~05 | 팀 합류 · BGManager · 프로필 씬 · 플레이어 애니메이션 |
| 2024-06 | Steam 첫 심사 제출 (v0.1.0) |
| 2024-07~08 | BIC 전시 준비 · 판정/JSON 버그 수정 · 사운드/키 바인딩 시스템 |
| **2024-09-02** | **Steam Early Access 정식 출시 (v0.2.0)** |
| 2024-09~10 | Hold / Rolling / LongRange 노트 설계·구현 |
| **2024-11** | **G-STAR 2024 전시** |
| 2024-12 | 노트 판정 범위 버그 수정 |
| 2025-01~02 | 에디터 v0.4.0 리팩토링 · ms 기반 판정 전환 · BackToBack Monster |
| 2025-05 | BeatHelper 구현 |
| 2025-07~09 | Firebase 연동 · BattlePass 뼈대 · Mission 시스템 |
| **2025-11** | **G-STAR 2025 전시** · EvilWood 노트 추가 |
| 2025-12~2026-01 | Steam IAP · Gem 구매 시스템 · 오프라인 모드 |
| 2026-02~03 | BattlePass UX 개편 · Mission 재설계 · 글로벌 리더보드 · Discord 봇 |
| 2026-04 | MIDI + DNN v3.0 채보 자동 생성 |
