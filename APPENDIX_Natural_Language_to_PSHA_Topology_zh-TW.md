# 附錄——自然語言判斷至 PSHA 拓撲

## 目的

本附錄延伸 PSHA 概念，說明自然語言中的判斷邏輯，如何轉譯為受治理的工作流拓撲。

核心原則不是把自然語言輸出直接視為可執行控制流。自然語言負責表達意圖、條件、依賴、失敗預期與回復需求；PSHA 再將這些語義轉換為可驗證、可執行的結構，最後由 Python Runtime 執行。

```text
自然語言
    ↓
語義意圖
    ↓
工作流中介表示
    ↓
拓撲驗證
    ↓
Python Runtime 執行
    ↓
結構化觀察結果
    ↓
Context 回注
```

這會形成語言、結構、執行與感知之間的閉環。

---

## 自然語言作為高階工作流描述

自然語言判斷通常隱含工作流結構。

例如：

```text
先驗證資料。
如果資料不完整，要求補充缺失欄位。
如果驗證成功，執行分析。
如果分析失敗，重試兩次。
如果仍然失敗，恢復前一狀態並通知操作人員。
```

| 自然語言語義 | 可能對應的 PSHA 拓撲 |
|---|---|
| 先做 A，再做 B | Pipeline |
| 如果 X，做 A，否則做 B | Branch / State Machine |
| A 與 B 可獨立執行 | Parallel / Fan-out |
| A 與 B 完成後才能進入 C | DAG Join |
| 失敗後再次嘗試 | Retry Policy |
| 連續失敗後停止 | Circuit Breaker |
| 恢復前一狀態 | Rollback |
| 執行補償動作 | Saga |
| 通知多個接收者 | Pub-Sub |
| 監看狀態改變 | Observer |
| 持續執行直到條件成立 | Loop |
| 保留輸出供後續使用 | State Store / Memory |

模型可以使用自然語言表達這些關係，但 Runtime 不應直接執行自然語言。自然語言必須先轉換成明確、可檢查的結構。

---

## 語義編譯

PSHA 可以將這個轉譯過程定義為 **語義編譯（Semantic Compilation）**：將自然語言意圖轉換為受治理且可執行的工作流表示。

```text
自然語言判斷
    ↓
意圖抽取
    ↓
結構解讀
    ↓
工作流中介表示
    ↓
拓撲編譯
    ↓
Runtime 執行
```

模型可以提出應該發生什麼。Runtime 則負責判斷哪些行為被允許，以及如何安全執行。

---

## 工作流中介表示

Workflow Intermediate Representation，中文可稱為工作流中介表示，簡稱 Workflow IR。

Workflow IR 位於自然語言與 Python 執行之間。它不是模型原始輸出，也不是不受限制的 Python 程式碼，而是一種受約束的結構表示，可描述工作流節點、依賴關係、條件、狀態轉移、重試上限、Timeout 規則、補償行為、失敗路徑、允許動作與終止條件。

```yaml
workflow:
  topology: pipeline

steps:
  - id: validate
    on_failure:
      action: request_missing_input

  - id: analyze
    retry:
      max_attempts: 2

  - id: persist
    compensation:
      action: remove_partial_output
```

此示例僅用於說明概念，不代表 PSHA 內部正式 Schema。

Workflow IR 的作用，是讓工作流意圖具備可檢查性、可序列化、可測試、可稽核、可版本化、可編譯，以及在不安全或不完整時可被拒絕。

---

## 拓撲選擇

自然語言語義不一定只對應一種拓撲。

例如：

```text
如果服務失敗，就改用另一個供應者。
```

這可能被表示為 Fallback Chain、Conditional Routing、Retry with Provider Switch、Circuit Breaker 或 State Machine Transition。

正確拓撲取決於失敗類型、是否允許重試、是否存在替代服務、狀態一致性、補償需求、成本、延遲與是否允許重複執行。

因此 PSHA 需要一套 **Topology Selection Policy（拓撲選擇策略）**。

```text
存在強順序依賴 → Pipeline
存在多分支依賴 → DAG
存在明確狀態轉移 → State Machine
存在跨步驟補償 → Saga
存在不穩定外部依賴 → Circuit Breaker
存在多個事件消費者 → Pub-Sub
存在可獨立並行工作 → Parallel / Fan-out
```

拓撲選擇策略可以結合確定性規則、模型提出的候選方案、Runtime 限制、驗證結果、人工審核與領域治理規則。

模型不應擁有不受限制的拓撲選擇與建構權限。

---

## 責任邊界

### 模型責任

- 語義解讀
- 意圖抽取
- 候選策略生成
- 下一步建議
- 例外說明
- 拓撲提案
- 模糊點辨識

### Runtime 責任

- 拓撲執行
- 狀態一致性
- 權限控管
- 重試
- Timeout
- Rollback
- Compensation
- 並行控制
- 驗證
- 持久化
- 稽核
- 安全規則執行

```text
模型決定應該嘗試什麼。

Runtime 決定什麼被允許、什麼有效，以及如何執行。
```

因此，自然語言判斷不等於 Runtime 控制權。

---

## Context 回注

執行後，Python Runtime 應將結構化觀察結果回傳給模型。

```json
{
  "status": "failed",
  "node": "validate_input",
  "reason": "missing customer_id",
  "retry_count": 0,
  "allowed_actions": [
    "request_missing_input",
    "abort"
  ]
}
```

這個觀察結果會被重新注入模型 Context。模型感知執行結果後，再選擇下一個被允許的語義動作。

```text
語言
    ↓
結構
    ↓
執行
    ↓
觀察結果
    ↓
Context
    ↓
語言
```

這會形成感知閉環，同時讓執行持續受到 Runtime 治理。

---

## 驗證要求

符合 PSHA 的 Validator 應能拒絕不完整或不安全的結構，包括：

- 節點不存在
- 依賴關係無效
- 無環拓撲中存在 Cycle
- 不可到達狀態
- 缺少終止條件
- 無上限重試
- 缺少補償步驟
- 未定義失敗路徑
- 無效狀態轉移
- 未授權動作
- 不安全外部呼叫
- 未解決歧義
- Context Contract 不完整

拓撲只有通過結構與政策驗證後，才具備執行資格。

---

## 設計期與執行期模式

### 設計期模式

```text
使用者描述工作流
    ↓
模型提出結構
    ↓
人工審核
    ↓
建立 PSHA 表示
```

### 執行期模式

```text
模型讀取目前 Context
    ↓
模型選擇被允許的下一步
    ↓
Python Runtime 執行
    ↓
結構化結果回注
```

設計期模式較容易驗證，通常應先於執行期自治。

---

## 安全原則

PSHA 不應將模型生成 Python 程式碼，作為預設執行路徑。

```text
自然語言
    ↓
結構化意圖
    ↓
Workflow IR
    ↓
Validator
    ↓
Topology Compiler
    ↓
Python Runtime
```

這種方式可以降低任意程式碼執行風險、隱藏控制流錯誤、無效狀態轉移、不完整失敗處理、拓撲歧義、無界重試、靜默資料遺失與不安全工具使用。

模型維持語義判斷角色。Python Runtime 維持受治理的執行權限。

---

## 總結

```text
自然語言判斷
≠ 可執行 Runtime 控制

自然語言判斷
→ 語義意圖

PSHA
→ 受治理的可執行拓撲
```

PSHA 可以成為語言層推理與工程層工作流結構之間的橋接層。它的角色是將語義意圖轉換為可驗證、可觀察、可治理的拓撲。
