# Spec Delta

## Purpose

提供可依狀態查詢的裝置清單，讓 API 消費者取得目前可用的裝置資訊及明確的查詢錯誤。

## ADDED Requirements

### Requirement: 取得裝置清單

系統 SHALL 提供 `GET /api/devices`，在未提供 `status` 時以 HTTP 200 回傳所有裝置的 JSON 陣列。每個裝置 SHALL 包含數字 `id`、字串 `name`、`model`、`ipAddress`、值為 `online`、`offline` 或 `warning` 的 `status`，以及 ISO 8601 格式的 `lastSeen` 時間。

#### Scenario: 未篩選時取得所有裝置
- **WHEN** 用戶端呼叫 `GET /api/devices` 而未提供 `status`
- **THEN** 系統回傳 HTTP 200 和所有裝置的 JSON 陣列，每個項目包含指定欄位及格式

### Requirement: 依狀態篩選裝置

系統 SHALL 支援以 `status` 查詢參數篩選清單，接受精確的 `online`、`offline` 和 `warning` 值，並以 HTTP 200 回傳狀態相符的裝置 JSON 陣列；沒有符合的裝置時 SHALL 回傳空陣列。

#### Scenario: 篩選成功
- **WHEN** 用戶端呼叫 `GET /api/devices?status=online` 且存在 online 裝置
- **THEN** 系統回傳 HTTP 200，陣列包含所有且僅包含 `status` 為 `online` 的裝置

#### Scenario: 沒有符合的裝置
- **WHEN** 用戶端以合法的 `status` 查詢，且資料來源沒有該狀態的裝置
- **THEN** 系統回傳 HTTP 200 和空 JSON 陣列 `[]`

### Requirement: 拒絕不合法狀態

系統 SHALL 對提供的非法 `status` 值回傳 HTTP 400 和 ASP.NET Core ProblemDetails 錯誤回應，不得將其視為未篩選查詢。

#### Scenario: status 不合法
- **WHEN** 用戶端呼叫 `GET /api/devices?status=invalid`
- **THEN** 系統回傳 HTTP 400，回應內容為 ProblemDetails，且其 `status` 為 400

#### Scenario: status 為空
- **WHEN** 用戶端呼叫 `GET /api/devices?status=`
- **THEN** 系統回傳 HTTP 400，回應內容為 ProblemDetails，且其 `status` 為 400

### Requirement: 啟動假資料

系統 SHALL 在啟動時於記憶體建立 20 台假裝置，供裝置清單查詢使用。

#### Scenario: 啟動後查詢假資料
- **WHEN** 系統完成啟動後，用戶端呼叫不帶篩選的 `GET /api/devices`
- **THEN** 系統回傳 HTTP 200 和包含 20 台假裝置的陣列