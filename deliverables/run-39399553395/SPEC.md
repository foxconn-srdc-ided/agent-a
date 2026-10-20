# CSV 統計 CLI 工具規格說明

## 功能需求

開發一個 Python 標準函式庫實現的 CSV 統計命令列工具，處理包含 `category` 和 `amount` 欄位的 CSV 檔案。

### 輸入格式

- CSV 檔案必須包含兩個欄位：`category`（文字）和 `amount`（數值）
- 使用逗號分隔

### 輸出格式

輸出為 JSON 格式，包含以下欄位：

```json
{
  "total_count": int,
  "total_amount": Decimal,
  "category_stats": {
    "category": {
      "count": int,
      "amount": Decimal
    }
  }
}
```

## 資料處理規則

1. **amount 欄位**：使用 `Decimal` 類型，保留兩位小數。
2. **category 欄位**：若為空白，視為 `'UNKNOWN'`。
3. **負數金額**：允許並參與計算。

## 錯誤處理

| 情況 | 行為 |
|------|------|
| 只有標頭 | 回傳空結果 |
| 缺少欄位 | 抛出 `FieldError` 異常 |
| 金額非數字 | 抛出 `ValueError` 異常 |

## 範例

### 輸入
```
category,amount
食品,100.50
交通,200.00
食品,-50.25
```

### 輸出
```json
{"total_count": 3, "total_amount": 250.25, "category_stats": {"食品": {"count": 2, "amount": 50.25}, "交通": {"count": 1, "amount": 200.00}}}
```

## 測試案例

請確保測試能覆蓋以下情況：

1. 正常運作（含正/負金額）
2. 負金額
3. 空白分類
4. 只有標頭
5. 缺欄位
6. 非法金額

## 執行方式

```bash
python3 csv_stats.py input.csv
```
