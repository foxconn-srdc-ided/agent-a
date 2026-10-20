# CSV 統計 CLI 工具規格說明

## 功能需求

開發一個命令列工具 `csv_stats.py`，使用 Python 標準函式庫處理 CSV 檔案，計算包含 `category` 和 `amount` 欄位的統計數據。

### 輸入格式

- 檔案格式：CSV
- 欄位要求：
  - `category`: 文字型態
  - `amount`: 數值型態（Decimal）
- 精確度：金額保留兩位小數

### 輸出格式

輸出 JSON 格式，包含以下鍵值：

```json
{
  "total_count": 0,
  "total_amount": "0.00",
  "category_stats": {}
}
```

#### 說明

- `total_count`: 總筆數（int）
- `total_amount`: 總金額（str，保留兩位小數）
- `category_stats`: 分類統計，鍵為分類名稱，值為包含 `count` 和 `amount` 的物件

### 特殊處理

1. **空白分類**：將空白分類歸類為 `'UNKNOWN'`
2. **負金額**：支援負數金額計算
3. **異常處理**：
   - 只有標頭：返回空結果
   - 缺欄位：拋出 `FieldError`，CLI 輸出錯誤訊息至 stderr，並以非零退出碼退出
   - 非法金額：拋出 `ValueError`，CLI 輸出錯誤訊息至 stderr，並以非零退出碼退出

### 範例

假設輸入 CSV 如下：

```
category,amount
food,100.50
transport,50.25
food,-20.00
```

則輸出應為：

```json
{
  "total_count": 3,
  "total_amount": "130.75",
  "category_stats": {
    "food": {
      "count": 2,
      "amount": "80.50"
    },
    "transport": {
      "count": 1,
      "amount": "50.25"
    }
  }
}
```

## 程式設計要求

- 使用 Python 標準函式庫
- 使用 `csv` 模組讀檔
- 使用 `decimal.Decimal` 進行金額運算
- 使用 `json` 模組輸出結果
- 支援命令列參數，接收 CSV 檔案路徑
- 異常處理需明確拋出並正確回傳錯誤訊息
