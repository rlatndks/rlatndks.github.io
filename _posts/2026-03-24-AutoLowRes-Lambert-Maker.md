---
layout: post
title: "Auto LowRes Lambert Maker - VRay 쉐이더 자동 로우레스 변환 스크립트"
date: 2026-03-24
---

선택한 VRay 쉐이더에 `low_` 접두사가 붙은 Lambert 쉐이더를 자동 생성하고, 연결된 텍스처를 256×256으로 리사이즈하여 한 번에 연결해주는 Python 스크립트입니다.

**Maya Script Editor (Python 탭)** 에 붙여넣고 실행하면 GUI 창이 열립니다.

---

## 기능

- 선택한 VRay 쉐이더 수만큼 **Lambert 쉐이더 자동 생성** (`low_쉐이더명` 네이밍)
- Lambert에 **VRay Specific Surface Shader** 속성 추가 후 VRay 쉐이더 자동 연결
- 이미 연결된 Lambert가 있을 경우 **재생성 없이 이름만 교체** (중복 방지)
- 원본 텍스처를 **256×256으로 리사이즈** 후 `low_파일명`으로 저장
- 리사이즈된 텍스처 노드를 Lambert `.color`에 **자동 연결**
- PNG / JPEG / TIFF / TGA / BMP / EXR 포맷 자동 감지
- 다중 선택 일괄 처리 및 처리 결과 다이얼로그 출력
- Maya 2018+ / Python 2·3 호환

---

## 스크립트

```python
import maya.cmds as cmds
import maya.OpenMaya as om
import os

def auto_create_and_connect_vray_specific():
    selection = cmds.ls(sl=True)
    if not selection:
        cmds.warning("연결할 브이레이 쉐이더를 먼저 선택해주세요.")
        return
        
    success_count = 0
    
    for source_node in selection:
        try:
            # 1. 이미 연결된 렘버트가 있는지 확인하는 조건부 탐색 로직
            existing_lambert = None
            out_connections = cmds.listConnections("{}.outColor".format(source_node), destination=True, source=False, plugs=True)
            
            if out_connections:
                for conn in out_connections:
                    if "vraySpecificSurfaceShader" in conn:
                        connected_node = conn.split('.')[0]
                        if cmds.nodeType(connected_node) == 'lambert':
                            existing_lambert = connected_node
                            break
            
            current_lambert = None
            
            # 2. 조건에 따른 분기 처리
            if existing_lambert:
                # 렘버트가 이미 있는 경우: 네이밍만 교체
                print("알림: 기존 연결 렘버트 발견. 네이밍 교체 및 텍스처 작업만 진행합니다.")
                desired_name = "low_{}".format(source_node)
                current_lambert = cmds.rename(existing_lambert, desired_name)
                print("이름 변경 완료: {} -> {}".format(existing_lambert, current_lambert))
                
            else:
                # 렘버트가 없는 경우: 기존처럼 새로 생성하고 SG 교체
                connected_sgs = cmds.listConnections("{}.outColor".format(source_node), type="shadingEngine", destination=True, source=False)
                if connected_sgs:
                    connected_sgs = list(set(connected_sgs))
                
                lambert_name = "low_{}".format(source_node)
                current_lambert = cmds.shadingNode('lambert', asShader=True, name=lambert_name)
                print("새 렘버트 생성 완료: {}".format(current_lambert))
                
                cmds.vray("addAttributesFromGroup", current_lambert, "vray_specific_mtl", 1)
                attr_enable = "{}.vrayEnableSpecificSurfaceShader".format(current_lambert)
                if cmds.objExists(attr_enable):
                    cmds.setAttr(attr_enable, 1)
                    
                target_attr = "{}.vraySpecificSurfaceShader".format(current_lambert)
                cmds.connectAttr("{}.outColor".format(source_node), target_attr, force=True)
                
                if connected_sgs:
                    for sg in connected_sgs:
                        cmds.connectAttr("{}.outColor".format(current_lambert), "{}.surfaceShader".format(sg), force=True)
            
            # 3. 공통 작업: 텍스처 256 리사이즈 및 current_lambert에 연결
            color_attr = "{}.color".format(source_node)
            if cmds.objExists(color_attr):
                connected_texture = cmds.listConnections(color_attr, source=True, destination=False, plugs=True)
                
                if connected_texture:
                    texture_node = connected_texture[0].split('.')[0]
                    
                    if cmds.nodeType(texture_node) == 'file':
                        orig_path = cmds.getAttr("{}.fileTextureName".format(texture_node))
                        
                        if orig_path and os.path.exists(orig_path):
                            dir_name = os.path.dirname(orig_path)
                            base_name = os.path.basename(orig_path)
                            new_file_name = "low_{}".format(base_name)
                            new_file_path = os.path.join(dir_name, new_file_name).replace("\\", "/")
                            
                            ext = os.path.splitext(base_name)[1].lower().replace(".", "")
                            image_format = "png"
                            if ext in ["jpg", "jpeg"]: image_format = "jpeg"
                            elif ext in ["tif", "tiff"]: image_format = "tiff"
                            elif ext in ["tga"]: image_format = "targa"
                            elif ext in ["bmp"]: image_format = "bmp"
                            elif ext in ["exr"]: image_format = "exr"
                            
                            print("{} 리사이징 진행 중...".format(source_node))
                            
                            image = om.MImage()
                            image.readFromFile(orig_path)
                            image.resize(256, 256, True)
                            image.writeToFile(new_file_path, image_format)
                            
                            new_file_node = cmds.duplicate(texture_node, inputConnections=True)[0]
                            cmds.setAttr("{}.fileTextureName".format(new_file_node), new_file_path, type="string")
                            
                            cmds.connectAttr("{}.outColor".format(new_file_node), "{}.color".format(current_lambert), force=True)
            
            success_count += 1
            print("{} 최종 처리 완료".format(source_node))
            
        except Exception as e:
            print("에러 발생 ({}): {}".format(source_node, e))

    cmds.confirmDialog(title="다중 작업 완료", message="총 {}개의 쉐이더 작업이 완료되었습니다.".format(success_count), button=["확인"])

def create_ui():
    window_id = "AutoVRayConnWinMulti"
    if cmds.window(window_id, exists=True):
        cmds.deleteUI(window_id)
        
    cmds.window(window_id, title="Auto LowRes Lambert Maker", widthHeight=(320, 80), sizeable=False)
    cmds.columnLayout(adjustableColumn=True, rowSpacing=10)
    cmds.text(label="브이레이 쉐이더 전체 선택 후 버튼 클릭", align="center")
    cmds.button(label="자동 변환 (low_ 적용 및 텍스처 굽기)", command=lambda x: auto_create_and_connect_vray_specific(), bgc=(0.4, 0.7, 0.5))
    
    cmds.showWindow(window_id)

create_ui()
```

---

## 사용 방법

1. Maya **Script Editor** 열기 → **Python** 탭 선택
2. 위 스크립트 전체 복사 후 붙여넣기 → 실행
3. **Hypershade** 또는 뷰포트에서 변환할 **VRay 쉐이더 선택** (다중 선택 가능)
4. GUI 창에서 **[자동 변환 (low_ 적용 및 텍스처 굽기)]** 버튼 클릭
5. Script Editor 출력창에서 처리 과정 확인
6. 완료 다이얼로그에서 처리된 쉐이더 개수 확인

---

## 참고

- 텍스처 리사이즈 결과물은 **원본 텍스처와 동일한 폴더**에 `low_원본파일명` 으로 저장됩니다.
- VRay 쉐이더의 `.color` 슬롯에 텍스처가 연결되어 있지 않으면 텍스처 작업은 건너뜁니다.
- 이미 `low_` Lambert가 연결된 쉐이더를 재실행하면 **삭제 없이 이름만 갱신**됩니다.
