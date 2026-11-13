# Badminton 3D reconstruction demo

EasyMocap 五機重建的單一 rally 檢視頁：左邊是 SMPL mesh 在世界座標中的位置，
右邊是五個原始視角，橘色骨架為重建 3D 關節的回投影。

- 片段：`250112_2 / 1_16_23 / left`，261 frames @ 50 fps
- PA-MPJPE vs GT：mean 21.0 mm / max 40.8 mm
- 視角：cam 0 2 4 7 9（左半場；cam 4 為魚眼，未去畸變但投影已套畸變參數）

## 檔案

| 檔案 | 內容 |
|---|---|
| `index.html` | 整頁（three.js 由 cdnjs 載入） |
| `verts.b64.txt` | 261×6890×3 SMPL 頂點，uint16 量化後 base64（14 MB） |
| `faces.json` | SMPL 13776 個三角面 |
| `kp.json` | 每幀 25 個 3D 關節 + confidence |
| `cams.json` | 五台相機的 K / dist / R / T / 世界座標位置 C |
| `meta.json` | 頂點量化的 lo / scale、幀數 |
| `view*.mp4` | 五個視角的原始片段（960×600, 50 fps） |

## 座標系

原點在球場正中央、**地面往上 0.5 m**，且 **−Y 為上**。

- 地板 `y = +0.5`，頭頂約 `y = −1.1`，相機高度約 `y = −5.1`
- `z` 沿長邊（底線 ±6.70，網子 z = 0），`x` 沿短邊（雙打邊線 ±3.05），左半場為 z 正向
- 轉到 Y-up 引擎：`Y' = 0.5 − y`（本頁另將 x 反號以維持右手系）
- 外參是 world→camera 的 `[R|T]`，相機在世界座標的位置為 `C = −Rᵀ T`

## 本機預覽

```
python3 -m http.server 8000
```

## 部署到 GitHub Pages

推上 GitHub 後，到 repo 的 Settings → Pages，Source 選 `Deploy from a branch`，
branch 選 `main` / `(root)`，等一分鐘就會有公開網址。

## 重新產生資料

資料由 `~/badminton/scripts/` 的輸出匯出：
- mesh 頂點來自 `easymocap_runs/<match>/<rally>/<side>/output/smpl/*.json` 過 SMPL forward
- 關節來自同一路徑下的 `output/keypoints3d/*.json`
- 相機參數來自 `~/badminton/data/camera_params/Cam_*_{intrinsic,extrinsic}.npy`

---

# easymocap_runs 重建檢查頁（雙方選手 + GT 對照）

跟 `web_demo_241217_1` 同樣的資料來源（`easymocap_runs` + 原始影片，全長），版面一樣，另外加了：

- **GT 對照**：學長的 SMPL 以白色半透明 mesh 疊在我們的重建上（橘 = 左半場、藍 = 右半場）
- **偵測框**：每個視角畫出 `filter_court.py` 選中的 2D 偵測；跟 GT 回投影 IoU < 0.3 的標紅（選到非球員，例如裁判）。魚眼視角不畫框
- **手腕誤差曲線**：每幀手腕誤差 vs GT（骨盆對齊），紅點 = 該幀有視角選錯人
- **跳到** 關鍵幀的按鈕，播放速度最慢 0.1×；拖時間軸時影片會一起跳

| 頁面 | 內容 |
|---|---|
| [`umpire_250114_1_1_13_20/`](umpire_250114_1_1_13_20/) | Rally 250114_1 · 1_13_20（抓到裁判）：cam 7 / cam 8 偶爾選到高椅上的主審 |
| [`hand_241217_1_1_00_01/`](hand_241217_1_1_00_01/) | Rally 241217_1 · 1_00_01（結尾手部錯誤）：左半場選手撲球時 frame 212–216 手臂位置錯 |
| [`wrongperson_250114_2_2_17_21/`](wrongperson_250114_2_2_17_21/) | Rally 250114_2 · 2_17_21（多視角選錯人）：cam 7 / cam 0 等同時選到底線外的人，重建跳過去、最多偏 5.4 m |

### 每個資料夾的檔案

| 檔案 | 內容 |
|---|---|
| `index.html` | 整頁（three.js 由 cdnjs 載入），標題、說明、統計都從 `meta.json` 讀 |
| `verts_{left,right}.bin` / `verts_{left,right}_gt.bin` | 重建 / GT 的 SMPL 頂點，N×6890×3 uint16 (little-endian)，`value*scale+lo` 還原 |
| `kp_{left,right}.json` | 每幀 25 個 body25 關節（由重建 SMPL 回歸） |
| `boxes.json` | 每個視角每幀選中的 bbox `[x1,y1,x2,y2,wrong]` |
| `faces.json` / `cams.json` | SMPL 三角面 / 十台相機（原始內外參，跟原始影片一致） |
| `meta.json` | 幀數、量化參數、GT mask、每幀手腕誤差與選錯人數、說明文字 |
| `view*.mp4` | 十個視角的原始影片（960×600, 50 fps） |

# 學長重建結果（人 + 球拍 + 羽球）

學長（俊良）VGGT 方法的重建結果，跟上面同一套版面與相機：

- **兩位選手**：SMPL（neutral），用學長改過的 smplx 產生，橘 = 左半場、藍 = 右半場
- **球拍**：SMPL 右手延伸出的第 25 個關節（`smpl_pose[72:75]`），變換跟學長 `smpl_params_to_obj_racket.py` 相同；網頁上用依 `racket.obj` 尺寸做的簡化模型（拍框、拍線面、拍桿、握把）
- **羽球**：學長的 `ball_3d`，3D 畫出前 0.6 s 軌跡，各視角投影成黃圈
- **擊球事件**：下方曲線是羽球高度，直線標出每次擊球；有「第 N 拍」「落地」跳轉按鈕
- 學長的資料只重建第一拍到落地，所以頁面與影片只放這一段；「原影片」欄是對應的原始影格編號

| 頁面 | 內容 |
|---|---|
| [`vggt_241217_1_1_00_01/`](vggt_241217_1_1_00_01/) | Rally 241217_1 · 1_00_01：原影片 frame 63–211，3 拍 |
| [`vggt_250114_1_1_13_20/`](vggt_250114_1_1_13_20/) | Rally 250114_1 · 1_13_20：原影片 frame 83–284，5 拍 |
| [`vggt_250114_2_2_17_21/`](vggt_250114_2_2_17_21/) | Rally 250114_2 · 2_17_21：原影片 frame 93–225，3 拍 |

每個資料夾的檔案跟上面相同，差別是沒有 GT mesh / 偵測框，多了：

| 檔案 | 內容 |
|---|---|
| `ball3d.json` | 每幀羽球 3D 位置（世界座標，同 `cams.json`） |
| `meta.json` → `players.*.racket` | 每幀球拍的 3×4 變換（obj 座標 → 世界座標），`racket_shape` 是簡化球拍的尺寸 |
| `meta.json` → `f0` / `hits` | 頁面第 0 幀對應的原影片 frame、擊球幀（頁面座標） |
