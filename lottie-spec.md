# Extension tip 手指動畫：Lottie 規格 v1.1（Soft 曲線）

> **成品**：`lottie-output/extension_tip_finger.json`（5.7 KB）、`extension_tip_finger.lottie`（1.6 KB）。LottieFiles：Gogolook workspace，檔名 `extension_tip_finger_v1`（fileId 8925564e-e4e2-422a-9c4d-b566ac9973b8）。
> **結構**：主 comp `extension_tip_finger`（48×42）→ precomp `hand_motion`（加 48×42 遮罩）→ group `hand`（finger／wrist／sleeve 三個 path，只有 Position 有 keyframe）。

> 2026-11-20 定版 v1；2026-11-21 改為 v1.1：往上曲線改 Soft，避免「撞到頂」的急煞感。對應 prototype 設定：動畫範圍 48×42／Soft 曲線／點擊陰影關閉／往上 6pt／兩組間隔 1 秒／1× 速度。
> Prototype：https://vernagong-lab.github.io/ios-extension-tip-animation/#mobile
> Figma：04_Protection → Home Extension card hint → Animation Area `16509:18327`（node 16509-18327）

---

## 1. 畫布與輸出

| 項目 | 定義 |
|---|---|
| 檔名 | `extension_tip_finger.json`（可另附 `.lottie`） |
| 畫布 | 48 × 42（1 unit = 1pt），背景透明 |
| FPS | 60 |
| 長度 | 258 格（4.3 秒），`ip` 0 → `op` 258 |
| 第 0 格＝最後一格 | 都是「休息姿勢」，跟 Figma 靜態畫面相同 |
| 內容 | 純向量 shape layer，不用圖片、文字、effect、expression |
| 裁切 | 手指超出畫布的部分要裁掉（手指圖形高 62，畫布只有 42）。JSON 內加一個 48×42 的 layer mask；RD 端的 view 也設 `clipsToBounds = true` |

## 2. 素材

- `lottie-assets/hand.svg`：手指向量，**往上姿勢**的座標（指尖頂端在 y = 0.75），未裁切。
- `lottie-assets/extension_tip_rest_48x42.svg`：第 0 格／最後一格的參考圖（休息姿勢，已裁切）。

手指由三個形狀組成，整組一起移動，不做個別變形：

| 形狀 | 顏色 | 來源 |
|---|---|---|
| finger（手掌與手指） | `#FFD8CB` | 插圖色，Mobile DS 沒有對應 token |
| wrist（手腕，旋轉 -15°） | `#FFD8CB` | 同上 |
| sleeve（袖口，旋轉 -15°） | `#0CD25F` | `Colors/Core/Accent`（Green/500） |

深色模式顏色不變，只交一份 JSON。

## 3. 動作

只有一個動畫屬性：**hand 圖層的 Y 位移**。
- 休息姿勢：Y = +6（指尖頂端在 y 6.75）
- 往上姿勢：Y = 0（指尖頂端在 y 0.75，貼到畫布上緣）

### 3.1 每一下（30 格 = 0.5 秒）

| 段落 | 格數 | Y 值 | 曲線 |
|---|---|---|---|
| 往上 | 11 格 | 6 → 0 | Soft：`cubic-bezier(0.4, 0, 0.2, 1)`，起步柔和、慢慢減速貼到頂 |
| 停在上緣 | 4 格 | 0 | 不動 |
| 往下 | 10 格 | 0 → 6 | Soft：`cubic-bezier(0.4, 0, 0.2, 1)` |
| 停在休息位置 | 5 格 | 6 | 不動 |

Lottie keyframe 的寫法：往上段、往下段起點 keyframe 都是 `o: {x: 0.4, y: 0}`、`i: {x: 0.2, y: 1}`。

> v1（已停用）：往上 8 格 ease-out expo `(0.16, 1, 0.3, 1)`、停 7 格、往下 10 格 ease-out quart `(0.25, 1, 0.5, 1)`。第一格就衝 3.5pt，到頂有急煞感。

### 3.2 時間軸（60fps）

| 格數 | 秒 | 內容 |
|---|---|---|
| 0–18 | 0.00–0.30 | 休息（開場延遲，包含在 JSON 內） |
| 18–48 | 0.30–0.80 | 第 1 組第 1 下 |
| 48–78 | 0.80–1.30 | 第 1 組第 2 下 |
| 78–108 | 1.30–1.80 | 第 1 組第 3 下 |
| 108–168 | 1.80–2.80 | 兩組之間停 1 秒（休息姿勢） |
| 168–198 | 2.80–3.30 | 第 2 組第 1 下 |
| 198–228 | 3.30–3.80 | 第 2 組第 2 下 |
| 228–258 | 3.80–4.30 | 第 2 組第 3 下，結束停在休息姿勢 |

### 3.3 Keyframe 清單（Y 位移）

每一下的起點 S ∈ {18, 48, 78, 168, 198, 228}：

| 格 | Y | 說明 |
|---|---|---|
| S | 6 | 往上起點（Soft：out 0.4, 0／in 0.2, 1） |
| S+11 | 0 | 到頂 |
| S+15 | 0 | 停完，往下起點（Soft：out 0.4, 0／in 0.2, 1） |
| S+25 | 6 | 落回休息位置 |

第 253 格落回休息位置後，第 258 格再放一個相同值的 keyframe 收尾（共 26 個 keyframe）。

## 4. 播放規則（給 RD）

| 項目 | 定義 |
|---|---|
| 觸發 | 每次 hint 出現在畫面上（進入可視範圍）就從第 0 格播一次 |
| 循環 | 不循環，播完停在最後一格（休息姿勢） |
| 播放前 | 顯示第 0 格（休息姿勢） |
| hint 被關掉 | 停止並移除，不再播放 |
| 減少動態效果 | iOS「減少動態效果」開啟時不播，只顯示第 0 格 |
| 版面 | 放在 Figma Animation Area：hint 標題列右側。尺寸以 48 × 42pt 為基準，隨螢幕尺寸等比例縮放（aspect fit，不變形） |

## 5. 驗收清單

- [ ] 在 LottieFiles 預覽或 lottie-ios 播放時，六下的頂點落在第 29、59、89、179、209、239 格
- [ ] 第 0 格、最後一格跟 Figma Animation Area 的靜態畫面一致
- [ ] 手指沒有超出 48 × 42（袖口在底緣被裁切）
- [ ] 沒有縮放、旋轉、透明度變化，只有 Y 位移
- [ ] 檔案只含向量 shape，沒有內嵌圖片
- [ ] 跟 prototype 並排播放，節奏一致
