# CSV 統計 CLI 工具規格說明

## 功能描述

此工具接受一個 CSV 檔案作為輸入，檔案需包含 `category` 與 `amount` 兩欄位。工具會計算以下統計資訊，並以 JSON 格式輸出：

- 總筆數 (`total_count`)
- 總金額 (`total_amount`)
- 分類統計 (`category_stats`)：每一分類的筆數與金額

## 資料格式

### 輸入 CSV

- 欄位順序：`category`, `amount`
- `category`：文字型態
- `amount`：數值型態，使用 `Decimal` 進行精確運算，保留兩位小數

### 輸出 JSON

```json
{
  "total_count": 3,
  "total_amount": "250.25",
  "category_stats": {
    "食品": {
      "count": 2,
      "amount": "150.00"
    },
    "交通": {
      "count": 1,
      "amount": "100.25"
    }
  }
}
```

## 特殊處理

- **空白分類**：若 `category` 為空白，則歸類為 `'UNKNOWN'`
- **負金額**：支援負數金額，正確參與總金額與分類金額計算

## 錯誤處理

### 只有標頭

輸入僅有標頭時，輸出為：

```json
{"total_count": 0, "total_amount": "0.00", "category_stats": {}}
```

### 缺欄位

當 CSV 檔案缺少 `category` 或 `amount` 欄位時，工具會拋出 `FieldError` 異常，CLI 輸出以下錯誤訊息至 `stderr`，並以非零退出碼退出：

```
錯誤：CSV 檔案缺少必要欄位。
```

### 非法金額

若 `amount` 欄位無法轉換為 `Decimal`，則拋出 `ValueError`，CLI 輸出錯誤訊息至 `stderr`，並以非零退出碼退出：

```
錯誤：金額格式無效。
```

## 測試範例

### 正常輸入範例

假設輸入 CSV 內容如下：

```
category,amount
食品,100.00
交通,100.25
食品,50.00
```

對應輸出為：

```json
{
  "total_count": 3,
  "total_amount": "250.25",
  "category_stats": {
    "食品": {
      "count": 2,
      "amount": "150.00"
    },
    "交通": {
      "count": 1,
      "amount": "100.25"
    }
  }
}
```

### 負金額輸入範例

```
category,amount
食品,-50.00
交通,100.25
```

對應輸出為：

```json
{
  "total_count": 2,
  "total_amount": "50.25",
  "category_stats": {
    "食品": {
      "count": 1,
      "amount": "-50.00"
    },
    "交通": {
      "count": 1,
      "amount": "100.25"
    }
  }
}
```

### 空白分類輸入範例

```
category,amount
食品,100.00
,50.00
```

對應輸出為：

```json
{
  "total_count": 2,
  "total_amount": "150.00",
  "category_stats": {
    "食品": {
      "count": 1,
      "amount": "100.00"
    },
    "UNKNOWN": {
      "count": 1,
      "amount": "50.00"
    }
  }
}
```

### 缺欄位錯誤輸入範例

```
category
食品
```

CLI 輸出錯誤訊息至 stderr：

```
錯誤：CSV 檔案缺少必要欄位。
```

退出碼：1

### 非法金額錯誤輸入範例

```
category,amount
食品,abc
```

CLI 輸出錯誤訊息至 stderr：

```
錯誤：金額格式無效。
```

退出碼：1
