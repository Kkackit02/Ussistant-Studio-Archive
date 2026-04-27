# SOS NoteEditor

Sound of Slash 전용 비트맵 에디터.

---

## 📋 개요

| 항목 | 내용 |
|------|------|
| **참여 기간** | 2025.01 – 2026.04 (약 16개월) |
| **커밋 수** | 61개 |
| **역할** | 에디터 메인테이너 인수 · 그리드 엔진 재설계 · 핵심 편집 시스템 구현 · 신규 노트 타입 추가 |
| **Tech Stack** | Unity C# · JSON (LitJson / JsonUtility) · LINQ |

> 2025-01-17 에디터 메인테이너를 인수하여 전체 코드베이스를 v0.4.0으로 대규모 리팩토링했습니다.
> 그리드 엔진의 구조적 불안정을 근본적으로 해결하고
> DragSystem · BeatSnap Divisor · Undo/Redo 등 핵심 편집 기능을 직접 설계·구현했습니다.
> 2026-04에는 MIDI 파일로부터 DNN v3.0 기반 채보를 자동 생성하는 기능을 추가했습니다.

---

## 📸 스크린샷

<table width="100%">
  <tr>
    <td align="center" width="50%">
      <img src="https://raw.githubusercontent.com/Kkackit02/Ussistant-Studio-Archive/main/SOS_NoteEditor/images/editor_screenshot_1.png" width="100%"/><br/>
      <sub>💻 에디터 메인 화면 — 그리드 및 타임라인</sub>
    </td>
    <td align="center" width="50%">
      <img src="https://raw.githubusercontent.com/Kkackit02/Ussistant-Studio-Archive/main/SOS_NoteEditor/images/editor_screenshot_2.png" width="100%"/><br/>
      <sub>⚙️ 에디터 설정 및 인스펙터</sub>
    </td>
  </tr>
  <tr>
    <td align="center" width="50%">
      <img src="https://raw.githubusercontent.com/Kkackit02/Ussistant-Studio-Archive/main/SOS_NoteEditor/images/editor_screenshot_3.png" width="100%"/><br/>
      <sub>🎵 채보 편집 및 프리뷰</sub>
    </td>
    <td align="center" width="50%">
      <img src="https://raw.githubusercontent.com/Kkackit02/Ussistant-Studio-Archive/main/SOS_NoteEditor/images/editor_screenshot_4.png" width="100%"/><br/>
      <sub>🎵 채보 편집 및 프리뷰(2)</sub>
    </td>
  </tr>

</table>

---

## 🌿 브랜치 전략

```
Note_Editor_JGN  ──── 2025.01.07 ~ 2025.01.15 (v0.3.7, 참여 초기 안정화)
                             │
                             └── Editor_Refectoring_2025 (2025.01.18 ~, 전면 리팩토링)
                                       ├── PR #6 → main  (v0.4.0,  2025.02.01)
                                       ├── PR #7 → main  (v0.4.2,  2025.02.15)
                                       ├── PR #8 → main  (v0.4.3,  2025.02.27)
                                       └── Direct → main (v0.4.5,  2025.04.14)
```

main 안정성을 위해 브랜치를 분리하고, 기능 완성 시점마다 점진적 PR 머지 전략을 택했습니다.

---

## 🛠 주요 구현

### 1. 그리드 엔진 재설계 (v0.4.0)

기존 그리드 시스템의 구조적 불안정을 근본적으로 해결하기 위해
초기화 로직·스크롤 연동·풀링 구조를 재설계했습니다.

- `GridManager.cs` +137줄 — 스크롤휠 타임라인, 재생 슬라이더, ResetGrid 재설계
- 총 변경량: 9개 파일, **498줄 추가 / 904줄 삭제**

---

### 2. DragSystem — 처음부터 작성

`DragManager.cs`(205줄) + `DragArea.cs`(45줄) 신규 작성.

드래그 범위 내 첫 번째 ~ 마지막 그리드의 시간 차이를 자동 계산해
Holding / BackToBack 노트의 `length`로 설정합니다.

```csharp
public void SetChineNote(List<Grid> selectedGrid) {
    float length = (float)(
        GridManager.instance.ConvertBeatToTime(selectedGrid[selectedGrid.Count - 1]._data.beatIndex)
        - GridManager.instance.ConvertBeatToTime(selectedGrid[0]._data.beatIndex)
    );
    selectedGrid[0].SetNoteData(true, GridManager.NoteType.Holding, length);
}
```

4개 트랙(Left / Right / LeftTop / RightTop)을 독립적으로 관리.

---

### 3. Object Pooling — 무한 타임라인 스크롤

화면에 보이는 셀(`initCount = 15`)만 유지,
셀이 화면 밖으로 나가면 반대쪽에서 재활용.

```csharp
private void FrontPooling() {
    _data.beatIndex += GridManager.instance.initCount;
    UpdateGridData(); // 새 beatIndex의 노트 데이터로 갱신
}
```

곡 길이와 무관하게 GC 없이 부드러운 무한 스크롤 구현.

---

### 4. BeatSnap Divisor (osu! 방식)

Divisor 변경 시 기존 노트의 `beatIndex`를 **비율로 재매핑**합니다.
(기존 코드는 `InitGrid()` 전체 초기화 → 노트 소실 문제)

```csharp
public void SetBeatSnapDivisor(int value) {
    int leastBeatSnapDivisor = beatSnapDivisor;
    beatSnapDivisor = value;
    UpdateNotesForNewDivisor(leftGridList,     leastBeatSnapDivisor, beatSnapDivisor);
    UpdateNotesForNewDivisor(rightGridList,    leastBeatSnapDivisor, beatSnapDivisor);
    UpdateNotesForNewDivisor(leftTopGridList,  leastBeatSnapDivisor, beatSnapDivisor);
    UpdateNotesForNewDivisor(rightTopGridList, leastBeatSnapDivisor, beatSnapDivisor);
    ResetGridUI();
}
```

`Destroy()` → `DestroyImmediate()` 교체: 같은 프레임 재생성 시 충돌 방지.

---

### 5. Undo/Redo — 전체 상태 스냅샷 방식

Command Pattern 대신 `PanelData` 전체를 JSON으로 직렬화해 스택에 적재.
새 노트 타입이 추가돼도 Undo/Redo 코드 수정 불필요.

```csharp
private Stack<string> undoStack = new Stack<string>();
private Stack<string> redoStack = new Stack<string>();

public void RecordSnapshot() {
    undoStack.Push(JsonUtility.ToJson(_EditPanelData));
    redoStack.Clear();
}
public void Undo() {
    redoStack.Push(JsonUtility.ToJson(_EditPanelData));
    _EditPanelData = JsonUtility.FromJson<PanelData>(undoStack.Pop());
    SyncUI();
    GridManager.instance.ResetGridUI();
}
```

`BatchAction` 패턴으로 복합 행동(Holding 드래그 등)을 하나의 Undo 단위로 묶음.

---

### 6. JSON 중복 제거

저장 시 같은 노트가 중복으로 기록되는 버그를 LINQ 한 줄로 해결.

```csharp
data.gridData = data.gridData
    .GroupBy(gd => new { gd.NoteName, gd.time, gd.beatIndex })
    .Select(g => g.First())
    .ToList();
```

---

### 7. Missile 노트 추가

5개 파일 수정으로 신규 노트 타입을 에디터 전체에 통합.

- `GridManager.cs` — `NoteType` enum에 `Missile` 추가
- `Grid.cs` — 스프라이트 표시 및 데이터 분기 처리
- `ShortCutManager.cs` — 단축키 등록

---

### 8. Offset / Calibration 시스템

에디터에서 저장하는 시간 값(ms 단위)과 인게임에서 읽는 시간 값(seconds 단위) 간 변환 로직.

```csharp
float calibrationOffsetSeconds = ((float)PlayerPrefs.GetInt("CALIBRATION_RESULT_MS", 0) / 1000.0f);
float hitOffset = calibrationOffsetSeconds + offset;
```

- 에디터 저장 값: ms 단위
- 인게임 로드 시: `× 0.001` 변환으로 seconds 단위로 처리
- 유저 캘리브레이션 결과(`CALIBRATION_RESULT_MS`)와 합산하여 최종 판정 오프셋 산출

이 변환 누락이 원인이 된 에디터-인게임 타이밍 불일치 버그(2024-07)를 해결한 핵심 로직.

### 8. Offset / Calibration — 타이밍 정합성 확보

에디터(ms 단위)와 인게임(seconds 단위) 간의 시간 단위 불일치로 인한 채보 밀림 현상을 해결한 핵심 변환 로직입니다.

```csharp
// 에디터 ms 값을 인게임 초 단위로 변환하며 유저 캘리브레이션 값 합산
float calibrationOffsetSeconds = ((float)PlayerPrefs.GetInt("CALIBRATION_RESULT_MS", 0) / 1000.0f);
float hitOffset = (msValue * 0.001f) + calibrationOffsetSeconds;
```

- **데이터 포맷 통일**: 에디터와 인게임이 동일한 JSON 스키마를 공유하도록 재설계하여 데이터 파싱 오류 원천 차단.
- **정밀도 보정**: `0.001f` 곱셈 연산을 통해 밀리초 단위의 정밀한 노트를 모든 플랫폼에서 동일한 타이밍에 재생.

---

### 9. MIDI + DNN v3.0 채보 자동 생성 파이프라인 (2026-04)

```
MIDI 파일 입력
  → MIDI Parser: 음표 데이터 추출 (시간, 음높이, 길이)
  → DNN v3.0: 리듬 패턴 예측 및 노트 배치 생성
  → SOS JSON 포맷으로 변환
  → 에디터에서 검토 및 수동 조정 가능
```

기획자가 MIDI 파일만 제공하면 기본 채보 초안을 자동 생성. 수작업 배치 시간을 대폭 단축.

---

## 📊 릴리즈 이력

| 버전 | 날짜 | 핵심 변경 |
|------|------|-----------|
| **v0.3.7** | 2025.01.15 | DisplayNote · Hit Sound 안정화 |
| **v0.4.0** | 2025.02.01 | 그리드 엔진 재설계 · DragSystem · Object Pooling |
| **v0.4.1** | 2025.02.01 | Chain 삭제 · 분할선 핫픽스 |
| **v0.4.2** | 2025.02.15 | Long Range 안정화 · JSON 중복 제거 |
| **v0.4.3** | 2025.02.27 | BeatSnap Divisor 전 노트 타입 적용 |
| **v0.4.5** | 2025.04.14 | 안정화 |
| **v0.4.7** | 2025.10.18 | Missile 노트 · Unity 6000.2.8f1 |
| **v0.4.8+** | 2026.04.24 | MIDI 변환 · DNN v3.0 채보 자동 생성 |

