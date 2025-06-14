# 編排

編排
With the foundational components in place, you can consider 編排 patterns to enable
your 代理 to execute 工作流程 effectively.

While it’s tempting to immediately build a fully autonomous 代理 with complex architecture,
客戶 typically achieve greater success with an incremental approach.
In general, 編排 patterns fall into two categories:
### 01 單一代理系統, where a single 模型 equipped with appropriate 工具 and
指令 executes 工作流程 in a loop
### 02 多代理系統, where 工作流程 execution is distributed across multiple
coordinated 代理
Let’s explore each pattern in detail.
13 A practical guide to building 代理

單一代理系統
A single 代理 can handle many tasks by incrementally adding 工具, keeping 複雜性
manageable and simplifying 評估 and 維護. Each new 工具 expands its capabilities
without prematurely forcing you to orchestrate multiple 代理.
Input 代理 Output
指令
工具
護欄機制
## Hooks
Every 編排 approach needs the concept of a ‘run’, typically implemented as a loop that
lets 代理 operate until an exit condition is reached. Common exit conditions include 工具 calls,
a certain structured output, errors, or reaching a maximum number of turns.
14 A practical guide to building 代理

例如, in the 代理 SDK, 代理 are started using the Runner.run() method, which loops
over the LLM until either:
### 01 A final-output 工具 is invoked, defined by a specific output type
### 02 The 模型 returns a response without any 工具 calls (e.g., a direct 使用者 message)
Example usage:
```python
代理.run(代理, [UserMessage("What's the capital of the USA?")])
```
This concept of a while loop is central to the functioning of an 代理. In multi-代理 系統, as
you’ll see next, you can have a sequence of 工具 calls and handoffs between 代理 but allow the
模型 to run multiple steps until an exit condition is met.

An effective strategy for managing 複雜性 without switching to a multi-代理 框架 is to
use prompt templates. Rather than maintaining numerous individual prompts for distinct use
cases, use a single flexible base prompt that accepts policy variables. This template approach
adapts easily to various contexts, significantly simplifying 維護 and 評估. As new use
cases arise, you can update variables rather than rewriting entire 工作流程.
```unset
""" You are a call center 代理. You are interacting with
{{user_first_name}} who has been a member for {{user_tenure}}. The 使用者's
most common complains are about {{user_complaint_categories}}. Greet the
```
使用者, thank them for being a loyal 客戶, and answer any questions the
使用者 may have!
15 A practical guide to building 代理

When to consider creating multiple 代理
Our general recommendation is to maximize a single 代理’s capabilities first. More 代理 can
provide intuitive separation of concepts, but can introduce additional 複雜性 and overhead,
so often a single 代理 with 工具 is sufficient.
For many complex 工作流程, splitting up prompts and 工具 across multiple 代理 allows for
improved 效能 and 可擴展性. When your 代理 fail to follow complicated 指令
or consistently select incorrect 工具, you may need to further divide your 系統 and introduce
more distinct 代理.

Practical guidelines for splitting 代理 include:
Complex logic When prompts contain many conditional statements
(multiple if-then-else branches), and prompt templates get
difficult to scale, consider dividing each logical segment across
separate 代理.
工具 overload The issue isn’t solely the number of 工具, but their similarity
or overlap. Some implementations successfully manage
more than 15 well-defined, distinct 工具 while others struggle
with fewer than 10 overlapping 工具. Use multiple 代理
if improving 工具 clarity by providing descriptive names,
clear parameters, and detailed descriptions doesn’t
improve 效能.
16 A practical guide to building 代理

多代理系統
While multi-代理 系統 can be designed in numerous ways for specific 工作流程 and
requirements, our experience with 客戶 highlights two broadly applicable categories:
Manager (代理 as 工具) A central “manager” 代理 coordinates multiple specialized
代理 via 工具 calls, each handling a specific task or domain.
Decentralized (代理 handing Multiple 代理 operate as peers, handing off tasks to one
off to 代理) another based on their specializations.
多代理系統 can be modeled as graphs, with 代理 represented as nodes. In the manager
pattern, edges represent 工具 calls whereas in the decentralized pattern, edges represent handoffs
that transfer execution between 代理.

Regardless of the 編排 pattern, the same principles apply: keep components flexible,
composable, and driven by clear, well-structured prompts.
17 A practical guide to building 代理

管理者模式
The manager pattern empowers a central LLM—the “manager”—to orchestrate a network of
specialized 代理 seamlessly through 工具 calls. Instead of losing context or control, the manager
intelligently delegates tasks to the right 代理 at the right time, effortlessly synthesizing the results
into a cohesive interaction. This ensures a smooth, unified 使用者 experience, with specialized
capabilities always available on-demand.

This pattern is ideal for 工作流程 where you only want one 代理 to control 工作流程 execution
and have access to the 使用者.
Translate ‘hello’ to Task Spanish 代理
Spanish, French and
Italian for me!
Manager Task French 代理
...
Task Italian 代理
18 A practical guide to building 代理

例如, here’s how you could implement this pattern in the 代理 SDK:
```python
from 代理 import 代理, Runner



manager_agent = 代理(

name="manager_agent",

指令=(

```
"You are a translation 代理. You use the 工具 given to you to
7
translate."

8
"If asked for multiple translations, you call the relevant 工具."

9
),

10
工具=[

11
spanish_agent.as_tool(

12
tool_name="translate_to_spanish",

13
tool_description="Translate the 使用者's message to Spanish",

14
),

15
french_agent.as_tool(

16
tool_name="translate_to_french",

17
tool_description="Translate the 使用者's message to French",

18
),

19
italian_agent.as_tool(

20
tool_name="translate_to_italian",

21
tool_description="Translate the 使用者's message to Italian",

22
),

23 ],

19 A practical guide to building 代理

24
)


25

26
async def main():

27
msg = input("Translate 'hello' to Spanish, French and Italian for me!")


28

29
orchestrator_output = await Runner.run(

30
manager_agent,msg)


32

32
for message in orchestrator_output.new_messages:

33 print(f" - Translation step: {message.content}")
Declarative vs non-declarative graphs
Some 框架 are declarative, requiring 開發者 to explicitly define every branch, loop,
and conditional in the 工作流程 upfront through graphs consisting of nodes (代理) and
edges (deterministic or dynamic handoffs). While beneficial for visual clarity, this approach
can quickly become cumbersome and challenging as 工作流程 grow more dynamic and
complex, often necessitating the learning of specialized domain-specific languages.

In contrast, the 代理 SDK adopts a more flexible, code-first approach. 開發者 can
directly express 工作流程 logic using familiar programming constructs without needing to
pre-define the entire graph upfront, enabling more dynamic and adaptable 代理 編排.
20 A practical guide to building 代理

去中心化模式
In a decentralized pattern, 代理 can ‘handoff’ 工作流程 execution to one another. Handoffs are a
one way transfer that allow an 代理 to delegate to another 代理. In the 代理 SDK, a handoff is
a type of 工具, or function. If an 代理 calls a handoff function, we immediately start execution on
that new 代理 that was handed off to while also transferring the latest conversation state.
This pattern involves using many 代理 on equal footing, where one 代理 can directly hand
off control of the 工作流程 to another 代理. This is optimal when you don’t need a single 代理
maintaining central control or synthesis—instead allowing each 代理 to take over execution and
interact with the 使用者 as needed.
Issues and Repairs
Where is my order? Triage Sales
Orders
On its way!
21 A practical guide to building 代理

例如, here’s how you’d implement the decentralized pattern using the 代理 SDK for
a 客戶 service 工作流程 that handles both sales and support:
```python
from 代理 import 代理, Runner

technical_support_agent = 代理(

name="Technical Support 代理",

指令=(

```
"You provide expert assistance with resolving technical issues,
7
系統 outages, or 產品 troubleshooting."

8
),

9
工具=[search_knowledge_base]

10
)


11

12
sales_assistant_agent = 代理(

13
name="Sales Assistant 代理",

14
指令=(

15
"You help 企業 clients browse the 產品 catalog, recommend
16
suitable 解決方案, and facilitate purchase transactions."

17
),

18
工具=[initiate_purchase_order]

19
)


20

21
order_management_agent = 代理(

22
name="Order Management 代理",

23
指令=(

24
"You assist clients with inquiries regarding order tracking,
25

delivery schedules, and processing returns or refunds."

22 A practical guide to building 代理

26
),

27
工具=[track_order_status, initiate_refund_process]

28
)


29

30
triage_agent = 代理(

31
name=Triage 代理",

32
指令="You act as the first point of contact, assessing 客戶
33
queries and directing them promptly to the correct specialized 代理.",

34
handoffs=[technical_support_agent, sales_assistant_agent,
35
order_management_agent],

36
)


37

38
await Runner.run(

39
triage_agent,

40
input("Could you please provide an update on the delivery timeline for
41
our recent purchase?")

42 )

In the above example, the initial 使用者 message is sent to triage_agent. Recognizing that
the input concerns a recent purchase, the triage_agent would invoke a handoff to the
order_management_agent, transferring control to it.

This pattern is especially effective for scenarios like conversation triage, or whenever you prefer
specialized 代理 to fully take over certain tasks without the original 代理 needing to remain
involved. Optionally, you can equip the second 代理 with a handoff back to the original 代理,
allowing it to transfer control again if necessary.
23 A practical guide to building 代理

