# CRYENGINE 5.7.1 — Asset Browser 縮圖生成修復與升級

修復並升級 Sandbox 內建 Asset Browser 的縮圖生成（Mesh / SkinnedMesh / Skeleton / Material）。
基底版本：**CRYENGINE 5.7.1 原版源碼**（所有被改動的檔案在 5.7.1 之後未被官方變更過，patch 可直接套用）。

## 修了什麼

1. **截圖錯位修復**：原版第 N 顆資產的縮圖會存到第 N-1 顆的畫面（多執行緒渲染器一幀延遲 + `ScreenShot` 讀 presented backbuffer）。修法：截圖前連續渲染兩幀同內容再 flush。
2. **Environment probe 光照**：縮圖渲染管線由 `Minimum` 換成 `CharacterTool`，加入引擎預設 probe（`%ENGINE%/EngineAssets/Textures/default_probe_cm.dds`），縮圖有環境光照與反射。
3. **Mip 0 完全駐留閘**：貼圖串流到最高 mip 完成才截圖，杜絕低 mip 模糊縮圖；含 10 秒超時與 256-poll 停滯 fail-open 雙保底（貼圖源異常時照常出圖不卡死）。
4. **Studio 構圖**：逐軸八角點緊貼 fit（FOV 30°、方位角 60°、留邊 8%，Material 球另有專屬 margin），取代原版「外接球半徑 × 2」的粗略取景。
5. **灰色攝影棚背景**：`16_grey.dds` diffuse 的內面球 dome（半徑隨資產大小動態擴張），取代黑底/漸層背景。無地板。

所有新行為只在縮圖生成路徑生效（`EnableThumbnailPipeline` 旗標守門），互動預覽視窗（Material Browser、Create Object 面板等）行為完全不變。

## 改動檔案（共 6 個，全部在 Sandbox 編輯器內，引擎 runtime 零改動）

```
Code/Sandbox/EditorQt/QT/Widgets/QPreviewWidget.cpp   （主體）
Code/Sandbox/EditorQt/QT/Widgets/QPreviewWidget.h
Code/Sandbox/EditorQt/AssetSystem/MeshType.cpp        （opt-in 一行）
Code/Sandbox/EditorQt/AssetSystem/SkinnedMeshType.cpp （opt-in 一行）
Code/Sandbox/EditorQt/AssetSystem/SkeletonType.cpp    （opt-in 一行）
Code/Sandbox/EditorQt/AssetSystem/MaterialType.cpp    （opt-in 一行）
```

## 如何合併到你的項目

### 方式一：git patch（推薦，可留存修改紀錄）

在你的 CRYENGINE 源碼根目錄執行：

```bash
# 先確認可乾淨套用
git apply --check patches/cryengine-5.7.1-thumbnail-fix.patch
# 套用
git apply patches/cryengine-5.7.1-thumbnail-fix.patch
```

若你的 `QPreviewWidget.cpp` 等檔案已有自訂修改導致套用失敗，改用三方合併：

```bash
git apply --3way patches/cryengine-5.7.1-thumbnail-fix.patch
```

### 方式二：直接覆蓋檔案

本倉庫的 `Code/` 目錄結構與 CRYENGINE 源碼樹一致，把 `Code/` 整個覆蓋到你的源碼根目錄即可。
**前提**：你的這 6 個檔案是 5.7.1 原版（沒有自訂修改），否則會蓋掉你的改動——不確定就用方式一。

### 套用後

重新建置 **Sandbox** 目標（改動全在 `Sandbox.exe` 內，無其他模組需要重編）。
刪除既有 `.thmb` 縮圖快取後在 Asset Browser 右鍵 → Generate Thumbnails 即可看到新結果。

## 可調常數

視覺調校常數集中在 `QPreviewWidget.cpp` 頂部 `Private_PreviewWidget` namespace（FOV、方位角、留邊、probe 強度 `kThumbProbeIntensity`（預設 3.5）、ambient、dome 半徑等），改數值重編 Sandbox 即生效。
