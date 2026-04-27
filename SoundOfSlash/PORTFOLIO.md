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

---

### 2. 신규 노트 타입 4종 — 독립 설계

Steam Early Access 출시 후 지속적인 게임플레이 확장. 각 노트 타입은 **판정 로직 · 애니메이션 · JSON 포맷**을 독립 설계해 기존 시스템과 충돌 없이 추가.

| 노트 타입 | 도입 | 입력 방식 |
|-----------|------|-----------|
| **Hold Note** | 2024-09 | 키 유지 · 홀드 길이를 채보 `length` 필드로 관리 |
| **Rolling Note** | 2024-10 | 연속 입력 · 타이밍 누적 판정 |
| **Long Range Note** | 2024-10 | 원거리 판정 · 히트라인 도달 전 조기 입력 허용 |
| **EvilWood Note** | 2025-11 | G-STAR 2025 전시용 특수 패턴 |

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

---

### 4. BattlePass 시스템 — 설계부터 UX 개편까지

| 시점 | 작업 |
|------|------|
| 2025-07 | 시즌 구조 · 티어 설계 · 뼈대 구현 |
| 2025-09 | 티어별 보상 지급 · Firebase 연동 |
| 2025-12 | Steam IAP 연동 프리미엄 패스 구매 |
| 2026-02 | 진행도 UI · 보상 애니메이션 **전면 UX 개편** |

서버 권위 모델 적용 — 보상 지급은 Cloud Function에서만 처리, 클라이언트 조작 불가.

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

---

### 11. BeatHelper — 박자 가이드 시각화 (v0.3.22)

osu! 스타일 비트 링으로 노트 타이밍을 시각화. 플레이어 진입 장벽 완화.

---

### 12. BackToBack Monster 시스템 (v0.3.14)

특수 몬스터 연속 등장 메카닉. 에디터에서 BackToBack 노트 타입으로 채보에 직접 배치 가능.

---

### 13. 글로벌 리더보드 + 플레이어 아이콘 시스템 (2026-02)

- Firebase Firestore 기반 전 세계 점수 순위. 오프라인 결과는 재연결 후 자동 반영.
- 플레이어 아이콘 획득/장착 상태 서버 동기화.

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

| 버전 | 날짜 | 주요 내용 |
|------|------|-----------|
| v0.1.0 | 2024-06-04 | Steam 심사 준비 첫 빌드 |
| v0.1.32 | 2024-08-21 | BIC 전시 최종 빌드 |
| **v0.2.0** | **2024-09-02** | **Steam Early Access 정식 출시** |
| v0.3.0 | 2024-10-14 | Hold · Rolling · LongRange 노트 추가 |
| v0.3.3 | 2024-10-22 | CBF 전시 빌드 |
| v0.3.10 | 2024-12-27 | 노트 판정 범위 조정 · Fatal Error 수정 |
| v0.3.13 | 2025-02-15 | **ms 기반 판정 시스템 전환** |
| v0.3.14 | 2025-02-27 | BackToBack Monster 시스템 |
| v0.3.21 | 2025-03-21 | Accuracy 이펙트 · 해상도 수정 |
| v0.3.24 | 2025-07 | BeatHelper · Firebase 초기 연동 |

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
