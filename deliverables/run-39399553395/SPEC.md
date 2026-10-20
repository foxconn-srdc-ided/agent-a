# CSV 統計 CLI 工具規格

## 資料格式

- **CSV 檔案**：包含 `category` 和 `amount` 兩欄位。
- **category**：文字型態。
- **amount**：數值型態，使用 `Decimal` 類型進行運算，精確到分（2位小數）。

## 輸出格式

輸出為 JSON 格式，包含以下鍵值：

- `total_count`: int
- `total_amount`: str（保留兩位小數的字串，例如 "250.25"），運算仍使用 `Decimal`
- `category_stats`: dict，鍵為分類名稱，值為包含 `count` 和 `amount` 的物件，其中 `amount` 同樣為保留兩位小數的字串

## 特殊情況處理

- **空白分類**：將空白分類歸類為 `'UNKNOWN'`。
- **只有標頭**：返回空結果，輸出為 `{"total_count": 0, "total_amount": "0.00", "category_stats": {}}`。
- **缺欄位**：拋出 `FieldError` 異常，CLI 輸出錯誤訊息至 `stderr`，並以非零退出碼退出。
- **非法金額**：拋出 `ValueError` 異常，CLI 輸出錯誤訊息至 `stderr`，並以非零退出碼退出。

## 錯誤訊息範例

- 缺欄位錯誤：`ERROR: Missing required field(s): category, amount`
- 非法金額錯誤：`ERROR: Invalid amount value: not_a_number`

## 退出碼

- 0：成功
- 1：發生錯誤（如缺欄位或非法金額）
