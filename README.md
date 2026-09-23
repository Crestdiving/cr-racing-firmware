# cr-racing-firmware

**賽車錶韌體的公開發佈來源，兩個產品：QSTARZ（EOQ05）與 STINT GTM（EOS05）。**
App 從這裡查有沒有新版、去哪拿、拿到的對不對。

**產物與清單而已 —— 沒有韌體原始碼。** 原始碼在私有的
`Crestdiving/Claude_EOC05`；這個 repo 只放發佈出去的東西。

---

## 一個產品一份清單

| 產品 | 錶回報的版號前綴 | 清單檔 | 清單內 `product` |
|---|---|---|---|
| QSTARZ | `QSZ`（例 `QSZ004`） | `releases.json` | `cr-racing`（歷史名稱，不改） |
| STINT GTM | `GTM`（例 `GTM001`） | `releases-stint-gtm.json` | `stint-gtm` |

兩個產品的韌體**不能互裝**，錶上的 bootloader 不會檢查產品身分，所以清單絕不混放：
QSTARZ 的版本只進 `releases.json`，STINT GTM 的只進 `releases-stint-gtm.json`。發佈工具與驗證器都會擋。

## App 要打的網址

清單（每次檢查更新時取，依錶回報版號的前 3 字元選一份）：

```
https://raw.githubusercontent.com/Crestdiving/cr-racing-firmware/main/releases.json
https://raw.githubusercontent.com/Crestdiving/cr-racing-firmware/main/releases-stint-gtm.json
```

產物（下載時取，網址由清單的 `artifact.url` 給，不要自己組）：

```
https://github.com/Crestdiving/cr-racing-firmware/releases/download/<版號>/<版號>.bin
```

全部公開、免驗證。與 `cr-racing-speedcam-data` 的 `cameras.bin` 同一個模式。
清單檔不存在（404）代表該產品尚未發佈任何版本，App 當作「沒有更新」。

---

## 東西放哪、為什麼

| 內容 | 位置 | 理由 |
|---|---|---|
| `releases.json`、`releases-stint-gtm.json`（幾 KB 文字） | git 追蹤的檔案 | 要有版本歷史；改一筆就是一個 commit |
| `QSZ###.bin`、`GTM###.bin`（約 1.3 MB） | **GitHub Release asset** | 存在 git 之外。git 會永遠留著每一個版本，20 版就是 26MB 的歷史且刪不掉 |

**韌體 `.bin` 絕對不要 commit 進這個 repo。**

Release 的 tag 就用裝置版號本身（`QSZ003`、`GTM001`），與清單的
`device_version` 一致。

---

## 契約

清單的欄位定義、驗證規則與 App 端流程，唯一來源是韌體 repo 的

```
docs/protocol/firmware_release_manifest_v1.md
```

App 與 Web **只實作、不重新定義**。這個 repo 不重述契約內容，避免兩份文件漂移。

重點四條：

- **`artifact.bytes` 不得超過 1 792 000（1750 KiB）** —— 超過會讓錶上 bootloader
  卡在 SHA-1 驗證迴圈變磚。已出貨錶的 bootloader 無法 OTA 更新，所以發佈端把關
  是主防線。
- **`device_ota_filename` 必須採用清單裡的值**，不可從產物檔名推論。目前所有產品都是
  `EOC5L.BIN`——現役錶的 bootloader（0.0.7）只認這個檔名。推錯的失敗模式是靜默沒反應。
- **版號永不重用**，排序只看前綴後三碼的整數值，且只在同一份清單內比較。不支援降版。
- **產品不可混放**：`releases.json` 只能有 `QSZ` 版號，`releases-stint-gtm.json` 只能有 `GTM` 版號。

---

## 發佈一版

在韌體 repo 打包完之後，一支指令做完（建 Release、上傳產物、登錄清單、commit、push）：

```bash
python3 tools/release/publish_release.py \
  --artifact-dir EOC05/MDK-ARM/package_output/EOS05/GTM001 \
  --firmware-repo <本 repo 的本地 clone> \
  --display-version 0.0.1 --notes "STINT GTM 發佈流程測試"
```

產品由打包證據（`packaging-evidence.json`）的 `product_id` 決定：`EOQ05` 進 `releases.json`、
`EOS05` 進 `releases-stint-gtm.json`，不用也不能自己指定。工具是冪等的：中途失敗重跑同一指令，
做過的步驟會略過。它會在這些情況拒絕：打包時工作區是髒的、版號已經在清單裡、產物與 evidence
的 SHA-256 不符、同 tag 已有不同內容的 Release、產出的清單過不了驗證器。

需要 `gh` 已登入且對本 repo 有 push 權限；工具不讀、不存任何憑證。

## 撤回一版

把那筆從對應的清單檔刪掉再 push。**Release asset 與 tag 保留不刪**，
歷史要查得回去；**那個版號永遠不能再用**，重新 build 必須換新版號。

已經裝上去的錶不受影響 —— 系統不支援降版，撤回保護的是還沒更新的錶。
