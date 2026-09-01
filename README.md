# cr-racing-firmware

**CR-Racing（QSTARZ / EOQ05）手錶韌體的公開發佈來源。**
App 從這裡查有沒有新版、去哪拿、拿到的對不對。

**產物與清單而已 —— 沒有韌體原始碼。** 原始碼在私有的
`Crestdiving/Claude_EOC05`；這個 repo 只放發佈出去的東西。

---

## App 要打的兩個網址

清單（每次檢查更新時取）：

```
https://raw.githubusercontent.com/Crestdiving/cr-racing-firmware/main/releases.json
```

產物（下載時取，網址由清單的 `artifact.url` 給，不要自己組）：

```
https://github.com/Crestdiving/cr-racing-firmware/releases/download/QSZ003/QSZ003.bin
```

兩個都公開、免驗證。與 `cr-racing-speedcam-data` 的 `cameras.bin` 同一個模式。

---

## 東西放哪、為什麼

| 內容 | 位置 | 理由 |
|---|---|---|
| `releases.json`（幾 KB 文字） | git 追蹤的檔案 | 要有版本歷史；改一筆就是一個 commit |
| `QSZ###.bin`（約 1.2 MB） | **GitHub Release asset** | 存在 git 之外。git 會永遠留著每一個版本，20 版就是 24MB 的歷史且刪不掉 |

**韌體 `.bin` 絕對不要 commit 進這個 repo。**

Release 的 tag 就用裝置版號本身（`QSZ003`），與 `releases.json` 的
`device_version` 一致。

---

## 契約

`releases.json` 的欄位定義、驗證規則與 App 端流程，唯一來源是韌體 repo 的

```
docs/protocol/firmware_release_manifest_v1.md
```

App 與 Web **只實作、不重新定義**。這個 repo 不重述契約內容，避免兩份文件漂移。

重點三條：

- **`artifact.bytes` 不得超過 1 792 000（1750 KiB）** —— 超過會讓錶上 bootloader
  卡在 SHA-1 驗證迴圈變磚。已出貨錶的 bootloader 無法 OTA 更新，所以發佈端把關
  是主防線。
- **`device_ota_filename` 必須採用清單裡的值**（QSTARZ 是 `FW.BIN`），
  不可從產物檔名推論。推錯的失敗模式是靜默沒反應。
- **版號永不重用**，排序只看 `QSZ` 後三碼的整數值。不支援降版。

---

## 發佈一版

在韌體 repo 打包完之後：

```bash
# 1. 上傳產物，tag 用版號
gh release create QSZ003 -R Crestdiving/cr-racing-firmware \
  package_output/EOQ05/QSZ003/QSZ003.bin \
  --title QSZ003 --notes "GPS Auto Lap 分圈更穩定"

# 2. 登錄進清單（版號 / 檔名 / commit / bytes / sha256 都從 evidence 讀）
python3 tools/release/add_release.py \
  --evidence package_output/EOQ05/QSZ003/packaging-evidence.json \
  --releases <本 repo>/releases.json \
  --display-version 2.2.0 \
  --url https://github.com/Crestdiving/cr-racing-firmware/releases/download/QSZ003/QSZ003.bin \
  --channel beta --notes "GPS Auto Lap 分圈更穩定。"

# 3. commit + push releases.json
```

`add_release.py` 會在三種情況拒絕：打包時工作區是髒的、版號已經在清單裡、
產出的清單過不了驗證器。

## 撤回一版

把那筆從 `releases.json` 刪掉再 push。**Release asset 與 tag 保留不刪**，
歷史要查得回去。

已經裝上去的錶不受影響 —— 系統不支援降版，撤回保護的是還沒更新的錶。
