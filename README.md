<div align="center">

<img src="https://cdn.akamai.steamstatic.com/steam/apps/3036410/header.jpg" width="800"/>

# Sound of Slash

[![Steam](https://img.shields.io/badge/Steam-Sound_of_Slash-1b2838?style=flat-square&logo=steam&logoColor=white)](https://store.steampowered.com/app/3036410/Sound_of_Slash/)
[![Early Access](https://img.shields.io/badge/Early_Access-2024.08.31-f4a42a?style=flat-square)](https://store.steampowered.com/app/3036410/)
[![Rating](https://img.shields.io/badge/Steam_Rating-90%25_Positive-brightgreen?style=flat-square&logo=steam)](https://store.steampowered.com/app/3036410/)
[![Ussistant Studio](https://img.shields.io/badge/Ussistant_Studio-Website-blueviolet?style=flat-square)](https://www.ussistantstudio.com/)

**리듬 액션 게임 · PC (Steam) · Early Access**

> 8개의 지형으로 나뉜 세계. 방랑자 G가 음악에 맞춰 적을 베고, 오염된 룬과 수호신을 정화하는 리듬 액션 게임.

</div>

---

## 📌 나의 역할

| 항목 | 내용 |
|------|------|
| **기간** | 2023.09 – 2026.03 (약 2.5년) |
| **역할** | Lead System Architect · Backend Developer · 인게임 로직 개발 |
| **Tech Stack** | Unity C# · Firebase (Firestore / Cloud Functions / Auth) · Steamworks.NET · Node.js (TypeScript) |

팀원이 **콘텐츠 확장**(새 노트 타입, 2P 모드, 튜토리얼)에 집중하는 동안,
나는 그것을 안전하고 안정적으로 담는 **인프라와 인게임 시스템**을 구축했습니다.

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
**스팀 평가:** 90% 긍정적 (얼리 액세스)

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

**구조:**
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
- `Resources.LoadAll<BGData>("BGSO")`로 전체 BG 동적 로드
- 인게임 캐릭터 이동에 따라 레이어가 서로 다른 속도로 스크롤

---

### 2. Server-Authoritative 아키텍처 전환

**Before:** 클라이언트가 아이템 가격 계산 → 메모리 해킹으로 무료 구매 가능

**After:** 클라이언트는 "의도"만 전달, 모든 계산은 서버(Firebase Cloud Functions)가 수행

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

**Before:** 리스트 기반 판정 → Miss 1번 발생 시 이후 전부 Miss되는 치명적 버그

**After:** Collider 기반 판정으로 전환
- 각 노트가 자체 Collider 보유
- Miss 발생 시 해당 노트 Collider만 비활성화, 다음 노트와 독립적으로 동작

**정밀도 개선:** X좌표 기반 판정 → DSP-time ms 차이 기반 판정
- 배속(1.0x ~ 6.0x)에 관계없이 동일한 판정 체감
- 프레임레이트(30 / 60 / 144fps) 독립적 판정 보장

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
  → MissionGenerator.cs (Editor Script)
  → ScriptableObject 자동 생성
  → ChartDataGenerator.cs
  → Firebase Firestore 자동 반영
```

디자이너가 JSON만 수정하면 개발자 개입 없이 서버까지 자동 반영. QA 사이클 약 **40% 단축**.

---

### 7. 서버 모듈화 (`index.ts` 분리)

1,245줄짜리 God Object를 기능별로 분리:

```
index.ts (진입점)
├── auth.ts       — 인증 / 유저 생성
├── economy.ts    — 재화 · 아이템 구매
├── battlepass.ts — 배틀패스 보상
├── missions.ts   — 미션 진행도
├── admin.ts      — 어드민 도구
├── helpers.ts    — 공통 유틸리티
└── types.ts      — 공유 타입
```

---

### 8. Discord 운영 봇

```
/user <uid>         → 유저 정보 조회
/grant <uid> <item> → 아이템 직접 지급
/reset <uid>        → 계정 초기화
```

서버 에러 발생 시 Discord `#server-alerts` 채널에 자동 알림 → Firebase 콘솔 없이 실시간 모니터링

---

## 🎛 SOS NoteEditor 기여

Sound of Slash 전용 비트맵 에디터. **2024.12 ~ 2026.01 (13개월, 60커밋)**

### 주요 구현

| 시스템 | 내용 |
|--------|------|
| **그리드 엔진 재설계** | 초기화·스크롤·풀링 구조 전면 재설계 (498줄 추가 / 904줄 삭제) |
| **DragSystem** | `DragManager.cs` + `DragArea.cs` 처음부터 작성 (205줄) — 드래그로 Holding/BackToBack 노트 길이 자동 계산 |
| **Object Pooling** | 화면에 보이는 셀(15개)만 유지, 스크롤 시 재활용 → GC 없는 무한 타임라인 |
| **BeatSnap Divisor** | Divisor 변경 시 모든 노트의 beatIndex를 비율로 재매핑 (osu! 방식) |
| **Undo/Redo** | 전체 상태 스냅샷 방식 — 새 노트 타입 추가 시 Undo 코드 수정 불필요 |
| **JSON 중복 제거** | `GroupBy(NoteName, time, beatIndex)`로 저장 시 중복 노트 자동 제거 |
| **Missile 노트** | 5개 파일 수정으로 신규 노트 타입을 에디터 전체에 통합 |

### 릴리즈 이력

| 버전 | 날짜 | 내용 |
|------|------|------|
| v0.3.7 | 2025.01.15 | DisplayNote · Hit Sound 안정화 |
| v0.4.0 | 2025.02.01 | 그리드 엔진 재설계 · DragSystem |
| v0.4.2 | 2025.02.15 | Long Range 안정화 · JSON 중복 제거 |
| v0.4.3 | 2025.02.27 | BeatSnap Divisor 전 노트 타입 적용 |
| v0.4.7 | 2025.10.18 | Missile 노트 · Unity 6000.2.8f1 |

---

## 📺 관련 영상 & 링크

[![Steam](https://img.shields.io/badge/Steam_페이지-1b2838?style=flat-square&logo=steam&logoColor=white)](https://store.steampowered.com/app/3036410/Sound_of_Slash/)
[![Ussistant Studio](https://img.shields.io/badge/공식_웹사이트-blueviolet?style=flat-square)](https://www.ussistantstudio.com/)

