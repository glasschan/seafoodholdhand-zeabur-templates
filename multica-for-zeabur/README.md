# Multica Template for Zeabur

這是一個為 nocode 用戶設計的 Multica 模板，專為 Zeabur 平台部署而創建。

[![Deploy on Zeabur](https://zeabur.com/button.svg)](https://seafoodholdhand.com/recommends/zeabur-multica-template/)

## 模板連結
[Zeabur Multica Template](https://seafoodholdhand.com/recommends/zeabur-multica-template/)

## 特點

- 開箱即用：模板已經按照邏輯編寫並經過測試，可以成功運行
- 簡單易用：專為 nocode 用戶設計，無需編程經驗
- 快速部署：一鍵部署到 Zeabur 平台
- 包含全部三個服務：PostgreSQL 17 (pgvector) + Backend + Frontend

## 關於環境變數

此模板包含以下可配置的環境變數：

- `PUBLIC_DOMAIN`：你的 Multica 前端網域
- `API_DOMAIN`：你的 Multica 後端 API 網域
- `JWT_SECRET`：用於簽署 JWT 令牌的密鑰（自動生成）
- `RESEND_API_KEY`：可選，用於郵件登入。留空則需從後端日誌獲取驗證碼

## 登入方式

部署完成後，打開你的網域。在登入頁面輸入你的郵箱，然後通過以下方式獲取驗證碼：

- **推薦**：設定 `RESEND_API_KEY` → 驗證碼會發送到你的郵箱
- **沒有 Resend**：打開 Zeabur 控制台中的 **Backend** 服務日誌 → 搜索 `[DEV] Verification code for` → 複製 6 位驗證碼並貼到登入頁面

## 使用方法

1. [點擊這裡](https://seafoodholdhand.com/recommends/zeabur-multica-template/) 使用模版部署
2. 等待所有服務啟動完成（Backend 會自動執行資料庫遷移）
3. 打開你的網域並登入
4. 安裝 CLI：`brew install multica-ai/tap/multica`
5. 配置並啟動 daemon（替換為你的實際域名）：
   ```bash
   multica setup self-host --server-url https://your-api-domain.zeabur.app --app-url https://your-multica-domain.zeabur.app
   ```
   自建部署使用自定義域名：
   ```bash
   multica setup self-host --server-url https://api.example.com --app-url https://app.example.com
   ```
6. 開始使用 Multica 管理你的 AI 代理團隊！

## 關於作者

我是一個 nocode 的普通用家，請多多指教。有空來我網站看看 [SEAFOODHOLDHAND 史佛浩恒](https://seafoodholdhand.com)

## 反饋與改進

如果你有任何建議或改進意見，歡迎在 GitHub 上提交 issue。雖然我不太懂 GitHub 的使用，但我會通過 AI 的幫助盡力更新和維護這個模板。

如在 Deploy 時遇到任何問題，可以到 Github issue 告訴我，我會盡力解決：
https://github.com/glasschan/seafoodholdhand-zeabur-templates

## 許可證

本項目採用 MIT 許可證 - 詳情請參閱 LICENSE 文件
