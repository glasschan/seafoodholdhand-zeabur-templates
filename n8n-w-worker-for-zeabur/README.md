# n8n Worker Template for Zeabur

這是一個為 nocode 用戶設計的 n8n worker 模板，專為 Zeabur 平台部署而創建。

[![Deploy on Zeabur](https://zeabur.com/button.svg)](https://zeabur.com/templates/6LRV95?referralCode=glasschan)

## 模板連結
[Zeabur n8n Worker Template](https://seafoodholdhand.com/recommends/zeabur-n8n-w-worker-template/)

## 特點

- 開箱即用：模板已經按照邏輯編寫並經過測試，可以成功運行
- 簡單易用：專為 nocode 用戶設計，無需編程經驗
- 快速部署：一鍵部署到 Zeabur 平台
- 未來兼容：預先啟用關鍵環境變數，確保與未來 n8n 版本相容

## 更新日誌

### 2025-03-17
- 修復了 n8n 主實例和工作程序之間的 N8N_ENCRYPTION_KEY 不匹配問題
- 確保工作程序使用與主實例相同的加密密鑰，提高系統穩定性

## 關於環境變數

此模板啟用了兩個重要的環境變數以防止未來的相容性問題：

- `N8N_RUNNERS_ENABLED=true`：不使用任務執行器運行 n8n 的方式已被棄用。未來版本將預設開啟任務執行器。此設定確保與即將到來的變更相容。
  
- `OFFLOAD_MANUAL_EXECUTIONS_TO_WORKERS=true`：在擴展模式下於主實例執行手動工作流程的方式已被棄用。未來版本中，手動執行將轉由工作程序處理。此設定為此變更做好準備。

了解更多關於任務執行器的信息：https://docs.n8n.io/hosting/configuration/task-runners/

## 使用方法

1. [點擊這裡](https://seafoodholdhand.com/recommends/zeabur-n8n-w-worker-template/) 按鈕使用這個模版創建你自己的倉庫
2. 按照 Zeabur 的部署指南進行部署或觀看我的[教學影片](https://youtu.be/SmPqXcmHNag)
3. 開始使用 n8n 自動化你的工作流程！

## 關於作者

我是一個 nocode 的普通用家，請多多指教。有空來我網站看看 [SEAFOODHOLDHAND 史佛浩恒](https://seafoodholdhand.com)

## 反饋與改進

如果你有任何建議或改進意見，歡迎在 GitHub 上提交 issue。雖然我不太懂 GitHub 的使用，但我會通過 AI 的幫助盡力更新和維護這個模板。

如在 Deploy 時遇到任何問題，可以到 Github issue 告訴我，我會盡力解決：
https://github.com/glasschan/seafoodholdhand-zeabur-templates

## 許可證

本項目採用 MIT 許可證 - 詳情請參閱 LICENSE 文件
