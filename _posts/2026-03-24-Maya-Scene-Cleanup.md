---
layout: post
title: "Maya 씬 클린업 - 불필요 노드 일괄 제거 스크립트"
date: 2026-03-24
---

Mental Ray 노드, Unknown 노드, Turtle 노드를 한 번에 정리해주는 Python 씬 클린업 스크립트입니다.

**Maya Script Editor (Python 탭)** 에 붙여넣고 실행하면 세 가지 불필요 노드가 순서대로 자동 제거됩니다.

---

## 기능

- **Mental Ray 노드** 자동 탐색 후 일괄 삭제
- **Unknown 노드** 탐색 → 잠금 해제 후 삭제 (씬 저장 오류 방지)
- **Turtle 노드** 잠금 해제 후 삭제
- 각 단계별 처리 결과를 Script Editor에 출력
- 노드가 없는 경우 건너뜀 (오류 없이 안전하게 처리)
- Maya 2018+ / Python 2·3 호환

---

## 스크립트

```python
import maya.cmds as cmds

# ──────────────────────────────────────────────
# 1. Mental Ray 노드 제거
# ──────────────────────────────────────────────

mental_types = [t for t in cmds.allNodeTypes() if "mentalray" in t.lower() or "mental" in t.lower()]
mental_nodes = []
for t in mental_types:
    mental_nodes += cmds.ls(type=t) or []

if mental_nodes:
    for node in mental_nodes:
        try:
            cmds.lockNode(node, lock=False)
            cmds.delete(node)
        except Exception as e:
            print("Mental Ray 삭제 실패: {0} - {1}".format(node, e))
    print("[완료] Mental Ray 노드 {0}개 삭제".format(len(mental_nodes)))
else:
    print("[건너뜀] Mental Ray 노드 없음")


# ──────────────────────────────────────────────
# 2. Unknown 노드 제거
# ──────────────────────────────────────────────

unknown_nodes = cmds.ls(type="unknown") or []

if unknown_nodes:
    for node in unknown_nodes:
        if not cmds.objExists(node):
            continue
        try:
            lock_state = cmds.lockNode(node, query=True, lock=True)
            if lock_state and lock_state[0]:
                cmds.lockNode(node, lock=False)
            cmds.delete(node)
        except Exception as e:
            print("Unknown 노드 삭제 실패: {0} - {1}".format(node, e))
    print("[완료] Unknown 노드 {0}개 삭제".format(len(unknown_nodes)))
else:
    print("[건너뜀] Unknown 노드 없음")


# ──────────────────────────────────────────────
# 3. Turtle 노드 제거
# ──────────────────────────────────────────────

turtle_nodes = [
    "TurtleDefaultBakeLayer",
    "TurtleBakeLayerManager",
    "TurtleRenderOptions",
    "TurtleUIOptions",
]

deleted_turtle = []
for node in turtle_nodes:
    if cmds.objExists(node):
        try:
            cmds.lockNode(node, lock=False)
            cmds.delete(node)
            deleted_turtle.append(node)
        except Exception as e:
            print("Turtle 노드 삭제 실패: {0} - {1}".format(node, e))

if deleted_turtle:
    print("[완료] Turtle 노드 {0}개 삭제: {1}".format(len(deleted_turtle), deleted_turtle))
else:
    print("[건너뜀] Turtle 노드 없음")


print("\n씬 클린업 완료. 파일을 저장하세요.")
```

---

## 사용 방법

1. Maya **Script Editor** 열기 → **Python** 탭 선택
2. 위 스크립트 전체 복사 후 붙여넣기 → 실행
3. Script Editor 출력창에서 각 단계 처리 결과 확인
4. 스크립트 실행 후 반드시 **씬 저장** (`Ctrl + S`)

---

## 참고

| 노드 종류 | 원인 | 영향 |
|---|---|---|
| Mental Ray 노드 | Mental Ray 플러그인 사용 이력 | 불필요한 렌더 설정 잔존 |
| Unknown 노드 | 플러그인 미설치 또는 버전 불일치 | 씬 저장·열기 시 경고 메시지 |
| Turtle 노드 | nCloth/Turtle 플러그인 이력 | 씬 로드 속도 저하 |
