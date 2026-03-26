- Agent Do Something =  Model + Harness
- Harness with
	- System Prompt
	- Tools, Skills, MCPs + Description
	- Bundled Infrastructure (filesystem, sandbox, browser)
	- Orchestration Logic (subagent spawning, handoffs, model routing)
	- Hooks/Middleware for deterministic execution (compaction, continuation, lint checks)
- Agent Harness 的产品形态
	- CLI - for human use，build new agents
	- SDK - for system integration，using in agents

- 哪些是 Agent 做不了，所以需要 Harness 做的事情
	- 和真实数据交互 —— Harness 需要包含文件管理能力和 Git 管理能力；
	- 写 & 执行代码 —— 要有 Bash 能力（最泛） + Code Execution 能力（maybe 包在 Bash 里）
	- 安全执行  —— 沙箱环境
	- 知识记忆 —— Memory Files & Web Search & MCPs
	- 在长期执行情况下保证效果 —— 上下文压缩 、Tools 的选择性 Loading 、Skills
	- 完整的长时间运行 —— Ralph Loops + Planning + vertificaition
- 必要的功能
	- Bash
	- Code Execution
	- Sandbox ：保证让 Agent 可以放心的操作
	- Observe：可观测性确保推动下一步优化
- 长期演进方向
	- Multiple Agent
	- Agent to Agent
	- Multiplexing Agent
- 目标：
	- 让模型可以可靠、高效的在 AgentHarness 上运行
	- Model is CPU；
	- Context Window is RAM；
	- Agent Harness is Operating System；
	- Agent is Application
- 对比 Agent Framework & Agent Harness

|      项目       | Agent Framework           | Agent Harness   |
| :-------------: | ------------------------- | --------------- |
|  Prompt Preset  | NO，just do it your self  | YES             |
|      Tool       | Just way to archive tools | predefine tools |
| Life Cycle Hook | NO，just do it your self  | YES             |
|    Plan Mode    | NO，just do it your self  | YES             |
|   FileSystem    | NO，just do it your self  | YES             |

## 一个好的 Agent Harness 应该包含什么？

**运行循环与工具编排**  
需要有稳定的 agent loop：调用工具、把结果送回模型、继续推理，直到任务结束；还要有清晰的 tool / handoff 机制，让 agent 能和外部世界交互，而不是只停留在文本里。
**上下文工程与记忆系统**  
这不只是 system prompt，而是“把正确的信息和工具，以正确格式，在正确时机提供给 agent”。
**规划、分解与委派**  
Harness 里要有 planner / todo / subagent / handoff 之类的机制。很多 agent 失败不是不会做，而是一次想做太多、或者中间过程把主上下文撑爆。LangChain 明确把 planning、task delegation 放进 harness，而 subagents 的直接价值之一就是隔离 context bloat。
**状态持久化与可恢复执行**  
生产级 harness 必须能 pause、resume，并把“做到哪一步了、下一步做什么、约束是什么”沉淀为结构化 artifact。Anthropic 的 memory pattern 里，initializer session、progress log、feature checklist 都是典型做法；OpenAI 的 HITL 也强调用 RunState 去序列化并恢复执行。
**安全边界与审批机制**  
至少要有 sandbox、敏感工具审批、输入/输出 guardrails。LangChain 的 sandbox 文档明确强调 autonomous agent 需要隔离执行环境；OpenAI 和 LangChain 都把 human-in-the-loop / approvals 当成一等能力。
**验证与评测层**  
不能只看“最后答得像不像”，而要验证“这一步是不是做成了、改动有没有回归、系统升级后有没有退化”。Anthropic 在 long-running harness 和 eval 文章里都强调：真正该评测的是 harness 与 model 的联合作用，而不是单看模型。
**观测与调试层**  
需要 traces、events、streaming。OpenAI Agents SDK 的 tracing 会记录 LLM generations、tool calls、handoffs、guardrails；streaming 则让你看到 agent 运行中的进度与部分输出，而不是只在最后拿一个结果。
**高级增强：并行与多 agent 协同**  
这不是每个场景都必需，但任务一长就会变得很重要。Anthropic 最近关于 long-running app 和 agent teams 的文章都在强调：planner / generator / evaluator 的分工、并行 agent、结构化交接，可以明显扩展复杂任务下的稳定性和覆盖面。

没有记忆、没有审批、没有验证、没有 tracing 的 agent，通常还只是一个会调工具的 demo；真正的 harness，要让 agent **长期运行、可控、可恢复、可评估、可迭代**。

## Reference
- https://blog.langchain.com/the-anatomy-of-an-agent-harness/
- https://ghuntley.com/loop/?ref=blog.langchain.com
- https://www.philschmid.de/agent-harness-2026
- https://www.jasonwei.net/blog/asymmetry-of-verification-and-verifiers-law
- https://www.salesforce.com/agentforce/ai-agents/agent-harness/
- https://aakashgupta.medium.com/2025-was-agents-2026-is-agent-harnesses-heres-why-that-changes-everything-073e9877655e
- https://paddo.dev/blog/agent-harnesses-from-diy-to-product/
- https://martinfowler.com/articles/exploring-gen-ai/harness-engineering.html
- https://dev.to/apssouza22/building-a-production-ready-ai-agent-harness-2570
- https://ezz.sh/posts/agent_vs_harness
- https://conais.com/agent-harness/
- https://medium.com/@bijit211987/agent-harness-b1f6d5a7a1d1
- https://blog.langchain.com/improving-deep-agents-with-harness-engineering/
- https://blog.langchain.com/improving-deep-agents-with-harness-engineering/
- https://mostlycopyandpaste.com/articles/2026/02/ai-agent-harnesses/
- https://codenote.net/en/posts/harness-engineering-ai-agent-era/
- https://openai.com/zh-Hans-CN/index/harness-engineering/
- https://mitchellh.com/writing/my-ai-adoption-journey