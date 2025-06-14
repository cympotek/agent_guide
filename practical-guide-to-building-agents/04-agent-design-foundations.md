# 代理設計基礎

在最基本的形式中，代理由三個核心組件組成：

### 01 模型
驅動代理推理和決策制定的大型語言模型

### 02 工具
代理可用來採取行動的外部函數或API

### 03 指令
定義代理行為方式的明確指導方針和護欄

以下是使用 OpenAI 的代理 SDK 時在程式碼中的樣子。您也可以使用您偏好的函式庫或從頭開始直接建構相同的概念。

```python
weather_agent = Agent(
    name="Weather agent",
    instructions="You are a helpful agent who can talk to users about the weather.",
    tools=[get_weather],
)
```

## 選擇您的模型

不同的模型在任務複雜性、延遲和成本方面有不同的優勢和權衡。正如我們將在下一節關於編排中看到的，您可能想要考慮為工作流程中的不同任務使用各種模型。

並非每個任務都需要最聰明的模型——簡單的檢索或意圖分類任務可能由更小、更快的模型處理，而更難的任務（如決定是否批准退款）可能受益於更有能力的模型。

一個有效的方法是使用最有能力的模型為每個任務建構您的代理原型，以建立效能基準。從那裡，嘗試替換為較小的模型，看看它們是否仍能達到可接受的結果。這樣，您不會過早限制代理的能力，並且可以診斷較小模型成功或失敗的地方。

總之，選擇模型的原則很簡單：

### 01 設置評估以建立效能基準
### 02 專注於使用可用的最佳模型滿足您的準確性目標
### 03 在可能的情況下，通過用較小的模型替換較大的模型來優化成本和延遲

您可以在這裡找到選擇 OpenAI 模型的綜合指南。

## 定義工具

工具通過使用底層應用程式或系統的API來擴展您代理的能力。對於沒有API的舊系統，代理可以依靠計算機使用模型直接通過網頁和應用程式使用者介面與這些應用程式和系統互動，就像人類一樣。

每個工具都應該有標準化的定義，在工具和代理之間啟用靈活的多對多關係。文件完善、經過徹底測試和可重複使用的工具提高了可發現性、簡化了版本管理並防止了冗餘定義。

廣義上講，代理需要三種類型的工具：

| 類型 | 描述 | 範例 |
|------|------|------|
| 資料 | 使代理能夠檢索執行工作流程所需的上下文和資訊 | 查詢交易資料庫或CRM等系統、讀取PDF文件或搜索網路 |
| 行動 | 使代理能夠與系統互動，採取諸如將新資訊添加到資料庫、更新記錄或發送訊息等行動 | 發送電子郵件和簡訊、更新CRM記錄、將客戶服務工單移交給人工 |
| 編排 | 代理本身可以作為其他代理的工具——請參閱編排部分中的管理者模式 | 退款代理、研究代理、寫作代理 |

例如，以下是使用代理SDK時如何為上面定義的代理配備一系列工具：

```python
from agents import Agent, WebSearchTool, function_tool

@function_tool
def save_results(output):
    db.insert({"output": output,"timestamp": datetime.time()})
    return "File saved"

search_agent = Agent(
    name="Search agent",
    instructions="Help the user search the internet and save results if asked.",
    tools=[WebSearchTool(), save_results],
)
```

隨著所需工具數量的增加，考慮將任務分散到多個代理（參見編排）。

## 配置指令

高品質的指令對於任何大型語言模型驅動的應用程式都至關重要，但對代理尤其重要。清晰的指令減少了歧義並改善了代理決策制定，導致更順暢的工作流程執行和更少的錯誤。

### 代理指令的最佳實務

**使用現有文件**  
在創建例程時，使用現有的操作程序、支援腳本或政策文件來創建大型語言模型友好的例程。例如，在客戶服務中，例程可以大致映射到您知識庫中的各個文章。

**提示代理分解任務**  
從密集資源提供更小、更清晰的步驟有助於最小化歧義並幫助模型更好地遵循指令。

**定義清晰的行動**  
確保您例程中的每個步驟都對應於特定的行動或輸出。例如，一個步驟可能指示代理詢問使用者的訂單號碼或呼叫API來檢索帳戶詳細資訊。明確說明行動（甚至是面向使用者的訊息的措辭）為解釋錯誤留下了更少的空間。

**捕獲邊緣情況**  
現實世界的互動經常創建決策點，例如當使用者提供不完整資訊或提出意外問題時如何進行。健全的例程預期常見變化並包括如何處理它們的指令，具有條件步驟或分支，例如如果缺少所需資訊片段的替代步驟。

您可以使用進階模型，如 o1 或 o3-mini，從現有文件自動生成指令。以下是說明此方法的範例提示：

```
"You are an expert in writing instructions for an LLM agent. Convert the following help center document into a clear set of instructions, written in a numbered list. The document will be a policy followed by an LLM. Ensure that there is no ambiguity, and that the instructions are written as directions for an agent. The help center document to convert is the following {{help_center_doc}}"
```
