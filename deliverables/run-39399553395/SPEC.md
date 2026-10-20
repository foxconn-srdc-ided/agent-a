# CSV 統計 CLI 工具規格

## 功能概述

此工具處理符合特定格式的 CSV 檔案，並輸出 JSON 格式的統計結果。

## 輸入格式

- 檔案必須為 CSV 格式。
- 必須包含 `category` 和 `amount` 兩個欄位。
- `category` 為文字型態。
- `amount` 為數值型態，使用 `Decimal` 類型進行運算，精確到分（2位小數）。

## 輸出格式

輸出為 JSON 格式，包含以下鍵值：

```json
{
  "total_count": int,
  "total_amount": "0.00",
  "category_stats": {
    "分類名稱": {
      "count": int,
      "amount": "0.00"
    }
  }
}
```

### 特殊處理

- **空白分類**：若 `category` 為空白或僅含空白字元，則歸類為 `'UNKNOWN'`。
- **負金額**：支援負數金額，會正確累加至總金額與分類金額。

## 錯誤處理

### 1. 只有標頭

當 CSV 檔案僅包含標頭時，輸出為：

```json
{"total_count": 0, "total_amount": "0.00", "category_stats": {}}
```

### 2. 缺欄位

若 CSV 檔案缺少 `category` 或 `amount` 欄位，則拋出 `FieldError` 異常，CLI 輸出以下錯誤訊息至 stderr，並以非零退出碼退出：

```
Error: Missing required field(s): category, amount
```

### 3. 非法金額

若 `amount` 欄位無法轉換為 `Decimal`，則拋出 `ValueError` 異常，CLI 輸出以下錯誤訊息至 stderr，並以非零退出碼退出：

```
Error: Invalid amount value 'invalid' in row 2
```

## 範例輸入與輸出

### 範例 1：正常輸入

輸入 CSV:

```
category,amount
food,100.50
transport,50.25
food,25.75
```

輸出 JSON:

```json
{
  "total_count": 3,
  "total_amount": "176.50",
  "category_stats": {
    "food": {
      "count": 2,
      "amount": "126.25"
    },
    "transport": {
      "count": 1,
      "amount": "50.25"
    }
  }
}
```

### 範例 2：包含空白分類

輸入 CSV:

```
category,amount
food,100.50
,50.25
food,25.75
```

輸出 JSON:

```json
{
  "total_count": 3,
  "total_amount": "176.50",
  "category_stats": {
    "food": {
      "count": 2,
      "amount": "126.25"
    },
    "UNKNOWN": {
      "count": 1,
      "amount": "50.25"
    }
  }
}
```

### 範例 3：包含負金額

輸入 CSV:

```
category,amount
food,-100.50
transport,50.25
food,25.75
```

輸出 JSON:

```json
{
  "total_count": 3,
  "total_amount": "-24.50",
  "category_stats": {
    "food": {
      "count": 2,
      "amount": "-74.75"
    },
    "transport": {
      "count": 1,
      "amount": "50.25"
    }
  }
}
```

### 範例 4：只有標頭

輸入 CSV:

```
category,amount
```

輸出 JSON:

```json
{"total_count": 0, "total_amount": "0.00", "category_stats": {}}
```
