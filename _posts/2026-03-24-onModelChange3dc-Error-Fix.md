---
layout: post
title: "onModelChange3dc 에러 해결 스크립트"
date: 2026-03-24
---

Maya 실행 시 또는 씬 작업 중 간헐적으로 발생하는 `Cannot find procedure "onModelChange3dc"` 에러를 해결하는 Python 스크립트입니다.

**Maya Script Editor (Python 탭)** 에 붙여넣고 실행하면 에러 원인이 되는 ModelEditor 이벤트가 즉시 초기화됩니다.

---

## 기능

- 씬 내 모든 **ModelEditor** 자동 탐색
- 각 ModelEditor에 잘못 등록된 **editorChanged 이벤트 초기화**
- 3D-Coat 등 외부 플러그인 연동 후 남은 **잔여 콜백 제거**
- 재시작 없이 **즉시 적용**
- Maya 2018+ / Python 2·3 호환

---

## 에러 원인

`onModelChange3dc` 는 **3D-Coat** 플러그인이 Maya ModelEditor에 등록하는 콜백 함수입니다.
플러그인을 제거하거나 Maya를 재설치한 이후에도 ModelEditor에 해당 콜백이 잔존하여
씬을 열거나 뷰포트를 조작할 때 위 에러가 반복적으로 출력됩니다.

---

## 스크립트

```python
import pymel.core as pm

# 씬 내 모든 ModelEditor를 탐색하여 editorChanged 이벤트 초기화
for item in pm.lsUI(editors=True):
    if isinstance(item, pm.ui.ModelEditor):
        pm.modelEditor(item, edit=True, editorChanged="")
```

---

## 사용 방법

1. Maya **Script Editor** 열기 → **Python** 탭 선택
2. 위 스크립트 전체 복사 후 붙여넣기 → 실행
3. 에러 메시지가 더 이상 출력되지 않는지 확인
4. 씬을 저장하면 이후 재오픈 시에도 에러 미발생

---

## 참고

- 스크립트 실행 후에도 에러가 반복된다면 Maya **userSetup.mel** 또는 **userSetup.py** 파일 안에 `onModelChange3dc` 관련 호출이 남아있는지 확인하세요.
- 3D-Coat 플러그인을 계속 사용 중이라면 플러그인을 **정상 절차로 재설치**하는 것을 권장합니다.
