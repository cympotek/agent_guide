# 代理設計基礎

代理 design
foundations
在最基本的形式中，代理由三個核心組件組成：
### 01 模型 The LLM powering the 代理’s reasoning and 決策-making
### 02 工具 代理可用來採取行動的外部函數或API
### 03 指令 Explicit guidelines and 護欄機制 defining how the
代理 behaves
Here’s what this looks like in code when using OpenAI’s 代理 SDK. You can also implement the
same concepts using your preferred library or building directly from scratch.
```python
weather_agent = 代理(

name="Weather 代理",

指令="You are a helpful 代理 who can talk to 使用者 about the
```
weather.",

5
工具=[get_weather],

6 )
7 A practical guide to building 代理

Selecting your 模型
Different 模型 have different strengths and tradeoffs related to task 複雜性, latency, and
cost. As we’ll see in the next section on 編排, you might want to consider using a variety
of 模型 for different tasks in the 工作流程.

Not every task requires the smartest 模型—a simple retrieval or intent classification task may be
handled by a smaller, faster 模型, while harder tasks like deciding whether to approve a refund
may benefit from a more capable 模型.

An approach that works well is to build your 代理 prototype with the most capable 模型 for
every task to establish a 效能 baseline. From there, try swapping in smaller 模型 to see
if they still achieve acceptable results. This way, you don’t prematurely limit the 代理’s abilities,
and you can diagnose where smaller 模型 succeed or fail.
總結, the principles for choosing a 模型 are simple:
### 01 Set up evals to establish a 效能 baseline
### 02 Focus on meeting your accuracy target with the best 模型 available
### 03 Optimize for cost and latency by replacing larger 模型 with smaller ones
where possible
You can find a comprehensive guide to selecting OpenAI 模型 here.
8 A practical guide to building 代理

Defining 工具
工具 extend your 代理’s capabilities by using API from underlying 應用程式 or 系統. For
legacy 系統 without API, 代理 can rely on computer-use 模型 to interact directly with
those 應用程式 and 系統 through web and 應用程式 UIs—just as a human would.

Each 工具 should have a standardized definition, enabling flexible, many-to-many relationships
between 工具 and 代理. Well-documented, thoroughly tested, and reusable 工具 improve
discoverability, simplify version management, and prevent redundant definitions.

Broadly speaking, 代理 need three types of 工具:
Type Description Examples
Data Enable 代理 to retrieve context and Query transaction 資料庫 or
information necessary for executing 系統 like CRMs, read PDF
the 工作流程. documents, or search the web.
Action Enable 代理 to interact with Send emails and texts, update a CRM
系統 to take actions such as record, hand-off a 客戶 service
adding new information to ticket to a human.
資料庫, updating records, or
sending messages.
編排 代理 themselves can serve as 工具 Refund 代理, Research 代理,
for other 代理—see the Manager Writing 代理.
Pattern in the 編排 section.
9 A practical guide to building 代理

例如, here’s how you would equip the 代理 defined above with a series of 工具 when using
the 代理 SDK:
```python
from 代理 import 代理, WebSearchTool, function_tool

```
@function_tool

3
def save_results(output):

4
db.insert({"output": output,"timestamp": datetime.time()})

5
return "File saved"


6

7
search_agent = 代理(

8
name="Search 代理",

8
指令="Help the 使用者 search the internet and save results if
10
asked.",

11
工具=[WebSearchTool(),save_results],

12 )
As the number of required 工具 increases, consider splitting tasks across multiple 代理
(see 編排).
10 A practical guide to building 代理

Configuring 指令
High-quality 指令 are essential for any LLM-powered app, but especially critical for 代理.
Clear 指令 reduce ambiguity and improve 代理 決策-making, resulting in smoother
工作流程 execution and fewer errors.
最佳實務 for 代理 指令
Use existing documents When creating routines, use existing operating procedures,
support scripts, or policy documents to create LLM-friendly
routines. In 客戶 service 例如, routines can roughly
map to individual articles in your knowledge base.
Prompt 代理 to break Providing smaller, clearer steps from dense resources
down tasks helps minimize ambiguity and helps the 模型 better
follow 指令.
Define clear actions Make sure every step in your routine corresponds to a specific
action or output. 例如, a step might instruct the 代理
to ask the 使用者 for their order number or to call an API to
retrieve account details. Being explicit about the action (and
even the wording of a 使用者-facing message) leaves less room
for errors in interpretation.
Capture edge cases Real-world interactions often create 決策 points such as
how to proceed when a 使用者 provides incomplete information
or asks an unexpected question. A robust routine anticipates
common variations and includes 指令 on how to handle
them with conditional steps or branches such as an alternative
step if a required piece of info is missing.
11 A practical guide to building 代理

You can use advanced 模型, like o1 or o3-mini, to automatically generate 指令 from
existing documents. Here’s a sample prompt illustrating this approach:
```unset
“You are an expert in writing 指令 for an LLM 代理. Convert the
```
following help center document into a clear set of 指令, written in
a numbered list. The document will be a policy followed by an LLM. Ensure
that there is no ambiguity, and that the 指令 are written as
directions for an 代理. The help center document to convert is the
following {{help_center_doc}}”
12 A practical guide to building 代理

