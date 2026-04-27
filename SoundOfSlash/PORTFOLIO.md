<div align="center">

# Sound of Slash — 포트폴리오 (SoS Portfolio)

[![Steam](https://img.shields.io/badge/Steam-Sound_of_Slash-1b2838?style=flat-square&logo=steam&logoColor=white)](https://store.steampowered.com/app/3036410/Sound_of_Slash/)
[![Studio](https://img.shields.io/badge/Ussistant_Studio-Website-blueviolet?style=flat-square)](https://www.ussistantstudio.com/)

← [게임 소개로 돌아가기](./README.md)

<br>

| 참여 기간 | 총 커밋 | 구현 기능 | 해결 버그 | 오프라인 전시 |
|:---------:|:-------:|:---------:|:---------:|:-------------:|
| **2024.04 – 2026.04** | **587커밋** | **61개** | **12건** | **3회** |

</div>

---

## 📌 내 역할 (Roles & Responsibilities)

| 항목 | 내용 |
|------|------|
| **역할** | 인게임 클라이언트 아키텍처 설계 · 채보 에디터 메인테이너 · Firebase 서버 권위 경제 시스템 구축 |
| **핵심 기여** | ms 기반 정밀 판정 엔진 전환 · 특수 노트 5종 설계 · 서버 기반 경제/결제 파이프라인 완성 |
| **Tech Stack** | Unity C# · Firebase (RTDB / Cloud Functions / Auth) · Steamworks.NET · Node.js (TypeScript) |

---

## 🔄 Before / After — 2년간의 기술적 성장

<table width="100%">
  <tr>
    <td align="center" width="50%">
      <img src="https://raw.githubusercontent.com/Kkackit02/Ussistant-Studio-Archive/main/SoundOfSlash/images/ingame_early.png" width="100%"/><br/>
      <sub><b>Before (2024-05)</b> — UI/UX 부재, 로컬 저장 기반의 프로토타입</sub>
    </td>
    <td align="center" width="50%">
      <img src="https://raw.githubusercontent.com/Kkackit02/Ussistant-Studio-Archive/main/SoundOfSlash/images/ingame_1.png" width="100%"/><br/>
      <sub><b>After (2026-04)</b> — 정밀 판정 엔진, 서버 연동, 다이나믹 스테이지 시스템</sub>
    </td>
  </tr>
</table>

| 분류 | **2024.04 합류 시점** | **2026.04 현재** |
|---|---|---|
| **판정 엔진** | x좌표 픽셀 기반 (해상도/배속 의존적) | **ms(밀리초) 기반** (프레임레이트/배속 독립적) |
| **노트 시스템** | 기본 2종 (Normal · Monster) | **특수 5종** (Hold · BackToBack · Missile 등) |
| **데이터 보안** | 클라이언트 로컬 연산 (메모리 조작 취약) | **Server-Authoritative** (서버 검증 기반 경제) |
| **인프라** | 로컬 저장 (데이터 휘발성) | **Firebase RTDB** (클라우드 동기화 및 오프라인 캐시) |
| **제작 도구** | 수작업 채보 배치 (낮은 생산성) | **AI 자동 생성** (MIDI + DNN v3.0 파이프라인) |

---

## 🛠 핵심 기술 구현 (Key Implementations)

### 1. ms 기반 정밀 판정 시스템 (Precision Hit Engine)

리듬 게임의 본질인 '일관된 타격감'을 위해 기존 좌표 기반 판정의 한계를 극복했습니다.

- **문제점**: 픽셀 거리 기반 판정 시, 노트 배속이 빨라지거나 해상도가 달라지면 실제 판정 범위가 물리적으로 변하는 문제 발생.
- **해결책**: 노트 도달 예정 시각과 실제 입력 시각의 차이를 **ms(밀리초)** 단위로 정밀 계산하도록 엔진 전면 재설계.
- **결과**: 배속(1x~6x) 및 해상도(FHD/QHD), 프레임레이트(30~144fps)와 무관하게 동일한 판정 체감 제공.
- **추가 개선**: 리스트 순차 탐색 방식에서 **Collider 이벤트 기반**으로 전환하여 대량의 연타 노트 발생 시에도 CPU 부하를 최소화하고 판정 꼬임 현상 해결.

<table width="100%">
  <tr>
    <td align="center" width="50%">
      <img src="https://raw.githubusercontent.com/Kkackit02/Ussistant-Studio-Archive/main/SoundOfSlash/images/ingame_1.png" width="100%"/><br/>
      <sub>인게임 정밀 판정 인터페이스</sub>
    </td>
    <td align="center" width="50%">
      <img src="https://raw.githubusercontent.com/Kkackit02/Ussistant-Studio-Archive/main/SoundOfSlash/images/scoreboard.png" width="100%"/><br/>
      <sub>결과 화면 — ms 오차 기반 판정 통계</sub>
    </td>
  </tr>
</table>

---

### 2. 다이나믹 스테이지 & 비주얼 시스템 (Dynamic Stage System)

스테이지별 고유한 분위기와 시각적 깊이를 위해 **ScriptableObject 기반의 Parallax 시스템**을 구축했습니다.

- **Parallax Scrolling**: 4개 이상의 레이어(Ground, Fore, Mid, Back)를 독립된 속도로 제어하여 공간감 구현.
- **Horror 스테이지 최적화**: Horror 스테이지 전용 셰이더 이펙트 및 안개(Fog) 연출을 시스템에 통합. 특히 Horror 스테이지에서는 몬스터 캐릭터(BAT, GHOST)의 가시성을 높이기 위한 다이나믹 라이팅 효과 적용.
- **배경 동기화**: 프로필에서 선택한 커스텀 배경 데이터가 인게임 로딩 시 `Resources.LoadAll`을 통해 즉시 동기화되는 파이프라인 구축.

<table width="100%">
  <tr>
    <td align="center" width="50%">
      <img src="https://raw.githubusercontent.com/Kkackit02/Ussistant-Studio-Archive/main/SoundOfSlash/images/ingame_2.png" width="100%"/><br/>
      <sub><b>Horror 스테이지</b> — 패럴랙스 배경 및 전용 몬스터 연출</sub>
    </td>
    <td align="center" width="50%">
      <img src="https://raw.githubusercontent.com/Kkackit02/Ussistant-Studio-Archive/main/SoundOfSlash/images/profile.png" width="100%"/><br/>
      <sub>스테이지 데이터 관리 UI</sub>
    </td>
  </tr>
</table>

---

### 3. 특수 노트 시스템 — 몬스터 메카닉 (Monster-Integrated Notes)

단순한 기하학적 형태가 아닌, 몬스터의 행동 양식과 결합된 5종의 특수 노트를 설계했습니다.

| 노트 타입 | 몬스터 (Horror 포함) | 입력 방식 & 메카닉 |
|-----------|--------------------|-------------------|
| **Hold** | 드래곤 / 대형 박쥐 | 키 유지 · 홀드 길이를 채보 데이터의 `length` 필드로 관리 |
| **BackToBack** | 쌍둥이 몬스터 | 연속 입력 · 타이밍 누적 판정 시스템 |
| **Long Range** | EvilWood | 원거리 발사체 · 포물선 궤적 연산 로직 |
| **Missile** | 부엉이 / 유령 | 플레이어를 향해 가속되는 발사체 판정 |
| **Combo** | 골렘 / 해골 | 좌우 동시 입력 · 멀티 터치/키 입력 처리 |

<table width="100%">
  <tr>
    <td align="center" width="33%">
      <img src="https://raw.githubusercontent.com/Kkackit02/Ussistant-Studio-Archive/main/SoundOfSlash/images/tutorial.png" width="100%"/><br/>
      <sub>노트 메카닉 튜토리얼</sub>
    </td>
    <td align="center" width="33%">
      <img src="https://raw.githubusercontent.com/Kkackit02/Ussistant-Studio-Archive/main/SoundOfSlash/images/char_horror.png" height="150"/><br/>
      <sub><b>Horror 전용 몬스터</b></sub>
    </td>
    <td align="center" width="33%">
      <img src="https://raw.githubusercontent.com/Kkackit02/Ussistant-Studio-Archive/main/SoundOfSlash/images/tutorial_3.png" width="100%"/><br/>
      <sub>몬스터 처치 및 콤보 연출</sub>
    </td>
  </tr>
</table>

---

### 4. Server-Authoritative 경제 시스템 (Economy Backend)

보안과 공정성을 위해 모든 재화 소모 및 보상 획득을 서버에서 검증하는 구조로 전환했습니다.

- **검증 로직**: 클라이언트의 가격 정보를 신뢰하지 않고, 서버 DB(Firestore)에 저장된 마스터 데이터를 기준으로 트랜잭션 처리.
- **Idempotency**: 네트워크 오류로 인한 중복 요청 방지를 위해 `transactionId` 기반의 멱등성 보장 로직 구현.
- **결제 파이프라인**: Steamworks SDK와 Firebase Cloud Functions를 연동하여 '구매 승인 -> 서버 검증 -> 아이템 지급 -> 영수증 처리'의 4단계 결제 프로세스 완성.

```typescript
// Cloud Functions: 결제 검증 및 아이템 지급 예시
export const purchaseItem = onCall(async (request) => {
  const { itemId, userId } = request.data;
  const itemData = await db.collection('GAME_ITEMS').doc(itemId).get();
  // 서버 측 검증 및 트랜잭션 처리...
});
```

<table width="100%">
  <tr>
    <td align="center" width="33%">
      <img src="https://raw.githubusercontent.com/Kkackit02/Ussistant-Studio-Archive/main/SoundOfSlash/images/shop_skin.png" width="100%"/><br/>
      <sub>서버 연동 상점 시스템</sub>
    </td>
    <td align="center" width="33%">
      <img src="https://raw.githubusercontent.com/Kkackit02/Ussistant-Studio-Archive/main/SoundOfSlash/images/battlepass.png" width="100%"/><br/>
      <sub>배틀패스 보상 서버 검증</sub>
    </td>
    <td align="center" width="33%">
      <img src="https://raw.githubusercontent.com/Kkackit02/Ussistant-Studio-Archive/main/SoundOfSlash/images/shop_bg.png" width="100%"/><br/>
      <sub>서버 기반 데이터 매핑</sub>
    </td>
  </tr>
</table>

---

### 5. 캐릭터 스킨 시스템 (Customization System)

플레이어의 수집 욕구를 자극하기 위한 다양한 스킨 시스템과 애니메이션을 관리했습니다.

- **Skin Matching**: Normal, Desert, Infernal 등 스테이지 테마와 어우러지는 스킨 3종 구현.
- **애니메이션 상태 머신**: 스킨별 고유 Idle, Attack, Death 애니메이션을 Animator Controller 리팩토링을 통해 효율적으로 관리.

<table width="100%">
  <tr>
    <td align="center" width="33%">
      <img src="https://raw.githubusercontent.com/Kkackit02/Ussistant-Studio-Archive/main/SoundOfSlash/images/player_normal.png" height="180"/><br/>
      <sub><b>Normal Skin</b></sub>
    </td>
    <td align="center" width="33%">
      <img src="https://raw.githubusercontent.com/Kkackit02/Ussistant-Studio-Archive/main/SoundOfSlash/images/player_desert.png" height="180"/><br/>
      <sub><b>Desert Skin</b></sub>
    </td>
    <td align="center" width="33%">
      <img src="https://raw.githubusercontent.com/Kkackit02/Ussistant-Studio-Archive/main/SoundOfSlash/images/player_infernal.png" height="180"/><br/>
      <sub><b>Infernal Skin</b></sub>
    </td>
  </tr>
</table>

---

## 🐛 주요 문제 해결 (Problem Solving)

| # | 핵심 이슈 | 발생 | 원인 및 해결책 |
|---|----------|------|---------------|
| 1 | **타이밍 정합성 오류** | 2024.07 | **[원인]** 에디터(ms)와 인게임(초) 간 단위 혼용. **[해결]** 데이터 스키마 통일 및 `× 0.001` 변환 레이어 추가로 정합성 100% 확보. |
| 2 | **해상도별 판정 불일치** | 2025.02 | **[원인]** 픽셀 좌표 기반 판정의 물리적 한계. **[해결]** 시간 오차(ms) 기반 판정 엔진으로 전면 교체하여 환경 독립성 확보. |
| 3 | **Fever 모드 중단 버그** | 2024.07 | **[원인]** ESC 일시정지 시 타임스케일 예외 처리 누락. **[해결]** 델타타임 보정 로직 및 상태 머신 동기화 조건 추가. |
| 4 | **DB 직렬화 실패** | 2026.03 | **[원인]** Firebase SDK의 익명 타입 직렬화 한계. **[해결]** 명시적 데이터 모델(Class-based) 도입 및 직렬화 유틸리티 구현. |

---

## 🎪 오프라인 전시 (Major Exhibits)

| 전시명 | 기간 | 비고 |
|--------|------|------|
| **BIC (Busan Indie Connect)** | 2024.08 | v0.1.32 Alpha 빌드 전시 및 유저 피드백 수집 |
| **G-STAR 2024** | 2024.11 | v0.3.x 안정화 빌드 전시, B2C 부스 운영 |
| **G-STAR 2025** | 2025.11 | **Horror 스테이지 및 EvilWood 신규 콘텐츠** 공개 전시 |

---

## 📈 빌드 히스토리 (Build History)

| 버전 | 주요 마일스톤 | 핵심 성과 |
|------|-------------|----------|
| **v0.2.0** | Steam Early Access 런칭 | 안정적인 클라이언트 베이스 및 Steamworks 연동 |
| **v0.3.13** | 판정 엔진 리밸런싱 | **ms 기반 판정 시스템 전환** — 유저 체감 품질 대폭 향상 |
| **v0.3.36** | 라이브 서비스 고도화 | **Server-Authoritative 시스템** 완성 및 배틀패스 UX 전면 개편 |
| **v0.4.0** | 에디터 엔진 리팩토링 | Grid 엔진 재설계 및 생산성 도구 고도화 (메인테이너) |
| **v0.4.8** | AI 파이프라인 도입 | **MIDI + DNN v3.0** 기반 채보 자동 생성 시스템 통합 |

<br>

👉 **[전체 빌드 이력 및 커밋 로그 상세 보기 (65개 마일스톤)](./PORTFOLIO.md#빌드-이력-상세)**
