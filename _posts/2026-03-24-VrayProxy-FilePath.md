---
layout: post
title: "VrayProxy-FilePath"
---

Vray Proxy (.vrmesh) 경로를 변경 해주는 python 스크립트 입니다.


```python
"""
VRayMesh Path Replacer
======================
씬 내 모든 VRayMesh 노드의 .vrmesh 파일 경로를
지정한 새 폴더로 일괄 변경합니다.
"""

import maya.cmds as cmds
import os

def get_all_vray_mesh_nodes():
    nodes = cmds.ls(type="VRayMesh") or []
    return nodes

def replace_vrmesh_paths(new_folder, dry_run=False):
    new_folder = new_folder.rstrip("/\\")
    results = []
    nodes = get_all_vray_mesh_nodes()
    
    if not nodes:
        cmds.warning("씬에 VRayMesh 노드가 없습니다.")
        return results

    for node in nodes:
        attr = "{0}.fileName".format(node)
        try:
            old_path = cmds.getAttr(attr) or ""
        except Exception:
            old_path = ""

        if not old_path:
            continue

        filename = os.path.basename(old_path)
        new_path = os.path.join(new_folder, filename).replace("\\", "/")
        results.append((node, old_path, new_path))

        if not dry_run:
            cmds.setAttr(attr, new_path, type="string")
    return results

WINDOW_ID = "vrmeshPathReplacerWin"

def _browse_folder(*args):
    folder = cmds.fileDialog2(fileMode=3, dialogStyle=2, caption="새 vrmesh 폴더 선택")
    if folder:
        cmds.textField("vrmeshNewFolderField", edit=True, text=folder[0])

def _run(dry_run=False):
    new_folder = cmds.textField("vrmeshNewFolderField", query=True, text=True).strip()
    if not new_folder:
        cmds.confirmDialog(title="경고", message="새 폴더 경로를 입력하세요.", button=["확인"])
        return
    results = replace_vrmesh_paths(new_folder, dry_run=dry_run)
    cmds.scrollField("vrmeshLogField", edit=True, text="")
    if not results:
        log = "처리할 VRayMesh 노드가 없습니다.\n"
    else:
        prefix = "[미리보기]" if dry_run else "[완료]"
        lines = ["{0} {1}개 노드 처리\n".format(prefix, len(results)), "-" * 60 + "\n"]
        for node, old, new in results:
            lines.append("노드 : {0}\n 이전: {1}\n 이후: {2}\n\n".format(node, old, new))
        log = "".join(lines)
    cmds.scrollField("vrmeshLogField", edit=True, text=log)

def _refresh_list(*args):
    nodes = get_all_vray_mesh_nodes()
    lines = []
    for node in nodes:
        attr = "{0}.fileName".format(node)
        try:
            path = cmds.getAttr(attr) or "(경로 없음)"
        except Exception:
            path = "(읽기 오류)"
        lines.append("{0} -> {1}".format(node, path))
    text = "\n".join(lines) if lines else "씬에 VRayMesh 노드가 없습니다."
    cmds.scrollField("vrmeshNodeListField", edit=True, text=text)

def show_ui():
    if cmds.window(WINDOW_ID, exists=True):
        cmds.deleteUI(WINDOW_ID)
    cmds.window(WINDOW_ID, title="VRayMesh Path Replacer", widthHeight=(560, 540),激izeable=True)
    cmds.columnLayout(adjustableColumn=True, rowSpacing=6, columnOffset=("both", 10))
    cmds.text(label="VRayMesh Path Replacer", font="boldLabelFont", height=22)
    cmds.separator(height=10)
    cmds.text(label="현재 씬의 VRayMesh 노드", align="left", font="boldLabelFont")
    cmds.scrollField("vrmeshNodeListField", editable=False, height=120, text="")
    cmds.button(label="목록 새로고침", height=28, command=_refresh_list)
    cmds.textField("vrmeshNewFolderField", placeholderText="새 폴더 경로 입력")
    cmds.button(label="찾아보기...", command=_browse_folder)
    cmds.button(label="경로 일괄 변경", height=34, backgroundColor=(0.25, 0.5, 0.3), command=lambda x: _run(False))
    cmds.scrollField("vrmeshLogField", editable=False, height=160)
    cmds.showWindow(WINDOW_ID)
    _refresh_list()

show_ui()
