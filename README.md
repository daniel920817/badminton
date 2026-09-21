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
