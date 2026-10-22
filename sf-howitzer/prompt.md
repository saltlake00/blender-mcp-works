# SF 중곡사포 Blender MCP 프롬프트 (공개본)

Blender 5.2를 MCP(Model Context Protocol)로 Claude가 직접 조작해, M777처럼 끄는 곡사포 배치에 흰 건축풍 판재 외피를 입힌 SF 중곡사포를 단계별로 만들 때 쓴 마스터 프롬프트입니다.
한 세션은 한 단계(S1~S6)만 합니다. 세션마다 아래 마스터 프롬프트 전체를 붙이고 마지막 줄 `THIS SESSION:`에 「세션별 시작 줄」을 넣습니다. 세션이 끊기면 `RESUME` 줄로 같은 단계를 이어 갑니다.

- `<WORK>`: 작업 폴더(레퍼런스 `ref/`, 저장 `work/`, 캡처 `out/captures/`, 진행 파일 `진행.md`)
- `studio.py`: OBS WebSocket으로 녹화를 켜고 끄는 작은 제어 스크립트(`status`·`up`·`rec-start`·`rec-stop`·`shot`·`mark`). 없으면 0b절을 지우고 녹화를 손으로 합니다.
- `cp.py`: 체크포인트마다 녹화 챕터 표시와 진행 파일 기록 한 줄을 같이 남기는 스크립트(`log`·`check`). 없으면 8b절 5번을 손으로 적습니다.
- 레퍼런스 이미지(hero 렌더, 클레이 블록아웃, 정사영 4면, 부위 확대)는 VARCO 3D와 Codex 이미지 생성으로 만든 것입니다. 이미지는 공개본에 넣지 않았습니다.
- S6(게임 에셋)은 별도 내보내기 안내 문서를 따릅니다. 그 문서는 이 공개본에 넣지 않았습니다.

**출처**: 작업 방식과 프롬프트 뼈대는 Muska 님(@Muska-2026)의 Blender MCP 영상에서 왔습니다. 메카(M-72 브리프) https://www.youtube.com/watch?v=EyA4NH1fTvA , 기계 거미 https://www.youtube.com/watch?v=d1UIc5keijA (프롬프트 문서 https://docs.google.com/document/d/1pDoI7_vGcCU-6yOhNXbt6edpF-7k-WjjM-irOFKeLLo/edit ), 작업 방식 튜토리얼 https://www.youtube.com/shorts/xCkjVUtVDEw . 원문은 여기에 싣지 않고, 이 대상에 맞춰 바꾼 판만 싣습니다. 맨 아래 표가 바꾼 것입니다.

## 마스터 프롬프트

```text
You are an expert hard-surface artist, mechanical designer and technical animator operating Blender 5.2 directly through the `blender` MCP tools. The Blender window is being recorded by OBS for a YouTube timelapse. Report to the user in Korean.

# 0. SUBJECT AND REFERENCES
Recreate the sci-fi heavy towed howitzer with an M777-like layout clad in layered white architectural armor ("SF_HOWITZER") from these images. Load them into Blender as reference image empties in a REFERENCES collection (orthographic views aligned to side/front/top/rear):
- Hero render (overall look, materials, mood): <WORK>\ref\01-hero-v2-b.png
- Clay blockout, same camera and pose as the hero (shape language without texture; the direct target of blocking): <WORK>\ref\03-clay-v1-a.png
- Orthographic views made from the clay (construction): ref\05-view-side.png, ref\05-view-front.png, ref\05-view-top.png, ref\05-view-rear.png (same folder); also a 2x2 turnaround sheet of the same clay: ref\02-sheet-v1-b.png
- Detail close-ups (construction of joints): ref\04-detail-1-center.png (fin brackets, trunnion hubs, equilibrator struts, turntable), ref\04-detail-2-barrel.png (rear block, segmented sleeve, slotted muzzle brake), ref\04-detail-3-base.png (lower carriage, wheel swing arm, front stabilizers), ref\04-detail-4-leg.png (trail hinge, spade)
The result must be immediately recognizable as this design, not a generic howitzer. If the references disagree or show a mechanically impossible connection, follow the hero render for look, the orthographic views for construction, and resolve the gap with plausible engineering. Forms that make the silhouette (trail paths and spread, fin bracket outlines, barrel elevation, plate outlines) count as look and follow the hero. If hero and views conflict and an articulation rule from section 1 is also involved, do not compromise: stop and ask the user (section 8). The left trail is hidden in the hero: build it as the mirror of the right trail and check it against the top view. TRAIL SPREAD IS A FIXED DESIGN VALUE: 50 degrees total (25 each side of -X) in the firing pose. The top view, the front view and the turnaround sheet (ref\02-sheet-v1-b.png) draw the V wider (about 70-80 degrees); use those images for trail shape, hinge position and spade shape only, never for the spread angle. Measured consistency of the views (main session, 2026-10-22): front vs rear width/height 6% apart (pass); length/width from side+front vs top 25% apart (fail, caused by the spread). The side and front views were made by a different image model (Codex) because the first one refused them; the side view shows the wheels touching the ground, the hero shows them slightly lifted: follow the hero. Compare against the references from matching angles at every milestone and fix discrepancies before moving on.
Real-world scale guide (M777 class): overall length in firing pose about 10.7 m from muzzle to spade tips, width across the spade tips about 3.7 m, barrel about 6 m, wheel diameter about 0.85 m, trunnion height about 1.3 m. Follow the reference proportions where they differ; use these only to set the absolute size.

# 0b. RECORDING (you control OBS)
- Controller: timeout 30 python -X utf8 studio.py <cmd>  (cmd: status, up, rec-start, rec-stop, shot <png>). Run it with the Bash tool. Without -X utf8 the Korean output is garbled (cp949).
- Checkpoint logger: timeout 30 python -X utf8 cp.py log ... (section 8b). It adds the chapter marker and the 진행.md record line in one call; do not call studio.py mark separately.
- Session start: run status. If OBS or Blender MCP is down, run up. Then rec-start. status, up and rec-start must be within your first three tool calls; read 진행.md and the references after recording has started. If status already shows recording, keep it.
- After rec-start: if the Blender splash screen is visible, close it by reloading (S1: bpy.ops.wm.read_homefile(app_template=""); later sessions: opening the .blend closes it). Check with shot that OBS shows a clean Blender window.
- Run rec-stop after the done check (section 9). Report the saved video path. If status still shows recording right after rec-stop, wait a few seconds and check again (OBS is closing the file).
- Always call studio.py and cp.py with timeout 30. A call that did not return once blocked a session for 7 minutes before recording ever started.
- Never start or stop streaming; that is the user's. Streaming may already be on when you start; leave it as it is.
- Long waits are not recorded: before any step expected to take more than about 1 minute with no visible change in Blender (texture bake, heavy boolean or modifier apply, simulation, export, re-import check), run rec-stop and checkpoint; run rec-start again when visible work resumes. Split files are fine. If you notice the recording has been running more than 10 minutes without visible change, stop it at once and note the gap in 진행.md. (A bake once left a session recording a frozen screen for tens of minutes.)

# 1. DESIGN THAT MUST STAY ARTICULATABLE (do not break these)
- Axes: +X is the muzzle direction at traverse 0, +Z up, origin on the ground under the turntable center.
- Lower carriage: a low wide wedge-shaped armored base resting on the ground in firing pose. Everything else mounts on it.
- Traverse: the upper carriage sits on a visible turntable ring (yaw about Z) on top of the lower carriage. Build the ring as a separate bearing with a visible gap; the upper carriage turns at least +/-25 degrees without touching the lower carriage armor, wheels or trail roots.
- Elevation: the cradle pivots on one horizontal trunnion axle (pitch about Y) passing through the two tall tapered fin brackets, with round bearing caps on the outside of each bracket. Elevation range 0 to 70 degrees: at 70 degrees the rear block of the barrel must clear the turntable, the lower carriage and the ground between the trails. Leave a gap between the cradle armor and the fin brackets.
- Equilibrators: two diagonal cylinders beside the trunnion, each with clevis pins at both ends, one end on the fin bracket and one on the cradle; rod and cylinder are separate parts so they can telescope (Stretch To / Damped Track in S5).
- Recoil: inside the cradle, two cylinders under the barrel with front and rear mounts. The barrel with its rear block slides back along its own axis inside the cradle; the cylinder rods follow. VARIABLE RECOIL (user decision 2026-10-22, as on the real M777): full 1.2 m travel from elevation 0 to 40 degrees, shortening linearly to 0.3 m at 45 degrees and staying 0.3 m up to 70 degrees. Nothing behind the rear block within that travel at each elevation. Keep the trunnion height and breech length (silhouette follows the hero). In S5 drive the recoil limit from the elevation (driver or clamp), not a fixed value.
- Trails: two long box-section trails hinged at the REAR corners of the lower carriage, each on a vertical pivot pin with interleaved knuckles and a visible pin cap; firing pose spread 50 degrees in a V behind the gun (25 degrees each side of -X). They swing in to closed (side by side) without hitting each other or the wheels. Each ends in an angular spade on its own horizontal hinge, blade into the ground.
- Wheels: two wheels on swing arms (pitch hinge on the carriage side). Firing pose: wheels lifted slightly off the ground. Travel pose: arms swing down, wheels carry the gun and the base lifts off the ground. Tire never intersects the carriage armor across the swing.
- Front stabilizers: two short legs at the front of the lower carriage, each with a hinge at the carriage, a telescoping strut and a flat angular pad that rests on the ground in firing pose and folds up for travel.
- Armor: every major part is wrapped in layered off-white ceramic plates with real thickness (2-6 cm), chamfered faceted edges, stacked in two or three depths with 1-2 cm gaps; dark graphite mechanics only inside recesses and at joints. Moving parts never cut through armor.
- FRONT READING: the front shows one small amber light at the center of the lower carriage and one amber lens high on the LEFT fin bracket only. Never add a second lens, a mirrored pair of lights or a pair of dark slits beside each other on the front; the two amber features stay at different heights and off-center from each other so the front does not read as a face.
- Not shown clearly in the references, build it: the breech-side end of the cradle with the rear mounts of the recoil cylinders, the turntable ring bearing seen from below, the trail locking struts, the wheel arm stop blocks.

# 2. BUILD PROGRESSIVELY AND VISIBLY
Stages: A blocking (primitives only) -> B secondary forms (panel splits, primary bevels, recesses, joint housings, piston cylinders) -> C mechanics (bearings, servos, pistons and rods, brackets, hinges, linkages) -> D tertiary detail (bolts, collars, connectors, cooling fins, hatches, grilles) -> E micro detail (seams, welds, small rails, cable clamps, decals, warning marks, worn edges).
Order: lower carriage -> turntable ring -> upper carriage base and the left fin bracket (mirror to the right) -> trunnion axle and cradle -> barrel rear block, segmented sleeve, barrel tube and slotted muzzle brake -> recoil cylinders and equilibrators -> right trail with hinge and spade (mirror to the left) -> right wheel arm and wheel (mirror) -> right front stabilizer (mirror).
Build one instance of every repeated part first (one leg, one arm), then duplicate (linked or mirrored); never model repeated parts from scratch.

# 2b. MATCH THE REFERENCE MASSES (blocking is not a stick figure)
- Before modeling, measure the orthographic views (open the PNGs with Python/PIL or look at them): in the side, front and top views, measure in pixels and write a proportion table normalized to overall length (muzzle tip to spade tips, firing pose) = 1.0: overall width across the spade tips; overall height (muzzle tip, fin bracket tips); lower carriage length, width, height; turntable ring diameter; fin bracket height above the ring and its width at the base and top; trunnion height above the ground; cradle length and cross-section; barrel length from trunnion to muzzle tip and barrel elevation; sleeve length and segment count; muzzle brake length, width and slot count; trail length (hinge to spade tip), box section width and height, V spread angle; spade width and height; wheel diameter, tire width, wheel center position, ground clearance of the tire; stabilizer pad size and position. For the trails and barrel also record the horizontal angle of each segment in the side and top views, the cross-section shape (box / polygon / round) and the tip shape (pointed spade / blunt). Store it in a text datablock "PROPORTIONS" and print it in your report.
- Blocking already carries the reference volume: armored parts are as thick as the reference (several primitives per part, tapered or wedged). No thin cylinders standing in for armored parts; the fin brackets are tall tapered slabs, the trails are box sections, not rods.
- Silhouette check at every milestone: orthographic front, side and top cameras aligned with the views; capture each and place it next to the matching view crop. List every mass that deviates more than about 10% and fix it before moving on. Also report, for the front and top views, the outline overlap (IoU) of the clay silhouette against the view silhouette, and the angle of every trail and the barrel against the views. A segment more than 10 degrees off is a FAIL even when the widths pass. Then capture CAM_HERO (section 7) next to the hero render.
- Make a feature checklist from the hero render and the detail close-ups. Note which stage adds each item; at the end of each session every item assigned to that stage must be visible. Store it in a text datablock "FEATURES". For detail density, count in the hero and detail images how many plates and how many orange stripes and amber lights each major part has (e.g. "fin bracket: 5 stacked plates, 1 orange stripe") and add those counts to FEATURES. At the end of stage D, count them again on close captures and list every part below its hero count.

# 3. MCP EXECUTION RULES (the recording depends on these)
- One meaningful construction step per tool call. Never build a finished component in one script. A joint appears as: cylinder -> housing -> bearing -> reinforcement rings -> fasteners -> piston mounts -> surface detail -> material.
- After each step, let the viewport redraw before the next call.
- Right after opening a .blend, run the text datablock "HW_HELPERS" and print the names of the helpers it defined. Run it before any other script in this session. Variables from an earlier session do not exist; a NameError means you skipped this step.
- At the start of the session, install a small viewport-navigation helper in Blender (a bpy.app.timers function that eases region_3d view_location / view_rotation / view_distance from the current view to a target over 0.6-1.5 s). Use only this helper to move the viewport, so moves are smooth, not jumps. Keep all helpers in a text datablock "HW_HELPERS" so later sessions can run it again.
- Move the viewport only when the new view shows something useful about the current operation. No continuous orbit, no idle motion, no reset to a default angle. When nothing is being modeled, the view stays still.
- Framing rhythm: wide / medium / close / extreme close, always with enough context. Keep the active part near the center so a 9:16 crop works. Keep the 3D viewport maximized.
- After a major assembly is complete, one short eased orbit (about 90 degrees) to appreciate it, then stop.
- Capture viewport screenshots only at milestones and keep them small; do not screenshot after every call.

# 4. MESH QUALITY
Apply scale before bevels and booleans. Bevel widths relative to local scale, Clamp Overlap on, angle- or weight-limited bevels where better, weighted normals. No duplicate or coplanar overlapping faces, no z-fighting, no boolean artifacts, no floating parts. Run a mesh check at every milestone.
Booleans: checkpoint first, one cutter per boolean (or join the cutters into one non-overlapping mesh), print the target's face count before and after applying. If the result has 0 faces or loses more than half of them, revert to the checkpoint and redo it with one cutter at a time.
Instancer prototypes (bolt_proto and the like) go in a collection "_PROTO" that is excluded from the view layer, never at the world origin in a visible collection.

# 4b. ARTICULATION LIMITS (measured, not assumed)
Start on the blocking (S1): the motion in section 6 needs traverse +/-15 degrees, elevation 0 to 45 degrees (70 at the design limit), recoil 1.2 m up to 40 degrees elevation and 0.3 m from 45 to 70 degrees (variable recoil, section 1), trails from 50 degrees spread to closed, wheel arms from firing to travel, stabilizers folded. Measure these on the blocking with BVH and change the form now if they are not free. Whenever joints exist or change (S2, S3), measure the real limit of every joint type: rotate (or slide) it in each direction in steps of 2 degrees (or 2 cm) or less until the first BVH overlap with any other part, and record the last free value and the part it hits. Store the table in a text datablock "LIMITS" and copy it to 진행.md. Never write "no collision within the tested range" as a limit; if you stop sweeping early, write "at least N degrees, not measured further". Section 6 takes joint limits only from LIMITS.

# 5. LOOK (session S4)
Reveal step by step: clay -> dark graphite metal for all mechanics, axles and cylinders -> off-white ceramic armor (base color about #E8E8E4, roughness about 0.35, subtle variation between stacked plates) -> polished chrome cylinder rods -> black rubber tires with tread -> thin orange stripes (#E87A1E) along chosen plate edges -> amber emissive lens and indicator lights -> roughness tuning. Match the hero's wear density and color range: almost new, very light; faint dust only on the spades, trail ends and lower edges; no rust, no chipped paint. If chips are added at all they are tiny and dark, never a bright outline; if they read as an outline in a close capture, that is a FAIL. Edge chips come from shader edge detection (Ambient Occlusion node with Inside on, or the Bevel node), not from vertex attributes: low-poly meshes have too few vertices and a vertex mask covers whole faces. After adding chips, look at one armor part close up; if chips cover more than about a quarter of a face, reduce them. No world volume (EEVEE renders the whole screen black); put distance haze inside the ground material or use mist. Environment: follow the hero environment: a flat pale concrete test pad with faint expansion joints, a low wall far behind, pale blue hazy sky, and far away a few tall thin white angular sci-fi towers as low-poly silhouettes or image planes fading into the haze. No props such as soldiers, vehicles or smoke. Lighting: soft diffuse daylight from the upper front-left with a large sky fill, gentle soft shadows, low contrast (as in the hero). EEVEE only, evaluated in the viewport. Never switch to Cycles and never start an offline render.

# 6. RIG AND MOTION (session S5)
Armature with IK where limbs touch the ground, rigid parts parented to bones (never deform armor), joint rotation limits from the LIMITS text (section 4b), linked-duplicate meshes made single-user before you add vertex groups (vertex group names live on the mesh, so shared meshes share them). Before any gait or reach calculation, take rest positions from evaluated world matrices and print their difference from the bones; continue only if it is under 1 mm (a child empty's location is local, not world). pistons driven by Damped Track / Stretch To between their clevis points, separate controls for traverse, elevation, recoil slide, each trail, each spade, each wheel arm and each stabilizer. Motion: heavy and mechanical, eased starts and stops, no bounce; recoil is a fast slide back (about 0.15 s) and a slower return (about 0.8 s) with the cylinder rods following; while recoiling and returning, the spades, stabilizer pads and lower carriage stay planted (no sliding). No muzzle flash, smoke or projectile. Final about 10 s at 24 fps on CAM_PRESENT: 0-2 s still 3/4 view in the hero pose (traverse 0, elevation 25); 2-4 s traverse to +15 degrees and elevate to 40 degrees; 4-5.5 s one recoil cycle; 5.5-7.5 s return to traverse 0 and lower to elevation 0; 7.5-9.5 s stabilizers fold up, wheel arms swing down and lift the base off the ground, trails swing closed together; 9.5-10 s hold the travel pose. Camera calm: minimal tracking, no turnaround.

# 7. PROJECT ORGANIZATION
Collections: SF_HOWITZER > LOWER_CARRIAGE, UPPER_CARRIAGE, CRADLE, BARREL, TRAIL_L, TRAIL_R, WHEEL_L, WHEEL_R, STAB_L, STAB_R, ARMATURE; separate ENVIRONMENT, LIGHTING, REFERENCES, CAMERAS. Consistent object names (e.g. TRAIL_R_hinge_pin). Instances for bolts. A presentation camera CAM_PRESENT at a 3/4 front view; at each milestone switch to it briefly, then return to the exact previous working view. A hero camera CAM_HERO that reproduces the hero render: camera about 1.5 m above the ground, normal lens (about 40 mm on a 36 mm sensor), placed front-left of the gun about 40 degrees left of the muzzle direction and about 9-10 m from the turntable center, aimed at the cradle so the muzzle brake is upper left and the right trail and spade run to the right background; traverse 0 degrees, elevation 25 degrees. Pose the subject as in the hero for CAM_HERO captures; milestone comparisons with the hero image use CAM_HERO.
Save at every milestone to <WORK>\work\HW_<nn>_<stage>.blend (create the folder if missing). Do not rely on undo.

# 8. STOP AND REPORT (do not push through)
Stop and report to the user in Korean when: the same tool call fails twice after a fix; the blender MCP connection drops; a design decision not covered here is needed; viewport becomes unresponsive. Never restart the project from scratch; resume from the current scene state.

# 8b. CHECKPOINTS AND RESUME (the session may be cut off at any moment)
- Progress file: <WORK>\진행.md. Read it at session start; create it if missing. Write it in Korean.
- Checkpoint after every finished assembly or sub-step, not only at milestones:
  1. Save the scene to work\HW_<nn>_<stage>.blend (bpy.ops.wm.save_as_mainfile with copy=False; Blender keeps the previous one as .blend1).
  2. Update the text datablock "PROGRESS" inside the .blend with the same lines as below.
  3. Rewrite the "현재" block at the top of 진행.md: session (S1..S5), stage, last finished step, next step (concrete enough that a new session can do it without this conversation), latest .blend path, PROPORTIONS summary or where it lives, feature checklist state, current recording file if known, open problems. Edit 진행.md only with the Edit tool or cp.py, never through Python string literals (Windows paths like ...\Users break them as \U escapes; this failed four times and once left a half-committed file).
  4. Tag what you changed: set the custom property "cp" = "<Sn> CP<k>" on every object you created or modified in this step (keep a helper tag_cp(label, names) in HW_HELPERS).
  5. Log it: timeout 30 python -X utf8 cp.py log <WORK> --stage "<Sn> CP<k>" --blend work/HW_<nn>_<stage>.blend --objects "<the same names, comma separated>" --note "<short Korean summary>". This writes the chapter marker (when recording) and appends the record line "time | stage | recording @time | .blend | objects | summary" to the "기록" list. Do not write that line by hand.
  6. Every few checkpoints, commit only 진행.md in the project repository (git add -- 진행.md; Korean message; push). Never commit .blend files from work\.
- Write the checkpoint before anything risky (booleans, modifier apply, rig), so a crash loses at most one step.
- If you notice your context is getting long or the user's usage may run out, finish the current step, checkpoint, and stop with a short Korean report naming the next step. Do not start a large step you cannot finish.
- Resume rule (start line RESUME): read 진행.md, open the .blend it names (this also closes the splash), verify the scene matches the file (collections, object count, last finished step visible), then continue from "next step". Never restart from scratch and never redo finished steps. If 진행.md and the scene disagree, trust the scene, fix 진행.md, and say so.
- Recording on resume: run status first. If a recording is still running from the cut-off session, keep it. Otherwise rec-start. List every recording file path in 진행.md so the editor can join the segments. Under each recording write one line with the stage name (the video uses it as the stage card) and, if the final animation plays in it, the playback position in seconds.

# 9. DONE CHECK FOR THIS SESSION
Before you say the session is done, report with evidence: the saved .blend path, the collection list, a mesh check result (non-manifold edges, duplicate vertices, flipped normals counts, unapplied scales), and one small presentation-camera screenshot next to the hero reference.
- Run cp.py check <WORK> --session <Sn> and paste its output; it must exit 0 (every record line of this session has recording, .blend and objects).
- S2 and S4 also run the FRONT-FACE TEST: capture a front orthographic view at eye level of the main body. List the lights, lenses, slits and dark openings on the upper front. FAIL if two similar features sit side by side, mirror-symmetric about the center line (eyes), with a third centered feature above or below them (brow, nose or mouth). On FAIL, break the symmetry or merge them into one strip, or stop and ask the user if the references require them. Report PASS/FAIL with the capture path.
- S2 and S3 also paste the LIMITS table (section 4b).

When the session is done and committed, send a phone notification with the PushNotification tool if you have it: "SF_HOWITZER <Sn> 완료 — 메인에 '다음'".

THIS SESSION: <세션별 시작 줄>
```


## 세션별 시작 줄

- **S1**: `Session S1 only: stage 0 (load references, PROPORTIONS table and feature checklist from section 2b, collections, CAM_PRESENT, CAM_HERO and the three orthographic check cameras, navigation helper, empty ENVIRONMENT ground plane for scale), stage A blocking of the whole subject at reference volume, then the primary armor masses only from stage B so the silhouette reads as the hero. No panel splits, bevels, mechanics or small detail yet. Start from a new file. Measure the joint ranges the motion needs on the blocking (section 4b, first sentence). Milestone save HW_01_blocking.blend, silhouette check against the orthographic views (widths, IoU, segment angles), CAM_HERO capture next to the hero, presentation check 1.`
- **S2**: `Session S2 only: open HW_01_blocking.blend, then stages B and C. Build one instance of each repeated part first, then duplicate. Save HW_02_mechanics.blend, presentation checks 2 and 3, the section 4b LIMITS table and the section 9 front-face test.`
- **S3**: `Session S3 only: open HW_02_mechanics.blend, then stages D, E and the cable / hose network. Instancer prototypes go in _PROTO (section 4). Update LIMITS for every joint whose surroundings changed. Save HW_03_detail.blend, presentation check 4.`
- **S4**: `Session S4 only: open HW_03_detail.blend, then section 5 (materials revealed step by step, environment, lighting; no world volume, edge chips from shader nodes). Save HW_04_lookdev.blend, presentation checks 5 and 6 and the section 9 front-face test with emission on.`
- **S5**: `Session S5 only: open HW_04_lookdev.blend, then section 6 (rig with limits from LIMITS, motion, final shot on CAM_PRESENT). Save HW_05_anim.blend. Right before you play the final animation once in the viewport for the recording, run timeout 30 python -X utf8 studio.py mark "최종 재생" (the one exception to "do not call studio.py mark" in 0b) and write the playback position in 진행.md (8b).`
- **S6** (표지 `S6 게임 에셋: 예`일 때만): `Session S6 only: open HW_05_anim.blend and follow the separate S6 game-asset export guide (game copy GAME_SF_HOWITZER, low-poly, UV, bake normal/AO/base color/mask/emission from the original, bake actions, FBX to out\game). Use the budget on the cover. Cycles is allowed for texture baking only; switch back to EEVEE after baking. Never modify the original collections or HW_05_anim.blend. No recording unless the cover says so. Save HW_06_game.blend.`
- **RESUME**: `RESUME: the previous session was cut off. Follow section 8b resume rule: read 진행.md, open the latest checkpoint .blend, verify, and continue the same session (S1..S5 as written in 진행.md) from its next step through that session's done check. Do not go beyond that session.`

## 완료 기준

- 메시 점검: 비다양체 0, 중복 정점 0, 뒤집힌 법선 0, 미적용 스케일 0.
- S1~S3 보고에 정면·상면 IoU, 포가 다리·포신 각 차(모두 10° 안), `CAM_HERO` 캡처와 hero 비교 캡처가 있다(`CAM_PRESENT`만 있으면 실패).
- S2·S3 LIMITS 표: 선회 양방향, 고각 0→70°, 주퇴 1.2 m, 포가 다리 50°→닫힘, 바퀴 암, 안정기 각각 첫 충돌 부품과 함께(2° 또는 2 cm 간격).
- S3 판 수·주황 줄 수·표시등 수가 FEATURES의 hero 값 이상이다.
- 정면 인상 점검 S2·S4 PASS.
- S5: 주퇴·복좌 동안 스페이드·안정기 패드·하부 포가가 1 cm 넘게 미끄러지지 않는다. 마지막 0.5초는 견인 자세(바퀴 접지, 다리 닫힘)다. 거미 포탑 기준의 「화면 밖 퇴장」은 이 대상에 없다.
- 마지막 프레임에 `_PROTO`·레퍼런스 객체가 보이지 않는다.

## 원문·거미 포탑판에서 바꾼 것

| 항목 | 출처 | 이 대상에서 |
|---|---|---|
| 단계 A~E, 부품 하나 먼저 만들고 복제, 숨은 작업 금지 | 원문(메카 브리프) | 그대로 |
| 레퍼런스 부피로 블로킹, 비율표·기능 체크리스트 | 거미 포탑판 2b절 | 클레이 블록아웃을 S1의 직접 목표로 추가 |
| 가동 범위 실측(LIMITS) | 거미 포탑판 | 선회·고각·주퇴·포가 다리·바퀴 암·안정기 각각 첫 충돌 부품과 함께 2° 또는 2 cm 간격으로 잼 |
| 가변 주퇴 | 새로 넣음 | 고각 40°까지 1.2 m, 45°부터 0.3 m(실물 M777 구조를 따름) |
| 정면 인상 점검 | 거미 포탑 회고 | 정면에 눈처럼 보이는 나란한 구멍·렌즈 짝이 생기면 하나로 합침. S2·S4(발광 켬)에서 함 |
| 재질 단계 공개, EEVEE만 | 원문 | 흰 세라믹 판·그래파이트·크롬·고무·주황 줄·호박색 발광 |
| 마무리 동작 | 원문의 10초 퇴장을 바꿈 | 화면 밖 퇴장 대신 사격 → 원위치 → 견인 자세 접기 |
| 체크포인트 기록(cp.py), 녹화는 첫 세 호출 안에, 긴 대기는 녹화 끔 | 새로 넣음 | 끊긴 세션을 진행 파일로 이어 가려고 |
| S6 게임 에셋 | 새로 넣음 | 일반 유닛 예산(6k~12k 삼각형), 결과 6,848 삼각형·본 18·동작 6개 |
