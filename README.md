# DoL 雲端服務設置.md

【此篇為寫給自己看的教學，過程有利用 Microsoft Copilot】

適用於 DoL + Maplebirch Cloud Save  
使用 Cloudflare Worker + R2 建立自己的雲端存檔服務。

本文件參考 https://github.com/MaplebirchLeaf/SCML-DOL-maplebirchFramework/blob/main/docs/CN/CloudSave.md 進行操作。  
本人依照此文件流程成功部署。

---

# 事前準備

### 1. Cloudflare 帳號

註冊並登入（要搞金融卡）：

```text
https://dash.cloudflare.com
```

---

### 2. 安裝 Node.js

下載 LTS 版本：

```text
https://nodejs.org
```

安裝完成後開啟 CMD 驗證：

```cmd
node -v
npm -v
```

---

# 建立 R2 儲存空間

### 1. 進入 R2 Object Storage

從 Cloudflare 左側清單選擇：

```text
R2 Object Storage
```

若首次使用：

```text
Add R2 subscription to my account
```

---

### 2. 建立 Bucket

```text
Bucket Name:
maplebirch-save

Location:
Automatic

Default Storage Class:
Standard
```

---

# 安裝 Wrangler

```cmd
npm install -g wrangler
```

驗證：

```cmd
wrangler --version
```

---

# 準備 Worker 檔案

建立資料夾：

```text
D:\dol-cloud\
```

放入：

```text
worker.ts
wrangler.jsonc
```

## 更改 wrangler.jsonc

```json
{
  "name": "maplebirch-cloud-save",
  "main": "worker.ts",
  "compatibility_date": "2026-09-08",
  "r2_buckets": [
    {
      "binding": "SAVE_BUCKET",
      "bucket_name": "maplebirch-save"
    }
  ],
  "secrets": {
    "required": ["MAPLEBIRCH_TOKEN"]
  }
}
```

---

# 登入 Cloudflare

```cmd
cd /d D:\dol-cloud
wrangler login
```

---

# 建立訪問令牌

```cmd
wrangler secret put MAPLEBIRCH_TOKEN
```

輸入自行設定的 Token。

---

# 部署 Worker

檢查：
```cmd
wrangler deploy
```

應該會得到：
```
Total Upload: 7.49 KiB / gzip: 2.30 KiB

The following bindings need to be provisioned:
Binding                 Resource
env.SAVE_BUCKET         R2 Bucket


Provisioning SAVE_BUCKET (R2 Bucket)...
Resource name found in config: maplebirch-save
🌀 Creating new R2 Bucket "maplebirch-save"...
✨ SAVE_BUCKET provisioned 🎉

Your Worker was deployed with provisioned resources. We've written the IDs of these resources to your config file, which you can choose to save or discard. Either way future deploys will continue to work.
🎉 Resources provisioned, continuing with deployment...

Worker Startup Time: 1 ms
Your Worker has access to the following bindings:
Binding                                Resource
env.SAVE_BUCKET (maplebirch-save)      R2 Bucket

Uploaded maplebirch-cloud-save (7.82 sec)
▲ [WARNING] Because 'workers_dev' is not in your wrangler.jsonc, it will be enabled for this deployment by default.

  To override this setting, you can disable workers.dev by explicitly setting 'workers_dev = false'
  in your wrangler.jsonc.


▲ [WARNING] Because your 'workers.dev' route is enabled and your 'preview_urls' setting is not in your wrangler.jsonc, Preview URLs will be enabled for this deployment by default.

  To override this setting, you can disable Preview URLs by explicitly setting 'preview_urls =
  false' in your wrangler.jsonc.


Deployed maplebirch-cloud-save triggers (1.01 sec)
  https://maplebirch-cloud-save.[帳號].workers.dev
Current Version ID: ********
```

下面那個就是地址：

```text
https://maplebirch-cloud-save.[帳號].workers.dev
```

---

# 測試 Worker

```text
https://maplebirch-cloud-save.[帳號].workers.dev/health
```

若顯示：

```json
{"ok":true}
```

代表運作正常。

---

# 遊戲設定

地址：

```text
https://maplebirch-cloud-save.[帳號].workers.dev
```

訪問令牌：

```text
你設定的 MAPLEBIRCH_TOKEN
```

---

# FAQ

## 訪問令牌會過期嗎？

不會，除非自行修改 MAPLEBIRCH_TOKEN。

## 每次玩遊戲都要重新部署嗎？

不用，只需首次部署一次。

## 換電腦怎麼辦？

重新安裝 Node.js、Wrangler，並登入同一個 Cloudflare 帳號即可。

## 忘記訪問令牌怎麼辦？

重新執行：

```cmd
wrangler secret put MAPLEBIRCH_TOKEN
```

設定新的 Token。
