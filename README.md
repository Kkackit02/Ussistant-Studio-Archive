<div align="center">

# Ussistant Studio Archive

[![Ussistant Studio](https://img.shields.io/badge/Ussistant_Studio-Website-blueviolet?style=flat-square)](https://www.ussistantstudio.com/)

**Ussistant Studio** 팀 참여 프로젝트 포트폴리오 아카이브입니다.

</div>

---

## 📌 참여 요약

| 항목 | 내용 |
|------|------|
| **참여 기간** | 2024.04 – 2026.04 (약 2년) |
| **역할** | 인게임 클라이언트 개발 · 채보 에디터 메인테이너 · Firebase 백엔드 연동 |
| **총 커밋** | SoS-Refactoring 526 + SOS_NoteEditor 61 = **587커밋** |
| **구현 기능** | 61개 |
| **해결 버그** | 12건 |
| **오프라인 전시** | BIC · CBF · G-STAR 2024 · G-STAR 2025 (4회) |

---

## 📂 프로젝트 목록

### 🎵 [Sound of Slash](./SoundOfSlash/)
> 리듬 액션 게임 · PC (Steam) · Early Access

<a href="./SoundOfSlash/">
  <img src="https://cdn.akamai.steamstatic.com/steam/apps/3036410/header.jpg" width="600"/>
</a>

[![Steam](https://img.shields.io/badge/Steam-Sound_of_Slash-1b2838?style=flat-square&logo=steam&logoColor=white)](https://store.steampowered.com/app/3036410/Sound_of_Slash/)
[![Rating](https://img.shields.io/badge/Steam_Rating-90%25_Positive-brightgreen?style=flat-square)](https://store.steampowered.com/app/3036410/)

| 항목 | 내용 |
|------|------|
| **기간** | 2024.04 – 2026.04 (약 2년) · 526커밋 |
| **역할** | 인게임 클라이언트 개발 · 채보 에디터 메인테이너 · Firebase 백엔드 연동 |
| **Tech** | Unity C# · Firebase · Steamworks.NET · Node.js (TypeScript) |

**핵심 구현:**
- Hold · Rolling · LongRange · EvilWood 신규 노트 타입 4종 설계·구현
- x좌표 기반 → ms 기반 판정 시스템 재설계 (해상도/배속 완전 독립)
- Firebase Server-Authoritative 경제 시스템 (BattlePass · Mission · 글로벌 리더보드)
- Steam IAP 소액결제 파이프라인 · Discord 운영 봇

---

### 🎛 [SOS NoteEditor](./SOS_NoteEditor/)
> Sound of Slash 전용 채보 에디터

| 항목 | 내용 |
|------|------|
| **기간** | 2025.01 – 2026.04 (약 16개월) · 61커밋 |
| **역할** | 에디터 메인테이너 인수 · 전체 리팩토링 (v0.4.0) · MIDI 채보 자동 생성 |
| **Tech** | Unity C# · JSON (LitJson / JsonUtility) · LINQ · Python |

**핵심 구현:**
- 그리드 엔진 전면 재설계 (9개 파일, 498줄 추가 / 904줄 삭제)
- Object Pooling 기반 무한 타임라인 스크롤 (GC 없음)
- BeatSnap Divisor · DragSystem · Undo/Redo 스냅샷 방식
- MIDI + DNN v3.0 채보 자동 생성 파이프라인

---

## 🎪 오프라인 전시

| 전시명 | 기간 | 빌드 |
|--------|------|------|
| **BIC** (Busan Indie Connect) | 2024-08 | v0.1.32 |
| **CBF** (콘텐츠 비즈니스 포럼) | 2024-10-22~24 | v0.3.3 |
| **G-STAR 2024** | 2024-11-13~15 | v0.3.x |
| **G-STAR 2025** | 2025-11 | EvilWood 노트 포함 |
