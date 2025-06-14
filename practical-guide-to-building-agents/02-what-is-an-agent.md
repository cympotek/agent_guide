# 什麼是代理？

What is an
代理?
While conventional software enables 使用者 to streamline and automate 工作流程, 代理 are able
to perform the same 工作流程 on the 使用者’ behalf with a high degree of independence.
代理是代表您獨立完成任務的系統。
A 工作流程 is a sequence of steps that must be executed to meet the 使用者’s goal, whether that's
resolving a 客戶 service issue, booking a restaurant reservation, committing a code change,
or generating a report.

應用程式 that integrate LLMs but don’t use them to control 工作流程 execution—think simple
chatbots, single-turn LLMs, or sentiment classifiers—are not 代理.

More concretely, an 代理 possesses core characteristics that allow it to act reliably and
consistently on behalf of a 使用者:
### 01 It leverages an LLM to manage 工作流程 execution and make 決策. It recognizes
when a 工作流程 is complete and can proactively correct its actions if needed. In case
of failure, it can halt execution and transfer control back to the 使用者.
### 02 It has access to various 工具 to interact with external 系統—both to gather context
and to take actions—and dynamically selects the appropriate 工具 depending on the
工作流程’s current state, always operating within clearly defined 護欄機制.
4 A practical guide to building 代理

