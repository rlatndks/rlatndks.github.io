---
layout: post
title: "CgAbBlastPanelOptChangeCallback 에러 해결 스크립트"
date: 2026-03-24
---

Maya 실행 시 또는 씬 작업 중 간헐적으로 발생하는 `Cannot find procedure "CgAbBlastPanelOptChangeCallback"` 에러를 해결하는 Python 스크립트입니다.

**Maya Script Editor (Python 탭)** 에 붙여넣고 실행하면 모든 modelPanel에서 잘못된 콜백이 즉시 제거됩니다.

---

## 기능

- 씬 내 모든 **modelPanel** 자동 탐색
- 각 패널에 등록된 **editorChanged 콜백 조회**
- `CgAbBlastPanelOptChangeCallback` 콜백만 **선택적으로 제거** (다른 콜백에 영향 없음)
- 재시작 없이 **즉시 적용**
- Maya 2018+ / Python 2·3 호환

---

## 에러 원인

`CgAbBlastPanelOptChangeCallback` 는 **Maya 구버전 또는 일부 플러그인**이 modelPanel의 `editorChanged` 이벤트에 등록하는 콜백 함수입니다.
Maya 버전 업그레이드, 플러그인 제거, 씬 마이그레이션 이후에도 콜백 정보가 패널에 잔존하여
뷰포트 조작 시 위 에러가 반복적으로 출력됩니다.

---

## 스크립트

```python
from maya import cmds

# 모든 modelPanel을 순회하며 CgAbBlastPanelOptChangeCallback 콜백 제거
for model_panel in cmds.getPanel(typ="modelPanel"):
    
    # 현재 패널의 editorChanged 콜백 조회
    callback = cmds.modelEditor(model_panel, query=True, editorChanged=True)
    
    # CgAbBlastPanelOptChangeCallback 인 경우에만 제거
    if callback == "CgAbBlastPanelOptChangeCallback":
        cmds.modelEditor(model_panel, edit=True, editorChanged="")
```

---

## 사용 방법

1. Maya **Script Editor** 열기 → **Python** 탭 선택
2. 위 스크립트 전체 복사 후 붙여넣기 → 실행
3. 에러 메시지가 더 이상 출력되지 않는지 확인
4. 씬을 저장하면 이후 재오픈 시에도 에러 미발생

---

## 참고

- `onModelChange3dc` 에러와 발생 원인이 유사하며, 두 에러가 **동시에 발생**하는 경우도 있습니다. 해당 에러도 함께 발생한다면 `onModelChange3dc 에러 해결 스크립트`도 함께 실행하세요.
- 스크립트 실행 후에도 에러가 반복된다면 **userSetup.mel** 또는 **userSetup.py** 안에 관련 콜백 호출이 남아있는지 확인하세요.
