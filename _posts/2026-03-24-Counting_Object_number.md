---
layout: post
title: "선택 오브젝트 개수 카운터"
date: 2026-03-24
---

Maya 씬에서 현재 선택된 오브젝트의 개수를 즉시 출력해주는 Python 스크립트입니다.

**Maya Script Editor (Python 탭)** 에 붙여넣고 실행하면 Script Editor 하단에 선택 개수가 출력됩니다.

---

## 기능

- 현재 선택된 오브젝트 목록을 자동으로 감지
- 선택이 없을 경우 `0` 출력 (오류 없이 안전하게 처리)
- MEL의 반복문 없이 Python `len()` 함수로 간결하게 처리
- Maya 모든 버전 / Python 2·3 호환

---

## 스크립트

```python
import maya.cmds as cmds

# 선택된 오브젝트 목록 가져오기
selected = cmds.ls(selection=True)

# 리스트의 길이를 측정 (MEL의 for문 루프 대신 len() 함수 사용)
counter = len(selected) if selected else 0

print(counter)
```

---

## 사용 방법

1. Maya **Script Editor** 열기 → **Python** 탭 선택
2. 위 스크립트 전체 복사 후 붙여넣기
3. 뷰포트에서 오브젝트 선택
4. 스크립트 실행 → Script Editor 하단 출력창에 선택된 오브젝트 **개수 확인**
