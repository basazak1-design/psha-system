# PSHA System — Python型態 Skill-Hook 拓撲架構

**(Python-Structural Hook Architecture for AI Agent Workflows)**

> 把 Python 的結構概念——設計模式、拓撲、狀態管理——作為 AI 代理 skill↔hook 工作流程的架構語言，而不只是實作工具。

---

## 概述

大多數 AI 代理框架用 Python *執行*工作流程，但把工作流程的結構定義為平鋪的 prompt 指令或線性步驟序列。PSHA 翻轉了這個做法：Python 的設計詞彙成為決定 skill 與 hook 如何連接、協作與失敗的**形狀與拓撲**的語言。

核心洞見：如果 AI 框架是用 Python 建造的，那麼 Python 的結構思維也應該用來*設計*這些框架——不只是讓它們跑起來。

---

## 核心概念

```
工作流程的結構不是一份步驟清單。
它是一個有形狀的拓撲。
那個形狀用 Python 來描述。
```

**傳統做法：**
```
Skill → Hook → 執行 → 結束
（線性、平鋪、沒有結構詞彙）
```

**PSHA 做法：**
```
Skill（意圖層）
  └─ Hook（生命週期綁定，在定義時配對）
       └─ Python（架構語言層）
            ├─ 拓撲選擇：Pipeline / DAG / Circuit Breaker / Saga / Pub-Sub
            ├─ 設計模式：狀態機 / 回滾 / 防禦性設計 / 觀察者
            └─ Context 回注（感知層 → AI 感知輸出，閉環完成）
```

---

## 三層架構

| 層次 | 職責 | 機制 |
|------|------|------|
| **意圖層**（Skill） | 定義流程目標與步驟語義 | 在 YAML frontmatter 中攜帶 hook 配置 |
| **架構語言層**（Python） | 決定流程的形狀 | 設計模式與拓撲結構套用於 skill↔hook 管道 |
| **感知層**（Context 回注） | AI 感知 Python 輸出並決定下一步 | 結構化輸出回注進模型 context，完成閉環 |

---

## 設計原則

PSHA 建立在四個結構原則上，分三個粒度層次套用（單元 → 節點組合 → 流程串接）。每個原則對應到既有的 Python 設計詞彙——在此作為流程形狀的架構決策，而非單純的實作選擇。

*詳細規範未公開。有合作意向請直接聯繫。*

---

## 最小範式

以下示意核心結構概念——Python 模式詞彙決定流程拓撲，而不只是執行它：

```python
# Skill 在定義時攜帶配對 hook（生命週期綁定）
# YAML frontmatter 嵌入 hook 配置——不依賴全域 hook 登錄表

# Python 層依流程需求選擇拓撲
class SkillHookBridge:
    topology: str  # "pipeline" | "circuit_breaker" | "dag" | "saga" | ...

    def on_trigger(self, event):
        result = self.apply_topology(event)      # 形狀由 Python 模式決定
        return self.inject_context(result)        # 輸出回注 → AI 感知 → 閉環完成
```

這是結構示意。目前已有部分可運作實作，並在正式 AI 代理系統中驗證。

---

## 為什麼這很重要

業界框架在問：*「我應該用哪個框架？」*

PSHA 在問：*「這個流程的形狀應該長什麼樣？」*

這是兩個不同層次的問題。第二個問題長期無人系統性回答——因為工程背景的人優化的是執行，不是結構。Python 的設計詞彙一直都在，只是從未被套用到 AI 工作流程的架構層。

---

## 開發狀態

| 項目 | 狀態 |
|------|------|
| 概念設計 | ✅ 完成（2026-04-04） |
| 架構規範 | ✅ 完成 |
| 部分實作 | ✅ 已在正式系統驗證 |
| 完整開源發布 | ⏳ 規劃中 |

---

## 聯繫

有合作、研究或實作細節詢問：
**basazak1@gmail.com**

---

## 作者

**楊仕弘（William Yang）**
概念起源：2026-04-04

---

*授權：CC BY 4.0（概念文件）· 實作細節保留版權*
