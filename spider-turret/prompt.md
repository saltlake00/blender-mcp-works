# 거미 포탑 Blender MCP 프롬프트 (공개본)

Blender 5.2를 MCP(Model Context Protocol)로 Claude가 직접 조작해 사족 보행 포탑을 단계별로 만들 때 쓴 마스터 프롬프트입니다.
한 세션은 한 묶음의 단계(S1~S5)만 합니다. 세션마다 아래 마스터 프롬프트 전체를 붙이고 마지막 줄 `THIS SESSION:`에 「세션별 시작 줄」을 넣습니다.

- `<WORK>`: 작업 폴더(레퍼런스 `ref/`, 저장 `work/`, 진행 파일 `진행.md`)
- `studio.py`: OBS WebSocket으로 녹화를 켜고 끄는 작은 제어 스크립트(`status`·`up`·`rec-start`·`rec-stop`·`shot`). 없으면 0b절을 지우고 녹화를 손으로 합니다.
- 레퍼런스 이미지 2장(hero 렌더, 설계 시트)은 VARCO 3D로 만든 것입니다.
- 두 원문 프롬프트를 거미 포탑 하나에 맞춰 합치고 가동성 규칙·MCP 실행 규칙·세션 분할·멈춤 조건을 더했습니다. 맨 아래 표가 원문에서 가져온 것과 바꾼 것입니다.

## 마스터 프롬프트

```text
You are an expert hard-surface artist, mechanical designer and technical animator operating Blender 5.2 directly through the `blender` MCP tools. The Blender window is being recorded by OBS for a YouTube timelapse. Report to the user in Korean.

# 0. SUBJECT AND REFERENCES
Recreate the quadruped spider-walker artillery turret ("SPIDER_TURRET") from these two images. Load both into Blender as reference image empties in a REFERENCES collection (sheet views aligned to front/side/top):
- Hero render (overall look, materials, mood): <WORK>\ref\01-hero-v2-b.png
- Design sheet (front / side / rear / top orthographic + 4 detail callouts): <WORK>\ref\02-sheet-v2-a.png
The result must be immediately recognizable as this design, not a generic spider robot. If the references disagree or show a mechanically impossible connection, follow the hero render for look, the sheet for construction, and resolve the gap with plausible engineering. Compare against the references from matching angles at every milestone and fix discrepancies before moving on.

# 0b. RECORDING (you control OBS)
- Controller: python studio.py <cmd>  (cmd: status, up, rec-start, rec-stop, shot <png>). Run it with the Bash tool.
- Session start: run status. If OBS or Blender MCP is down, run up. If the Blender splash screen is visible, close it by reloading (S1: bpy.ops.wm.read_homefile(app_template=""); later sessions: opening the .blend closes it). Check with shot that OBS shows a clean Blender window.
- Run rec-start at session start, right after the status check and before you change anything in Blender (the user wants every session recorded from its first frame). If status already shows recording, keep it. Run rec-stop after the done check (section 9). Report the saved video path.
- Always call studio.py with a timeout (timeout 30 python ...). A call that did not return once blocked a session for 7 minutes before recording ever started.
- Never start or stop streaming; that is the user's. Streaming may already be on when you start; leave it as it is.

# 1. DESIGN THAT MUST STAY ARTICULATABLE (do not break these)
- Low, wide armored hull. Four legs leave the hull at its four corners in an X layout; feet planted wider than the hull. Front and rear stepping arcs must never overlap.
- Each hip = a vertical-axis turntable ring bearing (yaw, swing forward/back) carrying a horizontal hinge (pitch). The references do not show the yaw ring clearly: build it.
- Each leg = thick upper segment rising to a knee, single-axis knee hinge with a visible axle pin, long tapered lower segment, ankle pivot, clawed tip with a round contact pad.
- A large hydraulic piston spans each knee, both ends on clevis pins at real mechanical endpoints; a smaller piston at each hip. Hoses follow the legs with slack loops at joints and never pass through armor.
- All knee peaks stay below the turret ring. The turret rotates 360 degrees on a raised ring with clearance above every leg.
- One long cannon with a perforated muzzle brake (about 1.3 x hull length) balanced by the rear counterweight / ammo module. Small secondary gun and sensor mast. Cyan emissive sensor lights on the hull front.
- Armor plates have real thickness and consistent panel gaps, and leave clearance at every joint so it can bend without interpenetration.

# 2. BUILD PROGRESSIVELY AND VISIBLY
Stages: A blocking (primitives only: hull, turret, cannon, four legs as simple segments, feet) -> B secondary forms (panel splits, primary bevels, recesses, joint housings, piston cylinders) -> C mechanics (bearings, servos, pistons and rods, brackets, hinges, linkages) -> D tertiary detail (bolts, collars, connectors, cooling fins, hatches, grilles) -> E micro detail (seams, welds, small rails, cable clamps, decals, warning marks, worn edges).
Order: hull frame -> hull armor -> turret ring -> turret and cannon -> counterweight module -> sensor mast -> hip yaw rings -> first leg (FL) fully through the current stage -> mirror/duplicate to FR, BL, BR and adjust -> pistons -> feet -> cables and hoses -> details.
Build one leg first, then duplicate (linked or mirrored); never model all four legs from scratch.

# 2b. MATCH THE REFERENCE MASSES (blocking is not a stick figure)
A previous blocking attempt was rejected: thin strut legs splayed sideways from the hull middle, a small low boxy hull, a thin short cannon. It did not read as this machine even at blocking level. So:
- Before modeling, measure the design sheet (open the PNG with Python/PIL or look at it): in the front, side and top views, measure in pixels and write a proportion table normalized to hull length = 1.0: hull length / width / height, turret ring diameter and height, turret length, cannon length and barrel diameter, rear module size, upper leg length and its armor shell width and depth, knee height above ground, lower leg length and thickness, foot pad diameter, stance length and width (foot to foot). Store it in a text datablock "PROPORTIONS" and print it in your report.
- Blocking already carries the reference volume: leg segments are heavy armored shells as thick as the reference (several primitives per segment, tapered or wedged), the hull is wide and layered, the cannon is long and thick. No thin cylinders standing in for armored parts.
- Leg path as in the hero: from the hull corner up and outward to a high knee (still below the turret ring), then steeply down to the foot. Not sideways out of the hull middle.
- Silhouette check at every milestone: orthographic front, side and top cameras aligned with the sheet views; capture each and place it next to the matching sheet crop. List every mass that deviates more than about 10% and fix it before moving on.
- Make a feature checklist from the hero render (e.g. layered hull armor with cyan sensor slits at the front, segmented leg armor shells with exposed pistons behind them, rear ammo / counterweight module, turret box with mantlet, secondary gun, sensor mast, perforated muzzle brake). Note which stage adds each item; at the end of each session every item assigned to that stage must be visible.

# 3. MCP EXECUTION RULES (the recording depends on these)
- One meaningful construction step per tool call. Never build a finished component in one script. A joint appears as: cylinder -> housing -> bearing -> reinforcement rings -> fasteners -> piston mounts -> surface detail -> material.
- After each step, let the viewport redraw before the next call.
- At the start of the session, install a small viewport-navigation helper in Blender (a bpy.app.timers function that eases region_3d view_location / view_rotation / view_distance from the current view to a target over 0.6-1.5 s). Use only this helper to move the viewport, so moves are smooth, not jumps.
- Move the viewport only when the new view shows something useful about the current operation (a new surface, a joint axis, piston alignment, cable path). No continuous orbit, no idle motion, no reset to a default angle. When nothing is being modeled, the view stays still.
- Framing rhythm: wide (proportions) / medium (one assembly) / close (joints, pistons, cables) / extreme close (small detail), always with enough context. Keep the active part near the center so a 9:16 crop works. Keep the 3D viewport maximized.
- After a major assembly is complete, one short eased orbit (about 90 degrees) to appreciate it, then stop.
- Capture viewport screenshots only at milestones and keep them small; do not screenshot after every call.

# 4. MESH QUALITY
Apply scale before bevels and booleans. Bevel widths relative to local scale, Clamp Overlap on, angle- or weight-limited bevels where better, weighted normals. No duplicate or coplanar overlapping faces, no z-fighting, no boolean artifacts, no floating parts. Run a mesh check at every milestone.

# 5. LOOK (session S4)
Reveal step by step: clay -> dark gunmetal mechanics -> oxide-red / maroon armor -> black inserts -> brushed metal pistons -> optics -> cyan emissive -> roughness tuning. Weathering follows use: chipped paint on edges, dust low on legs, oil near joints; no random grunge. Environment: flat sandy desert ground with fine gravel and subtle dust at the feet, pale overcast sky, light haze, nothing else (no soldiers, smoke, props). Lighting: soft overcast sun plus sky, readable cavities. EEVEE only, evaluated in the viewport. Never switch to Cycles and never start an offline render.

# 6. RIG AND WALK (session S5)
Armature with IK per leg, rigid parts parented to bones (never deform armor), joint rotation limits, pistons driven by Damped Track / Stretch To between their clevis points, turret yaw and cannon pitch as separate controls. Walk like an engineered platform, not a biological spider: diagonal gait (FL+BR, then FR+BL), body shifts weight before each lift, slow heavy steps, feet planted with no sliding, small body stabilization and suspension settle, hoses follow. Final about 10 s at 24 fps: 0-2 s stands in a clean 3/4 view on the presentation camera; 2-8.5 s walks; 8.5-9.5 s leaves the frame completely (no fade); 9.5-10 s hold the empty desert. Camera calm: minimal tracking, no turnaround.

# 7. PROJECT ORGANIZATION
Collections: SPIDER_TURRET > HULL, TURRET, CANNON, LEG_FL, LEG_FR, LEG_BL, LEG_BR, HYDRAULICS, CABLES, DETAILS, ARMATURE; separate ENVIRONMENT, LIGHTING, REFERENCES, CAMERAS. Consistent object names (e.g. LEG_FL_knee_piston). Instances for bolts. A presentation camera CAM_PRESENT at a 3/4 front view; at each milestone switch to it briefly, then return to the exact previous working view.
Save at every milestone to <WORK>\work\ST_<nn>_<stage>.blend (create the folder if missing). Do not rely on undo.

# 8. STOP AND REPORT (do not push through)
Stop and report to the user in Korean when: the same tool call fails twice after a fix; the blender MCP connection drops; a design decision not covered here is needed; viewport becomes unresponsive. Never restart the project from scratch; resume from the current scene state.

# 8b. CHECKPOINTS AND RESUME (the session may be cut off at any moment)
- Progress file: <WORK>\진행.md. Read it at session start; create it if missing. Write it in Korean.
- Checkpoint after every finished assembly or sub-step (hull, turret, cannon, each leg, each check), not only at milestones:
  1. Save the scene to work\ST_<nn>_<stage>.blend (bpy.ops.wm.save_as_mainfile with copy=False; Blender keeps the previous one as .blend1).
  2. Update the text datablock "PROGRESS" inside the .blend with the same lines as below.
  3. Rewrite the "현재" block at the top of 진행.md: session (S1..S5), stage, last finished step, next step (concrete enough that a new session can do it without this conversation), latest .blend path, PROPORTIONS summary or where it lives, feature checklist state, current recording file if known, open problems. Append one line to the "기록" list at the bottom (time, step, file).
  4. Every few checkpoints, commit only 진행.md in the work repository (git add -- 진행.md; Korean message; push). Never commit .blend files from work\.
- Write the checkpoint before anything risky (booleans, modifier apply, rig), so a crash loses at most one step.
- If you notice your context is getting long or the user's usage may run out, finish the current step, checkpoint, and stop with a short Korean report naming the next step. Do not start a large step you cannot finish.
- Resume rule (start line RESUME): read 진행.md, open the .blend it names (this also closes the splash), verify the scene matches the file (collections, object count, last finished step visible), then continue from "next step". Never restart from scratch and never redo finished steps. If 진행.md and the scene disagree, trust the scene, fix 진행.md, and say so.
- Recording on resume: run status first. If a recording is still running from the cut-off session, keep it. Otherwise rec-start. List every recording file path in 진행.md so the editor can join the segments.

# 9. DONE CHECK FOR THIS SESSION
Before you say the session is done, report with evidence: the saved .blend path, the collection list, a mesh check result (non-manifold edges, duplicate vertices, flipped normals counts, unapplied scales), and one small presentation-camera screenshot next to the hero reference.

THIS SESSION: <세션별 시작 줄>
```

## 세션별 시작 줄

- **S1**: `Session S1 only: stage 0 (load references, PROPORTIONS table and feature checklist from section 2b, collections, CAM_PRESENT and the three orthographic check cameras, navigation helper, empty ENVIRONMENT ground plane for scale), stage A blocking of the whole machine at reference volume, then the primary armor shells only from stage B (hull armor layers, turret box, leg armor shells, rear module) so the silhouette reads as the hero. No panel splits, bevels, mechanics or small detail yet. Start from a new file. Milestone save ST_01_blocking.blend, silhouette check against the sheet, presentation check 1.`
- **RESUME**: `RESUME: the previous session was cut off. Follow section 8b resume rule: read 진행.md, open the latest checkpoint .blend, verify, and continue the same session (S1..S5 as written in 진행.md) from its next step through that session's done check. Do not go beyond that session.`
- **S2**: `Session S2 only: open ST_01_blocking.blend, then stages B and C. Leg FL first, then duplicate. Save ST_02_mechanics.blend, presentation checks 2 and 3.`
- **S3**: `Session S3 only: open ST_02_mechanics.blend, then stages D, E and the cable / hose network. Save ST_03_detail.blend, presentation check 4.`
- **S4**: `Session S4 only: open ST_03_detail.blend, then section 5 (materials revealed step by step, desert ground, lighting). Save ST_04_lookdev.blend, presentation checks 5 and 6.`
- **S5**: `Session S5 only: open ST_04_lookdev.blend, then section 6 (rig, walk, 10 s walk-off on CAM_PRESENT). Save ST_05_anim.blend and play the final animation once in the viewport for the recording.`

## 원문에서 가져온 것과 바꾼 것

| 항목 | 출처 | 메모 |
|---|---|---|
| 단계 A~E, 부품별 반복, 숨은 작업 금지 | 01 §3·4·7 | 그대로 |
| 카메라는 이유가 있을 때만 | 01 §5 | 02의 "계속 움직여라"는 버렸다. 마일스톤 뒤 감상 회전만 02에서 가져옴 |
| 다리 구성(어깨 관절·액추에이터·피스톤·발 접지) | 02 ROBOT MODELING | 가동성 점검 결과를 더해 §1로 강화 |
| 재질 단계 공개 | 02 MATERIAL REVEAL | 색은 hero 렌더(산화 적색)로 바꿈 |
| EEVEE만, 오프라인 렌더 금지 | 01 §12 | 02의 Cycles 권장은 버렸다(녹화가 목적) |
| 10초 퇴장 구성 | 02 FINAL ANIMATION | 사막 환경에 맞춤 |
| 사막 환경 | 01 §11 | 소품 없이 바닥·하늘·안개만 |
| 시점 이동 도우미, 호출당 한 단계, 세션 분할, 멈춤 조건, 증거 보고 | 새로 넣음 | 작업 흐름 문서 6·7절 |
