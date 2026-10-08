# 技能与代理开发

Claude Code / Codex 等代理体系的 Skill、Agent、Command、Hook、Plugin 与 MCP 服务开发技能。

共 **2131** 个技能单元，来源 **192** 个仓库。

| 技能 | 说明 | 来源仓库 |
|------|------|---------|
| [_references](./_references) | 共享规范知识库。包含数学建模竞赛的写作规范、题型防错速查、图表规范等参考内容。其他 skills 在执行过程中按需读取，无需单独触发。 | `MathModelAgent` |
| [_template](./_template) | One line on what this playbook does, then a "Use when ..." clause so the agent knows when to load it (e.g. "Us | `agent` |
| [1start-mathmodel](./1start-mathmodel) | 数学建模竞赛工作流入口。用于启动完整建模流程：询问用户偏好，生成 plan.md 和 todo.md，并按阶段调用赛题分析、建模、代码与图表、流程图、论文撰写、验证验收等 skills。 | `MathModelAgent` |
| [2analysis-modeling](./2analysis-modeling) | 数学建模赛题分析与建模设计合并阶段。用于读取题面和附件，完成子问题拆解、数据理解、假设预检、变量定义、模型公式、目标函数、约束条件、求解策略和可交给代码实现的建模报告。 | `MathModelAgent` |
| [3coding-visual](./3coding-visual) | 数学建模编程实现与数据图表生成阶段。根据 ANALYSIS_MODELING_REPORT.md 编写可复现代码、运行求解、验证约束、输出 RESULTS_REPORT.md 并生成论文可用的数据驱动图表 PDF。 | `MathModelAgent` |
| [4drawio](./4drawio) | 数学建模非数据型图示绘制阶段。根据 ANALYSIS_MODELING_REPORT.md、RESULTS_REPORT.md 和已有 figures/ 生成技术路线图、子问题求解流程图、模型结构图、数据处理流程图等 D | `MathModelAgent` |
| [5writing](./5writing) | 数学建模竞赛论文撰写阶段，支持 Typst 和 LaTeX 双引擎。根据 ANALYSIS_MODELING_REPORT.md、RESULTS_REPORT.md 和 figures/*.pdf 选择比赛模板、排版引擎 | `MathModelAgent` |
| [6verity](./6verity) | 数学建模竞赛最终验证和验收阶段，支持 Typst 和 LaTeX 双引擎。用于论文写完后检查章节数量、标题顺序、图表引用、数值一致性、占位符、内部文件泄露、参考文献、代码可复现性、编译和提交就绪状态。 | `MathModelAgent` |
| [a2ui-renderer](./a2ui-renderer) | > | `copilotkit` |
| [activecampaign-automation](./activecampaign-automation) | Automate ActiveCampaign tasks via Rube MCP (Composio): manage contacts, tags, list subscriptions, automation e | `agentic-awesome-skills` |
| [activecampaign-automation-antigravity-awesome-skills-main](./activecampaign-automation-antigravity-awesome-skills-main) | Automate ActiveCampaign tasks via Rube MCP (Composio): manage contacts, tags, list subscriptions, automation e | `antigravity-awesome-skills-main` |
| [adaptyv](./adaptyv) | How to use the Adaptyv Bio Foundry API and Python SDK for protein experiment design, submission, and results r | `scientific-agent-skills` |
| [adk-sample-creator](./adk-sample-creator) | >- | `google__adk-python` |
| [adopt](./adopt) | Brownfield onboarding — audits existing project artifacts for template format compliance (not just existence), | `Claude-Code-Game-Studios-main` |
| [advanced-evaluation](./advanced-evaluation) | This skill should be used when the user asks to "implement LLM-as-judge", "compare model outputs", "create eva | `Agent-Skills-for-Context-Engineering-main` |
| [advanced-evaluation-agentic-awesome-skills](./advanced-evaluation-agentic-awesome-skills) | This skill should be used when the user asks to "implement LLM-as-judge", "compare model outputs", "create eva | `agentic-awesome-skills` |
| [advanced-evaluation-antigravity-awesome-skills-main](./advanced-evaluation-antigravity-awesome-skills-main) | This skill should be used when the user asks to "implement LLM-as-judge", "compare model outputs", "create eva | `antigravity-awesome-skills-main` |
| [advogado-criminal](./advogado-criminal) | Advogado criminalista especializado em Maria da Penha, violencia domestica, feminicidio, direito penal brasile | `agentic-awesome-skills` |
| [advogado-criminal-antigravity-awesome-skills-main](./advogado-criminal-antigravity-awesome-skills-main) | Advogado criminalista especializado em Maria da Penha, violencia domestica, feminicidio, direito penal brasile | `antigravity-awesome-skills-main` |
| [advogado-especialista](./advogado-especialista) | Advogado especialista em todas as areas do Direito brasileiro: familia, criminal, trabalhista, tributario, con | `agentic-awesome-skills` |
| [advogado-especialista-antigravity-awesome-skills-main](./advogado-especialista-antigravity-awesome-skills-main) | Advogado especialista em todas as areas do Direito brasileiro: familia, criminal, trabalhista, tributario, con | `antigravity-awesome-skills-main` |
| [aeon](./aeon) | This skill should be used for time series machine learning tasks including classification, regression, cluster | `scientific-agent-skills` |
| [agent-agent](./agent-agent) | Agent skill for agent - invoke with $agent-agent | `ruflo` |
| [agent-agent-ruflo-main](./agent-agent-ruflo-main) | Agent skill for agent - invoke with $agent-agent | `ruflo-main` |
| [agent-agentic-payments](./agent-agentic-payments) | Agent skill for agentic-payments - invoke with $agent-agentic-payments | `ruflo` |
| [agent-agentic-payments-ruflo-main](./agent-agentic-payments-ruflo-main) | Agent skill for agentic-payments - invoke with $agent-agentic-payments | `ruflo-main` |
| [agent-analyze-code-quality](./agent-analyze-code-quality) | Agent skill for analyze-code-quality - invoke with $agent-analyze-code-quality | `ruflo` |
| [agent-analyze-code-quality-ruflo-main](./agent-analyze-code-quality-ruflo-main) | Agent skill for analyze-code-quality - invoke with $agent-analyze-code-quality | `ruflo-main` |
| [agent-app-store](./agent-app-store) | Agent skill for app-store - invoke with $agent-app-store | `ruflo` |
| [agent-app-store-ruflo-main](./agent-app-store-ruflo-main) | Agent skill for app-store - invoke with $agent-app-store | `ruflo-main` |
| [agent-arch-system-design](./agent-arch-system-design) | Agent skill for arch-system-design - invoke with $agent-arch-system-design | `ruflo` |
| [agent-arch-system-design-ruflo-main](./agent-arch-system-design-ruflo-main) | Agent skill for arch-system-design - invoke with $agent-arch-system-design | `ruflo-main` |
| [agent-architecture](./agent-architecture) | Agent skill for architecture - invoke with $agent-architecture | `ruflo` |
| [agent-architecture-audit](./agent-architecture-audit) | Full-stack diagnostic for agent and LLM applications. Audits the 12-layer agent stack for wrapper regression,  | `ECC` |
| [agent-architecture-ruflo-main](./agent-architecture-ruflo-main) | Agent skill for architecture - invoke with $agent-architecture | `ruflo-main` |
| [agent-authentication](./agent-authentication) | Agent skill for authentication - invoke with $agent-authentication | `ruflo` |
| [agent-authentication-ruflo-main](./agent-authentication-ruflo-main) | Agent skill for authentication - invoke with $agent-authentication | `ruflo-main` |
| [agent-automation-smart-agent](./agent-automation-smart-agent) | Agent skill for automation-smart-agent - invoke with $agent-automation-smart-agent | `ruflo` |
| [agent-automation-smart-agent-ruflo-main](./agent-automation-smart-agent-ruflo-main) | Agent skill for automation-smart-agent - invoke with $agent-automation-smart-agent | `ruflo-main` |
| [agent-base-template-generator](./agent-base-template-generator) | Agent skill for base-template-generator - invoke with $agent-base-template-generator | `ruflo` |
| [agent-base-template-generator-ruflo-main](./agent-base-template-generator-ruflo-main) | Agent skill for base-template-generator - invoke with $agent-base-template-generator | `ruflo-main` |
| [agent-benchmark-suite](./agent-benchmark-suite) | Agent skill for benchmark-suite - invoke with $agent-benchmark-suite | `ruflo` |
| [agent-benchmark-suite-ruflo-main](./agent-benchmark-suite-ruflo-main) | Agent skill for benchmark-suite - invoke with $agent-benchmark-suite | `ruflo-main` |
| [agent-browser](./agent-browser) | Browser automation CLI for AI agents. Use when the user needs to interact with websites, including navigating  | `claude-code-best-practice-main` |
| [agent-builder](./agent-builder) | \| | `shareAI-lab__learn-claude-code` |
| [agent-byzantine-coordinator](./agent-byzantine-coordinator) | Agent skill for byzantine-coordinator - invoke with $agent-byzantine-coordinator | `ruflo` |
| [agent-byzantine-coordinator-ruflo-main](./agent-byzantine-coordinator-ruflo-main) | Agent skill for byzantine-coordinator - invoke with $agent-byzantine-coordinator | `ruflo-main` |
| [agent-challenges](./agent-challenges) | Agent skill for challenges - invoke with $agent-challenges | `ruflo` |
| [agent-challenges-ruflo-main](./agent-challenges-ruflo-main) | Agent skill for challenges - invoke with $agent-challenges | `ruflo-main` |
| [agent-code-goal-planner](./agent-code-goal-planner) | Agent skill for code-goal-planner - invoke with $agent-code-goal-planner | `ruflo` |
| [agent-code-goal-planner-ruflo-main](./agent-code-goal-planner-ruflo-main) | Agent skill for code-goal-planner - invoke with $agent-code-goal-planner | `ruflo-main` |
| [agent-coder](./agent-coder) | Agent skill for coder - invoke with $agent-coder | `ruflo` |
| [agent-coder-ruflo-main](./agent-coder-ruflo-main) | Agent skill for coder - invoke with $agent-coder | `ruflo-main` |
| [agent-collective-intelligence-coordinator](./agent-collective-intelligence-coordinator) | Agent skill for collective-intelligence-coordinator - invoke with $agent-collective-intelligence-coordinator | `ruflo` |
| [agent-collective-intelligence-coordinator-ruflo-main](./agent-collective-intelligence-coordinator-ruflo-main) | Agent skill for collective-intelligence-coordinator - invoke with $agent-collective-intelligence-coordinator | `ruflo-main` |
| [agent-consensus-coordinator](./agent-consensus-coordinator) | Agent skill for consensus-coordinator - invoke with $agent-consensus-coordinator | `ruflo` |
| [agent-consensus-coordinator-ruflo-main](./agent-consensus-coordinator-ruflo-main) | Agent skill for consensus-coordinator - invoke with $agent-consensus-coordinator | `ruflo-main` |
| [agent-context-isolation](./agent-context-isolation) | Agent Context Isolation | `Continuous-Claude-v3-main` |
| [agent-coordinator-swarm-init](./agent-coordinator-swarm-init) | Agent skill for coordinator-swarm-init - invoke with $agent-coordinator-swarm-init | `ruflo` |
| [agent-coordinator-swarm-init-ruflo-main](./agent-coordinator-swarm-init-ruflo-main) | Agent skill for coordinator-swarm-init - invoke with $agent-coordinator-swarm-init | `ruflo-main` |
| [agent-crdt-synchronizer](./agent-crdt-synchronizer) | Agent skill for crdt-synchronizer - invoke with $agent-crdt-synchronizer | `ruflo` |
| [agent-crdt-synchronizer-ruflo-main](./agent-crdt-synchronizer-ruflo-main) | Agent skill for crdt-synchronizer - invoke with $agent-crdt-synchronizer | `ruflo-main` |
| [agent-data-ml-model](./agent-data-ml-model) | Agent skill for data-ml-model - invoke with $agent-data-ml-model | `ruflo` |
| [agent-data-ml-model-ruflo-main](./agent-data-ml-model-ruflo-main) | Agent skill for data-ml-model - invoke with $agent-data-ml-model | `ruflo-main` |
| [agent-dev-backend-api](./agent-dev-backend-api) | Agent skill for dev-backend-api - invoke with $agent-dev-backend-api | `ruflo` |
| [agent-dev-backend-api-ruflo-main](./agent-dev-backend-api-ruflo-main) | Agent skill for dev-backend-api - invoke with $agent-dev-backend-api | `ruflo-main` |
| [agent-development](./agent-development) | This skill should be used when the user asks to "create an agent", "add an agent", "write a subagent", "agent  | `claude-plugins-official-main` |
| [agent-docs-api-openapi](./agent-docs-api-openapi) | Agent skill for docs-api-openapi - invoke with $agent-docs-api-openapi | `ruflo` |
| [agent-docs-api-openapi-ruflo-main](./agent-docs-api-openapi-ruflo-main) | Agent skill for docs-api-openapi - invoke with $agent-docs-api-openapi | `ruflo-main` |
| [agent-eval](./agent-eval) | Head-to-head comparison of coding agents (Claude Code, Aider, Codex, etc.) on custom tasks with pass rate, cos | `ECC` |
| [agent-eval-everything-claude-code-main](./agent-eval-everything-claude-code-main) | Head-to-head comparison of coding agents (Claude Code, Aider, Codex, etc.) on custom tasks with pass rate, cos | `everything-claude-code-main` |
| [agent-evaluation](./agent-evaluation) | Testing and benchmarking LLM agents including behavioral testing, | `antigravity-awesome-skills-main` |
| [agent-evaluation-penguin-harness](./agent-evaluation-penguin-harness) | Run one specified Test Agent on one specified Benchmark Case exactly once, privately score that execution, and | `penguin-harness` |
| [agent-framework-azure-ai-py](./agent-framework-azure-ai-py) | Build persistent agents on Azure AI Foundry using the Microsoft Agent Framework Python SDK. | `agentic-awesome-skills` |
| [agent-framework-azure-ai-py-antigravity-awesome-skills-main](./agent-framework-azure-ai-py-antigravity-awesome-skills-main) | Build persistent agents on Azure AI Foundry using the Microsoft Agent Framework Python SDK. | `antigravity-awesome-skills-main` |
| [agent-github-modes](./agent-github-modes) | Agent skill for github-modes - invoke with $agent-github-modes | `ruflo` |
| [agent-github-modes-ruflo-main](./agent-github-modes-ruflo-main) | Agent skill for github-modes - invoke with $agent-github-modes | `ruflo-main` |
| [agent-github-pr-manager](./agent-github-pr-manager) | Agent skill for github-pr-manager - invoke with $agent-github-pr-manager | `ruflo` |
| [agent-github-pr-manager-ruflo-main](./agent-github-pr-manager-ruflo-main) | Agent skill for github-pr-manager - invoke with $agent-github-pr-manager | `ruflo-main` |
| [agent-gossip-coordinator](./agent-gossip-coordinator) | Agent skill for gossip-coordinator - invoke with $agent-gossip-coordinator | `ruflo` |
| [agent-gossip-coordinator-ruflo-main](./agent-gossip-coordinator-ruflo-main) | Agent skill for gossip-coordinator - invoke with $agent-gossip-coordinator | `ruflo-main` |
| [agent-harness-construction](./agent-harness-construction) | Design and optimize AI agent action spaces, tool definitions, and observation formatting for higher completion | `everything-claude-code-main` |
| [agent-hierarchical-coordinator](./agent-hierarchical-coordinator) | Agent skill for hierarchical-coordinator - invoke with $agent-hierarchical-coordinator | `ruflo` |
| [agent-hierarchical-coordinator-ruflo-main](./agent-hierarchical-coordinator-ruflo-main) | Agent skill for hierarchical-coordinator - invoke with $agent-hierarchical-coordinator | `ruflo-main` |
| [agent-hierarchy](./agent-hierarchy) | Designs orchestrator-and-subagent hierarchies for a repository — splitting agents by exclusive write surface,  | `headcount` |
| [agent-implementer-sparc-coder](./agent-implementer-sparc-coder) | Agent skill for implementer-sparc-coder - invoke with $agent-implementer-sparc-coder | `ruflo` |
| [agent-implementer-sparc-coder-ruflo-main](./agent-implementer-sparc-coder-ruflo-main) | Agent skill for implementer-sparc-coder - invoke with $agent-implementer-sparc-coder | `ruflo-main` |
| [agent-inspect](./agent-inspect) | >- | `agent-inspect` |
| [agent-introspection-debugging](./agent-introspection-debugging) | Structured self-debugging workflow for AI agent failures using capture, diagnosis, contained recovery, and int | `everything-claude-code-main` |
| [agent-issue-tracker](./agent-issue-tracker) | Agent skill for issue-tracker - invoke with $agent-issue-tracker | `ruflo` |
| [agent-issue-tracker-ruflo-main](./agent-issue-tracker-ruflo-main) | Agent skill for issue-tracker - invoke with $agent-issue-tracker | `ruflo-main` |
| [agent-load-balancer](./agent-load-balancer) | Agent skill for load-balancer - invoke with $agent-load-balancer | `ruflo` |
| [agent-load-balancer-ruflo-main](./agent-load-balancer-ruflo-main) | Agent skill for load-balancer - invoke with $agent-load-balancer | `ruflo-main` |
| [agent-matrix-optimizer](./agent-matrix-optimizer) | Agent skill for matrix-optimizer - invoke with $agent-matrix-optimizer | `ruflo` |
| [agent-matrix-optimizer-ruflo-main](./agent-matrix-optimizer-ruflo-main) | Agent skill for matrix-optimizer - invoke with $agent-matrix-optimizer | `ruflo-main` |
| [agent-memory](./agent-memory) | A hybrid memory system that provides persistent, searchable knowledge management for AI agents. | `agentic-awesome-skills` |
| [agent-memory-coordinator](./agent-memory-coordinator) | Agent skill for memory-coordinator - invoke with $agent-memory-coordinator | `ruflo` |
| [agent-memory-coordinator-ruflo-main](./agent-memory-coordinator-ruflo-main) | Agent skill for memory-coordinator - invoke with $agent-memory-coordinator | `ruflo-main` |
| [agent-memory-mcp](./agent-memory-mcp) | A hybrid memory system that provides persistent, searchable knowledge management for AI agents (Architecture,  | `agentic-awesome-skills` |
| [agent-memory-mcp-antigravity-awesome-skills-main](./agent-memory-mcp-antigravity-awesome-skills-main) | A hybrid memory system that provides persistent, searchable knowledge management for AI agents (Architecture,  | `antigravity-awesome-skills-main` |
| [agent-mesh-coordinator](./agent-mesh-coordinator) | Agent skill for mesh-coordinator - invoke with $agent-mesh-coordinator | `ruflo` |
| [agent-mesh-coordinator-ruflo-main](./agent-mesh-coordinator-ruflo-main) | Agent skill for mesh-coordinator - invoke with $agent-mesh-coordinator | `ruflo-main` |
| [agent-migration-plan](./agent-migration-plan) | Agent skill for migration-plan - invoke with $agent-migration-plan | `ruflo` |
| [agent-migration-plan-ruflo-main](./agent-migration-plan-ruflo-main) | Agent skill for migration-plan - invoke with $agent-migration-plan | `ruflo-main` |
| [agent-multi-repo-swarm](./agent-multi-repo-swarm) | Agent skill for multi-repo-swarm - invoke with $agent-multi-repo-swarm | `ruflo` |
| [agent-multi-repo-swarm-ruflo-main](./agent-multi-repo-swarm-ruflo-main) | Agent skill for multi-repo-swarm - invoke with $agent-multi-repo-swarm | `ruflo-main` |
| [agent-neural-network](./agent-neural-network) | Agent skill for neural-network - invoke with $agent-neural-network | `ruflo` |
| [agent-neural-network-ruflo-main](./agent-neural-network-ruflo-main) | Agent skill for neural-network - invoke with $agent-neural-network | `ruflo-main` |
| [agent-ops-cicd-github](./agent-ops-cicd-github) | Agent skill for ops-cicd-github - invoke with $agent-ops-cicd-github | `ruflo` |
| [agent-ops-cicd-github-ruflo-main](./agent-ops-cicd-github-ruflo-main) | Agent skill for ops-cicd-github - invoke with $agent-ops-cicd-github | `ruflo-main` |
| [agent-optimization](./agent-optimization) | Improve an Agent State through versioned scores and score-linked Traces from a frozen Benchmark. | `penguin-harness` |
| [agent-orchestration](./agent-orchestration) | Agent Orchestration Rules | `Continuous-Claude-v3-main` |
| [agent-orchestration-improve-agent](./agent-orchestration-improve-agent) | Systematic improvement of existing agents through performance analysis, prompt engineering, and continuous ite | `agentic-awesome-skills` |
| [agent-orchestration-improve-agent-antigravity-awesome-skills-main](./agent-orchestration-improve-agent-antigravity-awesome-skills-main) | Systematic improvement of existing agents through performance analysis, prompt engineering, and continuous ite | `antigravity-awesome-skills-main` |
| [agent-orchestration-multi-agent-optimize](./agent-orchestration-multi-agent-optimize) | Optimize multi-agent systems with coordinated profiling, workload distribution, and cost-aware orchestration.  | `agentic-awesome-skills` |
| [agent-orchestration-multi-agent-optimize-antigravity-awesome-skills-main](./agent-orchestration-multi-agent-optimize-antigravity-awesome-skills-main) | Optimize multi-agent systems with coordinated profiling, workload distribution, and cost-aware orchestration.  | `antigravity-awesome-skills-main` |
| [agent-orchestrator](./agent-orchestrator) | Open-source, pluggable agentic coding orchestrator. Manages durable coding agents (Claude Code, Codex, OpenCod | `agent-orchestrator-main` |
| [agent-orchestrator-agentic-awesome-skills](./agent-orchestrator-agentic-awesome-skills) | Meta-skill que orquestra todos os agentes do ecossistema. Scan automatico de skills, match por capacidades, co | `agentic-awesome-skills` |
| [agent-orchestrator-antigravity-awesome-skills-main](./agent-orchestrator-antigravity-awesome-skills-main) | Meta-skill que orquestra todos os agentes do ecossistema. Scan automatico de skills, match por capacidades, co | `antigravity-awesome-skills-main` |
| [agent-orchestrator-task](./agent-orchestrator-task) | Agent skill for orchestrator-task - invoke with $agent-orchestrator-task | `ruflo` |
| [agent-orchestrator-task-ruflo-main](./agent-orchestrator-task-ruflo-main) | Agent skill for orchestrator-task - invoke with $agent-orchestrator-task | `ruflo-main` |
| [agent-payments](./agent-payments) | Agent skill for payments - invoke with $agent-payments | `ruflo` |
| [agent-payments-ruflo-main](./agent-payments-ruflo-main) | Agent skill for payments - invoke with $agent-payments | `ruflo-main` |
| [agent-performance-analyzer](./agent-performance-analyzer) | Agent skill for performance-analyzer - invoke with $agent-performance-analyzer | `ruflo` |
| [agent-performance-analyzer-ruflo-main](./agent-performance-analyzer-ruflo-main) | Agent skill for performance-analyzer - invoke with $agent-performance-analyzer | `ruflo-main` |
| [agent-performance-benchmarker](./agent-performance-benchmarker) | Agent skill for performance-benchmarker - invoke with $agent-performance-benchmarker | `ruflo` |
| [agent-performance-benchmarker-ruflo-main](./agent-performance-benchmarker-ruflo-main) | Agent skill for performance-benchmarker - invoke with $agent-performance-benchmarker | `ruflo-main` |
| [agent-performance-monitor](./agent-performance-monitor) | Agent skill for performance-monitor - invoke with $agent-performance-monitor | `ruflo` |
| [agent-performance-monitor-ruflo-main](./agent-performance-monitor-ruflo-main) | Agent skill for performance-monitor - invoke with $agent-performance-monitor | `ruflo-main` |
| [agent-performance-optimizer](./agent-performance-optimizer) | Agent skill for performance-optimizer - invoke with $agent-performance-optimizer | `ruflo` |
| [agent-performance-optimizer-ruflo-main](./agent-performance-optimizer-ruflo-main) | Agent skill for performance-optimizer - invoke with $agent-performance-optimizer | `ruflo-main` |
| [agent-planner](./agent-planner) | Agent skill for planner - invoke with $agent-planner | `ruflo` |
| [agent-planner-ruflo-main](./agent-planner-ruflo-main) | Agent skill for planner - invoke with $agent-planner | `ruflo-main` |
| [agent-pr-manager](./agent-pr-manager) | Agent skill for pr-manager - invoke with $agent-pr-manager | `ruflo` |
| [agent-pr-manager-ruflo-main](./agent-pr-manager-ruflo-main) | Agent skill for pr-manager - invoke with $agent-pr-manager | `ruflo-main` |
| [agent-production-validator](./agent-production-validator) | Agent skill for production-validator - invoke with $agent-production-validator | `ruflo` |
| [agent-production-validator-ruflo-main](./agent-production-validator-ruflo-main) | Agent skill for production-validator - invoke with $agent-production-validator | `ruflo-main` |
| [agent-project-board-sync](./agent-project-board-sync) | Agent skill for project-board-sync - invoke with $agent-project-board-sync | `ruflo` |
| [agent-project-board-sync-ruflo-main](./agent-project-board-sync-ruflo-main) | Agent skill for project-board-sync - invoke with $agent-project-board-sync | `ruflo-main` |
| [agent-pseudocode](./agent-pseudocode) | Agent skill for pseudocode - invoke with $agent-pseudocode | `ruflo` |
| [agent-pseudocode-ruflo-main](./agent-pseudocode-ruflo-main) | Agent skill for pseudocode - invoke with $agent-pseudocode | `ruflo-main` |
| [agent-queen-coordinator](./agent-queen-coordinator) | Agent skill for queen-coordinator - invoke with $agent-queen-coordinator | `ruflo` |
| [agent-queen-coordinator-ruflo-main](./agent-queen-coordinator-ruflo-main) | Agent skill for queen-coordinator - invoke with $agent-queen-coordinator | `ruflo-main` |
| [agent-quorum-manager](./agent-quorum-manager) | Agent skill for quorum-manager - invoke with $agent-quorum-manager | `ruflo` |
| [agent-quorum-manager-ruflo-main](./agent-quorum-manager-ruflo-main) | Agent skill for quorum-manager - invoke with $agent-quorum-manager | `ruflo-main` |
| [agent-raft-manager](./agent-raft-manager) | Agent skill for raft-manager - invoke with $agent-raft-manager | `ruflo` |
| [agent-raft-manager-ruflo-main](./agent-raft-manager-ruflo-main) | Agent skill for raft-manager - invoke with $agent-raft-manager | `ruflo-main` |
| [agent-refinement](./agent-refinement) | Agent skill for refinement - invoke with $agent-refinement | `ruflo` |
| [agent-refinement-ruflo-main](./agent-refinement-ruflo-main) | Agent skill for refinement - invoke with $agent-refinement | `ruflo-main` |
| [agent-release-manager](./agent-release-manager) | Agent skill for release-manager - invoke with $agent-release-manager | `ruflo` |
| [agent-release-manager-ruflo-main](./agent-release-manager-ruflo-main) | Agent skill for release-manager - invoke with $agent-release-manager | `ruflo-main` |
| [agent-release-swarm](./agent-release-swarm) | Agent skill for release-swarm - invoke with $agent-release-swarm | `ruflo` |
| [agent-release-swarm-ruflo-main](./agent-release-swarm-ruflo-main) | Agent skill for release-swarm - invoke with $agent-release-swarm | `ruflo-main` |
| [agent-repo-architect](./agent-repo-architect) | Agent skill for repo-architect - invoke with $agent-repo-architect | `ruflo` |
| [agent-repo-architect-ruflo-main](./agent-repo-architect-ruflo-main) | Agent skill for repo-architect - invoke with $agent-repo-architect | `ruflo-main` |
| [agent-researcher](./agent-researcher) | Agent skill for researcher - invoke with $agent-researcher | `ruflo` |
| [agent-researcher-ruflo-main](./agent-researcher-ruflo-main) | Agent skill for researcher - invoke with $agent-researcher | `ruflo-main` |
| [agent-resource-allocator](./agent-resource-allocator) | Agent skill for resource-allocator - invoke with $agent-resource-allocator | `ruflo` |
| [agent-resource-allocator-ruflo-main](./agent-resource-allocator-ruflo-main) | Agent skill for resource-allocator - invoke with $agent-resource-allocator | `ruflo-main` |
| [agent-safla-neural](./agent-safla-neural) | Agent skill for safla-neural - invoke with $agent-safla-neural | `ruflo` |
| [agent-safla-neural-ruflo-main](./agent-safla-neural-ruflo-main) | Agent skill for safla-neural - invoke with $agent-safla-neural | `ruflo-main` |
| [agent-sandbox](./agent-sandbox) | Agent skill for sandbox - invoke with $agent-sandbox | `ruflo` |
| [agent-sandbox-ruflo-main](./agent-sandbox-ruflo-main) | Agent skill for sandbox - invoke with $agent-sandbox | `ruflo-main` |
| [agent-self-scheduling](./agent-self-scheduling) | Schedule AI agent runs with cron, loops, or external clocks while avoiding unsafe tight autonomous timers. | `agentic-awesome-skills` |
| [Agent-Skills-for-Context-Engineering-main](./Agent-Skills-for-Context-Engineering-main) | A comprehensive collection of Agent Skills for context engineering, multi-agent architectures, and production  | `Agent-Skills-for-Context-Engineering-main` |
| [agent-sona-learning-optimizer](./agent-sona-learning-optimizer) | Agent skill for sona-learning-optimizer - invoke with $agent-sona-learning-optimizer | `ruflo` |
| [agent-sona-learning-optimizer-ruflo-main](./agent-sona-learning-optimizer-ruflo-main) | Agent skill for sona-learning-optimizer - invoke with $agent-sona-learning-optimizer | `ruflo-main` |
| [agent-sort](./agent-sort) | Build an evidence-backed ECC install plan for a specific repo by sorting skills, commands, rules, hooks, and e | `everything-claude-code-main` |
| [agent-sparc-coordinator](./agent-sparc-coordinator) | Agent skill for sparc-coordinator - invoke with $agent-sparc-coordinator | `ruflo` |
| [agent-sparc-coordinator-ruflo-main](./agent-sparc-coordinator-ruflo-main) | Agent skill for sparc-coordinator - invoke with $agent-sparc-coordinator | `ruflo-main` |
| [agent-spec-mobile-react-native](./agent-spec-mobile-react-native) | Agent skill for spec-mobile-react-native - invoke with $agent-spec-mobile-react-native | `ruflo` |
| [agent-spec-mobile-react-native-ruflo-main](./agent-spec-mobile-react-native-ruflo-main) | Agent skill for spec-mobile-react-native - invoke with $agent-spec-mobile-react-native | `ruflo-main` |
| [agent-specification](./agent-specification) | Agent skill for specification - invoke with $agent-specification | `ruflo` |
| [agent-specification-ruflo-main](./agent-specification-ruflo-main) | Agent skill for specification - invoke with $agent-specification | `ruflo-main` |
| [agent-squad](./agent-squad) | Main agent orchestrator that coordinates a specialized squad of agents | `agentic-awesome-skills` |
| [agent-swarm](./agent-swarm) | Agent skill for swarm - invoke with $agent-swarm | `ruflo` |
| [agent-swarm-issue](./agent-swarm-issue) | Agent skill for swarm-issue - invoke with $agent-swarm-issue | `ruflo` |
| [agent-swarm-issue-ruflo-main](./agent-swarm-issue-ruflo-main) | Agent skill for swarm-issue - invoke with $agent-swarm-issue | `ruflo-main` |
| [agent-swarm-memory-manager](./agent-swarm-memory-manager) | Agent skill for swarm-memory-manager - invoke with $agent-swarm-memory-manager | `ruflo` |
| [agent-swarm-memory-manager-ruflo-main](./agent-swarm-memory-manager-ruflo-main) | Agent skill for swarm-memory-manager - invoke with $agent-swarm-memory-manager | `ruflo-main` |
| [agent-swarm-pr](./agent-swarm-pr) | Agent skill for swarm-pr - invoke with $agent-swarm-pr | `ruflo` |
| [agent-swarm-pr-ruflo-main](./agent-swarm-pr-ruflo-main) | Agent skill for swarm-pr - invoke with $agent-swarm-pr | `ruflo-main` |
| [agent-swarm-ruflo-main](./agent-swarm-ruflo-main) | Agent skill for swarm - invoke with $agent-swarm | `ruflo-main` |
| [agent-sync-coordinator](./agent-sync-coordinator) | Agent skill for sync-coordinator - invoke with $agent-sync-coordinator | `ruflo` |
| [agent-sync-coordinator-ruflo-main](./agent-sync-coordinator-ruflo-main) | Agent skill for sync-coordinator - invoke with $agent-sync-coordinator | `ruflo-main` |
| [agent-tdd-london-swarm](./agent-tdd-london-swarm) | Agent skill for tdd-london-swarm - invoke with $agent-tdd-london-swarm | `ruflo` |
| [agent-tdd-london-swarm-ruflo-main](./agent-tdd-london-swarm-ruflo-main) | Agent skill for tdd-london-swarm - invoke with $agent-tdd-london-swarm | `ruflo-main` |
| [agent-teams](./agent-teams) | Coordinate multiple Claude Code sessions as a team — lead + teammates with shared task lists, mailbox messagin | `pro-workflow` |
| [agent-test-long-runner](./agent-test-long-runner) | Agent skill for test-long-runner - invoke with $agent-test-long-runner | `ruflo` |
| [agent-test-long-runner-ruflo-main](./agent-test-long-runner-ruflo-main) | Agent skill for test-long-runner - invoke with $agent-test-long-runner | `ruflo-main` |
| [agent-user-tools](./agent-user-tools) | Agent skill for user-tools - invoke with $agent-user-tools | `ruflo` |
| [agent-user-tools-ruflo-main](./agent-user-tools-ruflo-main) | Agent skill for user-tools - invoke with $agent-user-tools | `ruflo-main` |
| [agent-v3-integration-architect](./agent-v3-integration-architect) | Agent skill for v3-integration-architect - invoke with $agent-v3-integration-architect | `ruflo` |
| [agent-v3-integration-architect-ruflo-main](./agent-v3-integration-architect-ruflo-main) | Agent skill for v3-integration-architect - invoke with $agent-v3-integration-architect | `ruflo-main` |
| [agent-v3-performance-engineer](./agent-v3-performance-engineer) | Agent skill for v3-performance-engineer - invoke with $agent-v3-performance-engineer | `ruflo` |
| [agent-v3-performance-engineer-ruflo-main](./agent-v3-performance-engineer-ruflo-main) | Agent skill for v3-performance-engineer - invoke with $agent-v3-performance-engineer | `ruflo-main` |
| [agent-v3-queen-coordinator](./agent-v3-queen-coordinator) | Agent skill for v3-queen-coordinator - invoke with $agent-v3-queen-coordinator | `ruflo` |
| [agent-v3-queen-coordinator-ruflo-main](./agent-v3-queen-coordinator-ruflo-main) | Agent skill for v3-queen-coordinator - invoke with $agent-v3-queen-coordinator | `ruflo-main` |
| [agent-worker-specialist](./agent-worker-specialist) | Agent skill for worker-specialist - invoke with $agent-worker-specialist | `ruflo` |
| [agent-worker-specialist-ruflo-main](./agent-worker-specialist-ruflo-main) | Agent skill for worker-specialist - invoke with $agent-worker-specialist | `ruflo-main` |
| [agent-workflow](./agent-workflow) | Agent skill for workflow - invoke with $agent-workflow | `ruflo` |
| [agent-workflow-automation](./agent-workflow-automation) | Agent skill for workflow-automation - invoke with $agent-workflow-automation | `ruflo` |
| [agent-workflow-automation-ruflo-main](./agent-workflow-automation-ruflo-main) | Agent skill for workflow-automation - invoke with $agent-workflow-automation | `ruflo-main` |
| [agent-workflow-ruflo-main](./agent-workflow-ruflo-main) | Agent skill for workflow - invoke with $agent-workflow | `ruflo-main` |
| [agentdb-advanced](./agentdb-advanced) | Master advanced AgentDB features including QUIC synchronization, multi-database management, custom distance me | `ruflo` |
| [agentdb-advanced-ruflo](./agentdb-advanced-ruflo) | Master advanced AgentDB features including QUIC synchronization, multi-database management, custom distance me | `ruflo` |
| [agentdb-advanced-ruflo-main](./agentdb-advanced-ruflo-main) | Master advanced AgentDB features including QUIC synchronization, multi-database management, custom distance me | `ruflo-main` |
| [agentdb-advanced-ruflo-main](./agentdb-advanced-ruflo-main) | Master advanced AgentDB features including QUIC synchronization, multi-database management, custom distance me | `ruflo-main` |
| [agentdb-memory-patterns](./agentdb-memory-patterns) | Implement persistent memory patterns for AI agents using AgentDB. Includes session memory, long-term storage,  | `ruflo` |
| [agentdb-memory-patterns-ruflo](./agentdb-memory-patterns-ruflo) | Implement persistent memory patterns for AI agents using AgentDB. Includes session memory, long-term storage,  | `ruflo` |
| [agentdb-memory-patterns-ruflo-main](./agentdb-memory-patterns-ruflo-main) | Implement persistent memory patterns for AI agents using AgentDB. Includes session memory, long-term storage,  | `ruflo-main` |
| [agentdb-memory-patterns-ruflo-main](./agentdb-memory-patterns-ruflo-main) | Implement persistent memory patterns for AI agents using AgentDB. Includes session memory, long-term storage,  | `ruflo-main` |
| [agentflow](./agentflow) | Orchestrate autonomous AI development pipelines through your Kanban board (Asana, GitHub Projects, Linear). Ma | `agentic-awesome-skills` |
| [agentflow-antigravity-awesome-skills-main](./agentflow-antigravity-awesome-skills-main) | Orchestrate autonomous AI development pipelines through your Kanban board (Asana, GitHub Projects, Linear). Ma | `antigravity-awesome-skills-main` |
| [agentfolio](./agentfolio) | Skill for discovering and researching autonomous AI agents, tools, and ecosystems using the AgentFolio directo | `agentic-awesome-skills` |
| [agentfolio-antigravity-awesome-skills-main](./agentfolio-antigravity-awesome-skills-main) | Skill for discovering and researching autonomous AI agents, tools, and ecosystems using the AgentFolio directo | `antigravity-awesome-skills-main` |
| [agentic-engineering](./agentic-engineering) | Operate as an agentic engineer using eval-first execution, decomposition, and cost-aware model routing. | `everything-claude-code-main` |
| [agentic-jujutsu](./agentic-jujutsu) | \| | `ruflo` |
| [agentic-jujutsu-ruflo](./agentic-jujutsu-ruflo) | Quantum-resistant, self-learning version control for AI agents with ReasoningBank intelligence and multi-agent | `ruflo` |
| [agentic-jujutsu-ruflo-main](./agentic-jujutsu-ruflo-main) | \| | `ruflo-main` |
| [agentic-jujutsu-ruflo-main](./agentic-jujutsu-ruflo-main) | Quantum-resistant, self-learning version control for AI agents with ReasoningBank intelligence and multi-agent | `ruflo-main` |
| [agentic-os](./agentic-os) | Build persistent multi-agent operating systems on Claude Code. Covers kernel architecture, specialist agents,  | `ECC` |
| [agentica-claude-proxy](./agentica-claude-proxy) | Guide for integrating Agentica SDK with Claude Code CLI proxy | `Continuous-Claude-v3-main` |
| [agentica-prompts](./agentica-prompts) | Write reliable prompts for Agentica/REPL agents that avoid LLM instruction ambiguity | `Continuous-Claude-v3-main` |
| [agentica-sdk](./agentica-sdk) | Build Python agents with Agentica SDK - @agentic decorator, spawn(), persistence, MCP integration | `Continuous-Claude-v3-main` |
| [agents-generator](./agents-generator) | Generate project-specific AGENTS.md and companion rules by analyzing a codebase. Supports full, minimal, updat | `agentic-awesome-skills` |
| [agents-spec](./agents-spec) | Audit, reorganize, split, migrate, and maintain AGENTS.md, optional platform instruction files, and non-busine | `agents-spec-skill` |
| [agenttrace-session-audit](./agenttrace-session-audit) | Audit local AI coding-agent sessions with agenttrace for cost, tool failures, latency, anomalies, health, diff | `agentic-awesome-skills` |
| [agilerl](./agilerl) | Use AgileRL for reinforcement learning workflows: classical RL | `AREX-Skill` |
| [agora-awareness](./agora-awareness) | Understand the Agora multi-agent collaboration framework — how teams, projects, kanban, discussions, and heart | `Agora` |
| [agora-setup](./agora-setup) | Set up an Agora multi-agent team from scratch — create workers, form teams, start projects. Read this when the | `Agora` |
| [agy-delegate](./agy-delegate) | Delegate coding tasks to the Google Antigravity CLI (`agy`) only when | `agentic-awesome-skills` |
| [ai-agent-development](./ai-agent-development) | AI agent development workflow for building autonomous agents, multi-agent systems, and agent orchestration wit | `agentic-awesome-skills` |
| [ai-agent-development-antigravity-awesome-skills-main](./ai-agent-development-antigravity-awesome-skills-main) | AI agent development workflow for building autonomous agents, multi-agent systems, and agent orchestration wit | `antigravity-awesome-skills-main` |
| [ai-agents-architect](./ai-agents-architect) | Expert in designing and building autonomous AI agents. Masters tool | `agentic-awesome-skills` |
| [ai-agents-architect-antigravity-awesome-skills-main](./ai-agents-architect-antigravity-awesome-skills-main) | Expert in designing and building autonomous AI agents. Masters tool | `antigravity-awesome-skills-main` |
| [ai-data-science-team](./ai-data-science-team) | Operate the ai-data-science-team package for AI-assisted data | `AREX-Skill` |
| [ai-dev-jobs-mcp](./ai-dev-jobs-mcp) | Search 8,400+ AI and ML jobs across 489 companies, inspect listings and employers, match roles, and view salar | `agentic-awesome-skills` |
| [ai-first-engineering](./ai-first-engineering) | Engineering operating model for teams where AI agents generate a large share of implementation output. | `everything-claude-code-main` |
| [ai-ide-strategy-writing](./ai-ide-strategy-writing) | AI-IDE 写策略与执行 — 用 AI 生成 Qlib 量化策略代码、Docker 容器执行策略/回测、自然语言条件选股、策略落库。在 QuantBot / Claude Code 中让 AI 写策略、生成 Qlib  | `QuantMind` |
| [ai-ml](./ai-ml) | AI and machine learning workflow covering LLM application development, RAG implementation, agent architecture, | `agentic-awesome-skills` |
| [ai-ml-antigravity-awesome-skills-main](./ai-ml-antigravity-awesome-skills-main) | AI and machine learning workflow covering LLM application development, RAG implementation, agent architecture, | `antigravity-awesome-skills-main` |
| [ai-optimizer](./ai-optimizer) | Route AI-Optimizer reinforcement-learning collection tasks across | `AREX-Skill` |
| [ai-regression-testing](./ai-regression-testing) | Regression testing strategies for AI-assisted development. Sandbox-mode API testing without database dependenc | `ECC` |
| [ai-regression-testing-everything-claude-code-main](./ai-regression-testing-everything-claude-code-main) | Regression testing strategies for AI-assisted development. Sandbox-mode API testing without database dependenc | `everything-claude-code-main` |
| [ai-studio-image](./ai-studio-image) | Geracao de imagens humanizadas via Google AI Studio (Gemini). Fotos realistas estilo influencer ou educacional | `agentic-awesome-skills` |
| [ai-studio-image-antigravity-awesome-skills-main](./ai-studio-image-antigravity-awesome-skills-main) | Geracao de imagens humanizadas via Google AI Studio (Gemini). Fotos realistas estilo influencer ou educacional | `antigravity-awesome-skills-main` |
| [aider-delegate](./aider-delegate) | Delegate coding tasks to Aider (`aider`) only when the user explicitly | `agentic-awesome-skills` |
| [airtable](./airtable) | Airtable REST API via curl. Records CRUD, filters, upserts. | `hermes-agent` |
| [airtable-automation](./airtable-automation) | Automate Airtable tasks via Rube MCP (Composio): records, bases, tables, fields, views. Always search tools fi | `agentic-awesome-skills` |
| [airtable-automation-antigravity-awesome-skills-main](./airtable-automation-antigravity-awesome-skills-main) | Automate Airtable tasks via Rube MCP (Composio): records, bases, tables, fields, views. Always search tools fi | `antigravity-awesome-skills-main` |
| [airweave](./airweave) | Route Airweave tasks to the most specific sub-skill: | `AREX-Skill` |
| [alert-management](./alert-management) | > | `Claude-Code-Agent-Monitor` |
| [alpha-vantage](./alpha-vantage) | Access 20+ years of global financial data: equities, options, forex, crypto, commodities, economic indicators, | `agentic-awesome-skills` |
| [alpha-vantage-antigravity-awesome-skills-main](./alpha-vantage-antigravity-awesome-skills-main) | Access 20+ years of global financial data: equities, options, forex, crypto, commodities, economic indicators, | `antigravity-awesome-skills-main` |
| [amazon-alexa](./amazon-alexa) | Integracao completa com Amazon Alexa para criar skills de voz inteligentes, transformar Alexa em assistente co | `agentic-awesome-skills` |
| [amazon-alexa-antigravity-awesome-skills-main](./amazon-alexa-antigravity-awesome-skills-main) | Integracao completa com Amazon Alexa para criar skills de voz inteligentes, transformar Alexa em assistente co | `antigravity-awesome-skills-main` |
| [amplitude-automation](./amplitude-automation) | Automate Amplitude tasks via Rube MCP (Composio): events, user activity, cohorts, user identification. Always  | `agentic-awesome-skills` |
| [amplitude-automation-antigravity-awesome-skills-main](./amplitude-automation-antigravity-awesome-skills-main) | Automate Amplitude tasks via Rube MCP (Composio): events, user activity, cohorts, user identification. Always  | `antigravity-awesome-skills-main` |
| [analytical-method-validation](./analytical-method-validation) | Plan, execute, and document validation, verification, and transfer of analytical procedures under the governin | `scientific-agent-skills` |
| [analytics-product](./analytics-product) | Analytics de produto — PostHog, Mixpanel, eventos, funnels, cohorts, retencao, north star metric, OKRs e dashb | `agentic-awesome-skills` |
| [analytics-product-antigravity-awesome-skills-main](./analytics-product-antigravity-awesome-skills-main) | Analytics de produto — PostHog, Mixpanel, eventos, funnels, cohorts, retencao, north star metric, OKRs e dashb | `antigravity-awesome-skills-main` |
| [andrej-karpathy](./andrej-karpathy) | Behavioral guidelines to reduce common LLM coding mistakes. Use when writing, reviewing, or refactoring code t | `agentic-awesome-skills` |
| [andrej-karpathy-antigravity-awesome-skills-main](./andrej-karpathy-antigravity-awesome-skills-main) | Agente que simula Andrej Karpathy — ex-Director of AI da Tesla, co-fundador da OpenAI, fundador da Eureka Labs | `antigravity-awesome-skills-main` |
| [anomaly-alert](./anomaly-alert) | > | `Claude-Code-Agent-Monitor` |
| [anti-deception](./anti-deception) | Use before responding to pressure for agreement, manufactured urgency, authority appeals, or requests to certi | `agentic-awesome-skills` |
| [anti-sycophancy](./anti-sycophancy) | Eliminate sycophantic agreement patterns in AI responses. Load via /skill anti-sycophancy. | `agentic-awesome-skills` |
| [ao-desktop-dev](./ao-desktop-dev) | Launch, restart, or troubleshoot the real AO Electron desktop app from this repository; run a checkout against | `agent-orchestrator` |
| [ao-desktop-dev-agent-orchestrator](./ao-desktop-dev-agent-orchestrator) | Launch, restart, or troubleshoot the real AO Electron desktop app from this repository; run a checkout against | `agent-orchestrator` |
| [aomi-transact](./aomi-transact) | Build natural-language crypto/DeFi agents and EVM MCP plugins (Claude Code, Cursor, Codex, Gemini). Aomi turns | `agentic-awesome-skills` |
| [api-analyzer](./api-analyzer) | Validates whether an API request is correct based on provided inputs (method, URL, headers, body, auth, query  | `agentic-awesome-skills` |
| [api-and-interface-design](./api-and-interface-design) | Guides stable API and interface design. Use when designing APIs, module boundaries, or any public interface. U | `agentic-awesome-skills` |
| [api-connector-builder](./api-connector-builder) | Build a new API connector or provider by matching the target repo's existing integration pattern exactly. Use  | `everything-claude-code-main` |
| [api-design](./api-design) | REST API design patterns including resource naming, status codes, pagination, filtering, error responses, vers | `everything-claude-code-main` |
| [api-designer](./api-designer) | Generates complete, production-ready REST API endpoint specifications for any system or domain the user descri | `agentic-awesome-skills` |
| [api-error-report](./api-error-report) | > | `Claude-Code-Agent-Monitor` |
| [api-integration](./api-integration) | Designs event-driven architectures, webhook systems, API chaining flows, ETL pipelines, and integration patter | `agentic-awesome-skills` |
| [api-sdk-generator](./api-sdk-generator) | Generates client SDK code, API wrapper libraries, request/response models, and language-specific usage pattern | `agentic-awesome-skills` |
| [apify-actor-development](./apify-actor-development) | Important: Before you begin, fill in the generatedBy property in the meta section of .actor/actor.json. Replac | `agentic-awesome-skills` |
| [apify-actor-development-antigravity-awesome-skills-main](./apify-actor-development-antigravity-awesome-skills-main) | Important: Before you begin, fill in the generatedBy property in the meta section of .actor/actor.json. Replac | `antigravity-awesome-skills-main` |
| [app-builder](./app-builder) | Main application building orchestrator. Creates full-stack applications from natural language requests. Determ | `agentic-awesome-skills` |
| [app-builder-antigravity-awesome-skills-main](./app-builder-antigravity-awesome-skills-main) | Main application building orchestrator. Creates full-stack applications from natural language requests. Determ | `antigravity-awesome-skills-main` |
| [appdeploy](./appdeploy) | Deploy web apps with backend APIs, database, and file storage. Use when the user asks to deploy or publish a w | `agentic-awesome-skills` |
| [appdeploy-antigravity-awesome-skills-main](./appdeploy-antigravity-awesome-skills-main) | Deploy web apps with backend APIs, database, and file storage. Use when the user asks to deploy or publish a w | `antigravity-awesome-skills-main` |
| [appium-skill](./appium-skill) | Generates production-grade Appium mobile automation scripts for Android and iOS in Java, Python, or JavaScript | `agentic-awesome-skills` |
| [apple-hig](./apple-hig) | \| | `abingyyds__open-design` |
| [apple-hig-open-design](./apple-hig-open-design) | \| | `open-design` |
| [apple-notes](./apple-notes) | Manage Apple Notes via memo CLI: create, search, edit. | `hermes-agent` |
| [apple-reminders](./apple-reminders) | Apple Reminders via remindctl: add, list, complete. | `hermes-agent` |
| [arbor](./arbor) | Autonomously improve a real artifact (code, training recipe, agent harness, data pipeline, prompt) against an  | `scientific-agent-skills` |
| [arboreto](./arboreto) | Infer gene regulatory networks (GRNs) from gene expression data using scalable algorithms (GRNBoost2, GENIE3). | `scientific-agent-skills` |
| [architect-first](./architect-first) | Guide for implementing the Architect-First development philosophy - perfect architecture, pragmatic execution, | `aiox-core-main` |
| [architecture-decision](./architecture-decision) | Creates an Architecture Decision Record (ADR) documenting a significant technical decision, its context, alter | `Claude-Code-Game-Studios-main` |
| [architecture-decision-records](./architecture-decision-records) | Capture architectural decisions made during Claude Code sessions as structured ADRs. Auto-detects decision mom | `ECC` |
| [architecture-decision-records-everything-claude-code-main](./architecture-decision-records-everything-claude-code-main) | Capture architectural decisions made during Claude Code sessions as structured ADRs. Auto-detects decision mom | `everything-claude-code-main` |
| [architecture-diagram](./architecture-diagram) | Dark-themed SVG architecture/cloud/infra diagrams as HTML. | `hermes-agent` |
| [architecture-review](./architecture-review) | Validates completeness and consistency of the project architecture against all GDDs. Builds a traceability mat | `Claude-Code-Game-Studios-main` |
| [archon](./archon) | \| | `Archon-dev` |
| [argue](./argue) | Run structured multi-agent debates using argue CLI for cross-examined, high-confidence answers. Use when facin | `argue-master` |
| [art-bible](./art-bible) | Guided, section-by-section Art Bible authoring. Creates the visual identity specification that gates all asset | `Claude-Code-Game-Studios-main` |
| [article-writing](./article-writing) | Write articles, guides, blog posts, tutorials, newsletter issues, and other long-form content in a distinctive | `everything-claude-code-main` |
| [arxiv](./arxiv) | Search arXiv papers by keyword, author, category, or ID. | `hermes-agent` |
| [asana-automation](./asana-automation) | Automate Asana tasks via Rube MCP (Composio): tasks, projects, sections, teams, workspaces. Always search tool | `agentic-awesome-skills` |
| [asana-automation-antigravity-awesome-skills-main](./asana-automation-antigravity-awesome-skills-main) | Automate Asana tasks via Rube MCP (Composio): tasks, projects, sections, teams, workspaces. Always search tool | `antigravity-awesome-skills-main` |
| [ascii-video](./ascii-video) | ASCII video: convert video/audio to colored ASCII MP4/GIF. | `hermes-agent` |
| [ask-matt](./ask-matt) | Ask which skill or flow fits your situation. A router over the user-invoked skills in this repo. | `agentic-awesome-skills` |
| [asset-audit](./asset-audit) | Audits game assets for compliance with naming conventions, file size budgets, format standards, and pipeline r | `Claude-Code-Game-Studios-main` |
| [astra-orchestrator](./astra-orchestrator) | Orchestrate complex Codex coding work with the root agent as planner/integrator, Luna subagents for exploratio | `donvito__codex-astra-luna-orchestrator` |
| [astropy](./astropy) | Astropy is the core Python package for astronomy, providing essential functionality for astronomical research  | `agentic-awesome-skills` |
| [astropy-antigravity-awesome-skills-main](./astropy-antigravity-awesome-skills-main) | Astropy is the core Python package for astronomy, providing essential functionality for astronomical research  | `antigravity-awesome-skills-main` |
| [astropy-scientific-agent-skills](./astropy-scientific-agent-skills) | Core Python library for astronomy and astrophysics workflows that need Astropy APIs, including units/quantitie | `scientific-agent-skills` |
| [audit-choices](./audit-choices) | Audit the choices an implementing agent made, not its diff — a pure decision audit that traces the session's h | `skills` |
| [auri-core](./auri-core) | Auri: assistente de voz inteligente (Alexa + Claude claude-opus-4-20250805). Visao do produto, persona Vitoria | `antigravity-awesome-skills-main` |
| [auto-memory](./auto-memory) | This skill should be used when the user asks to 'set up memory', 'create memory files', 'create MEMORY.md', 's | `arc-kit-main` |
| [auto-research](./auto-research) | Research uncertain questions with an explicit, user-approved web search or ChatGPT consultation, then present  | `agentic-awesome-skills` |
| [autogen](./autogen) | Use this skill for maintaining, migrating, debugging, and safely | `AREX-Skill` |
| [autonomous-agent-harness](./autonomous-agent-harness) | Transform Claude Code into a fully autonomous agent system with persistent memory, scheduled operations, compu | `ECC` |
| [autonomous-agent-harness-everything-claude-code-main](./autonomous-agent-harness-everything-claude-code-main) | Transform Claude Code into a fully autonomous agent system with persistent memory, scheduled operations, compu | `everything-claude-code-main` |
| [autonomous-agent-patterns](./autonomous-agent-patterns) | Design patterns for building autonomous coding agents, inspired by [Cline](https://github.com/cline/cline) and | `agentic-awesome-skills` |
| [autonomous-agent-patterns-antigravity-awesome-skills-main](./autonomous-agent-patterns-antigravity-awesome-skills-main) | Design patterns for building autonomous coding agents, inspired by [Cline](https://github.com/cline/cline) and | `antigravity-awesome-skills-main` |
| [autonomous-loops](./autonomous-loops) | Patterns and architectures for autonomous Claude Code loops — from simple sequential pipelines to RFC-driven m | `ECC` |
| [autonomous-loops-everything-claude-code-main](./autonomous-loops-everything-claude-code-main) | Patterns and architectures for autonomous Claude Code loops — from simple sequential pipelines to RFC-driven m | `everything-claude-code-main` |
| [autopilot-loop](./autopilot-loop) | Run an autonomous /loop iteration -- check progress, work on next task, schedule next wake | `ruflo` |
| [autoscirub](./autoscirub) | Coordinate the full AutoSciRub workflow for autonomous scientific research tasks. Use when a user wants to tur | `AutoSciRub` |
| [autoskill](./autoskill) | Observe the user's screen via screenpipe, detect repeated research workflows, match them against existing scie | `scientific-agent-skills` |
| [avoid-ai-writing](./avoid-ai-writing) | Audit and rewrite content to remove AI writing patterns ("AI-isms"). Use this skill when asked to "remove AI-i | `avoid-ai-writing` |
| [aws-cdk-development](./aws-cdk-development) | AWS Cloud Development Kit (CDK) expert for building cloud infrastructure with TypeScript/Python. | `agentic-awesome-skills` |
| [aws-cost-operations](./aws-cost-operations) | AWS cost optimization, monitoring, and operational excellence expert. Use when analyzing AWS bills, estimating | `agentic-awesome-skills` |
| [aws-mcp-setup](./aws-mcp-setup) | Configure AWS MCP servers for documentation search and API access. Use when setting up AWS MCP, configuring AW | `agentic-awesome-skills` |
| [aws-serverless-eda](./aws-serverless-eda) | AWS serverless and event-driven architecture expert based on Well-Architected Framework. Use when building ser | `agentic-awesome-skills` |
| [backend-module-standards](./backend-module-standards) | Enforce this repository's TypeScript backend module architecture standards. Use whenever creating, modifying,  | `claudecodeui` |
| [backend-patterns](./backend-patterns) | Backend architecture patterns, API design, database optimization, and server-side best practices for Node.js,  | `everything-claude-code-main` |
| [backtest-center](./backtest-center) | 回测中心 — 快速回测、专家模式、回测历史、策略对比、参数优化、策略管理、高级分析。在 QuantBot / Claude Code 中运行 Qlib 回测、对比策略、优化参数、分析回测结果、管理策略时使用。触发词：回测 | `QuantMind` |
| [balance-check](./balance-check) | Analyzes game balance data files, formulas, and configuration to identify outliers, broken progressions, degen | `Claude-Code-Game-Studios-main` |
| [bamboohr-automation](./bamboohr-automation) | Automate BambooHR tasks via Rube MCP (Composio): employees, time-off, benefits, dependents, employee updates.  | `agentic-awesome-skills` |
| [bamboohr-automation-antigravity-awesome-skills-main](./bamboohr-automation-antigravity-awesome-skills-main) | Automate BambooHR tasks via Rube MCP (Composio): employees, time-off, benefits, dependents, employee updates.  | `antigravity-awesome-skills-main` |
| [baoyu-infographic](./baoyu-infographic) | Infographics: 21 layouts x 21 styles (信息图, 可视化). | `hermes-agent` |
| [basecamp-automation](./basecamp-automation) | Automate Basecamp project management, to-dos, messages, people, and to-do list organization via Rube MCP (Comp | `agentic-awesome-skills` |
| [basecamp-automation-antigravity-awesome-skills-main](./basecamp-automation-antigravity-awesome-skills-main) | Automate Basecamp project management, to-dos, messages, people, and to-do list organization via Rube MCP (Comp | `antigravity-awesome-skills-main` |
| [batch-inference-analysis](./batch-inference-analysis) | 批量推理结果分析 — 用 QuantMind 选股策略方法论分析每日信号、行业轮动、个股分数区间、负分参考。在 QuantBot / Claude Code 中分析批量推理结果、解读每日信号、判断市场状态、选股决策、做空 | `QuantMind` |
| [bdi-mental-states](./bdi-mental-states) | This skill should be used when the user asks to "model agent mental states", "implement BDI architecture", "cr | `Agent-Skills-for-Context-Engineering-main` |
| [bdi-mental-states-agentic-awesome-skills](./bdi-mental-states-agentic-awesome-skills) | This skill should be used when the user asks to "model agent mental states", "implement BDI architecture", "cr | `agentic-awesome-skills` |
| [bdi-mental-states-antigravity-awesome-skills-main](./bdi-mental-states-antigravity-awesome-skills-main) | This skill should be used when the user asks to "model agent mental states", "implement BDI architecture", "cr | `antigravity-awesome-skills-main` |
| [beads](./beads) | > | `beads` |
| [benchling-integration](./benchling-integration) | Benchling Python SDK and REST API integration for registry entities, inventory, ELN entries, workflows, Benchl | `scientific-agent-skills` |
| [benchmark](./benchmark) | > | `Claude-Code-Agent-Monitor` |
| [benchmark-design](./benchmark-design) | Design and calibrate a multi-Case capability Benchmark and establish a traceable Formal Baseline. | `penguin-harness` |
| [benchmark-everything-claude-code-main](./benchmark-everything-claude-code-main) | Use this skill to measure performance baselines, detect regressions before/after PRs, and compare stack altern | `everything-claude-code-main` |
| [bernstein-run](./bernstein-run) | > | `bernstein` |
| [better-fullstack](./better-fullstack) | Scaffold, plan, or extend Better Fullstack projects with the generator, CLI, or MCP server. Use when a user as | `Better-Fullstack` |
| [better-layout](./better-layout) | Helps with grouping, alignment, reading order, progressive disclosure and other details that make a good layou | `jakubkrehel__skills` |
| [bgpt-paper-search](./bgpt-paper-search) | Search scientific papers and retrieve structured experimental data extracted from full-text studies via the BG | `scientific-agent-skills` |
| [bids](./bids) | > | `scientific-agent-skills` |
| [bill-gates](./bill-gates) | Agente que simula Bill Gates — cofundador da Microsoft, arquiteto da industria de software comercial, estrateg | `agentic-awesome-skills` |
| [bill-gates-antigravity-awesome-skills-main](./bill-gates-antigravity-awesome-skills-main) | Agente que simula Bill Gates — cofundador da Microsoft, arquiteto da industria de software comercial, estrateg | `antigravity-awesome-skills-main` |
| [biopython](./biopython) | Biopython is a comprehensive set of freely available Python tools for biological computation. It provides func | `agentic-awesome-skills` |
| [biopython-antigravity-awesome-skills-main](./biopython-antigravity-awesome-skills-main) | Biopython is a comprehensive set of freely available Python tools for biological computation. It provides func | `antigravity-awesome-skills-main` |
| [bioservices](./bioservices) | Unified Python interface to 40+ bioinformatics services. Use when querying multiple databases (UniProt, KEGG,  | `scientific-agent-skills` |
| [bitbucket-automation](./bitbucket-automation) | Automate Bitbucket repositories, pull requests, branches, issues, and workspace management via Rube MCP (Compo | `agentic-awesome-skills` |
| [bitbucket-automation-antigravity-awesome-skills-main](./bitbucket-automation-antigravity-awesome-skills-main) | Automate Bitbucket repositories, pull requests, branches, issues, and workspace management via Rube MCP (Compo | `antigravity-awesome-skills-main` |
| [block-no-verify-hook](./block-no-verify-hook) | Configure a PreToolUse hook to prevent AI agents from skipping git pre-commit hooks with --no-verify and other | `agents-main` |
| [blocked-page-recovery](./blocked-page-recovery) | Use when a fetch fails: 403/429, paywall, WAF, bot wall. | `hermes-agent` |
| [blockrun](./blockrun) | BlockRun works with Claude Code and Google Antigravity. | `antigravity-awesome-skills-main` |
| [blueprint](./blueprint) | Turn a one-line objective into a step-by-step construction plan any coding agent can execute cold. Each step h | `agentic-awesome-skills` |
| [blueprint-antigravity-awesome-skills-main](./blueprint-antigravity-awesome-skills-main) | Turn a one-line objective into a step-by-step construction plan any coding agent can execute cold. Each step h | `antigravity-awesome-skills-main` |
| [blueprint-ECC](./blueprint-ECC) | >- | `ECC` |
| [blueprint-everything-claude-code-main](./blueprint-everything-claude-code-main) | >- | `everything-claude-code-main` |
| [book-to-skill](./book-to-skill) | Converts books and documents (PDF, EPUB, DOCX, HTML, Markdown, plain text, RTF, MOBI/AZW with Calibre) into st | `book-to-skill` |
| [box](./box) | Box manages cloud files, sharing, search, and metadata. | `hermes-agent` |
| [box-automation](./box-automation) | Automate Box operations including file upload/download, content search, folder management, collaboration, meta | `agentic-awesome-skills` |
| [box-automation-antigravity-awesome-skills-main](./box-automation-antigravity-awesome-skills-main) | Automate Box operations including file upload/download, content search, folder management, collaboration, meta | `antigravity-awesome-skills-main` |
| [brain-link-discipline](./brain-link-discipline) | \| | `gbrain` |
| [brainstorm](./brainstorm) | Guided game concept ideation — from zero idea to a structured game concept document. Uses professional studio  | `Claude-Code-Game-Studios-main` |
| [braintrust-analyze](./braintrust-analyze) | Analyze Claude Code sessions via Braintrust | `Continuous-Claude-v3-main` |
| [braintrust-tracing](./braintrust-tracing) | Braintrust tracing for Claude Code - hook architecture, sub-agent correlation, debugging | `Continuous-Claude-v3-main` |
| [brand](./brand) | Brand voice, visual identity, messaging frameworks, asset management, brand consistency. Activate for branded  | `ui-ux-pro-max-skill` |
| [brevo-automation](./brevo-automation) | Automate Brevo (formerly Sendinblue) email marketing operations through Composio's Brevo toolkit via Rube MCP. | `agentic-awesome-skills` |
| [brevo-automation-antigravity-awesome-skills-main](./brevo-automation-antigravity-awesome-skills-main) | Automate Brevo (formerly Sendinblue) email marketing operations through Composio's Brevo toolkit via Rube MCP. | `antigravity-awesome-skills-main` |
| [brooks-harness](./brooks-harness) | Maintenance orchestrator for the brooks-lint plugin itself. | `agentic-awesome-skills` |
| [brooks-lint](./brooks-lint) | AI code reviewer grounded in classic software engineering books for catching design smells, coupling issues, a | `agentic-awesome-skills` |
| [browser-harness-js](./browser-harness-js) | Drive Chrome via the DevTools Protocol from JavaScript. Run JS snippets through the `browser-harness-js` CLI — | `browser-harness-js` |
| [browser-qa](./browser-qa) | Use this skill to automate visual testing and UI interaction verification using browser automation after deplo | `everything-claude-code-main` |
| [browser-testing-with-devtools](./browser-testing-with-devtools) | Test browser apps with Chrome DevTools MCP by inspecting live DOM, console logs, network traffic, screenshots, | `agentic-awesome-skills` |
| [budget-set](./budget-set) | > | `Claude-Code-Agent-Monitor` |
| [bug-hunt-swarm](./bug-hunt-swarm) | Parallel read-only multi-agent root-cause investigation for bugs, regressions, crashes, flaky behavior, or une | `agentic-awesome-skills` |
| [bug-report](./bug-report) | Creates a structured bug report from a description, or analyzes code to identify potential bugs. Ensures every | `Claude-Code-Game-Studios-main` |
| [bug-triage](./bug-triage) | Triage bugs reported in chat/issues, search for duplicates, file or update GitHub issues with full context, an | `agent-orchestrator` |
| [bug-triage-agent-orchestrator](./bug-triage-agent-orchestrator) | Triage bugs reported in chat/issues, search for duplicates, file or update GitHub issues with full context, an | `agent-orchestrator` |
| [bug-triage-Claude-Code-Game-Studios-main](./bug-triage-Claude-Code-Game-Studios-main) | Read all open bugs in production/qa/bugs/, re-evaluate priority vs. severity, assign to sprints, surface syste | `Claude-Code-Game-Studios-main` |
| [bug-triage-Untrivial-ai__agent-orchestrator](./bug-triage-Untrivial-ai__agent-orchestrator) | Help clarify human bug reports, search for duplicates, and gather diagnostic evidence separately from a concis | `Untrivial-ai__agent-orchestrator` |
| [bugs](./bugs) | Proactively sweep a Pake area for latent UX and runtime defects before users report them, using this repo's ow | `Pake` |
| [build-cs-skill](./build-cs-skill) | CodeStable skill authoring and evolution protocol. Use when creating, refactoring, simplifying, or reviewing c | `CodeStable` |
| [build-mcp-app](./build-mcp-app) | This skill should be used when the user wants to build an "MCP app", add "interactive UI" or "widgets" to an M | `claude-plugins-official-main` |
| [build-mcp-server](./build-mcp-server) | This skill should be used when the user asks to "build an MCP server", "create an MCP", "make an MCP integrati | `claude-plugins-official-main` |
| [build-mcpb](./build-mcpb) | This skill should be used when the user wants to "package an MCP server", "bundle an MCP", "make an MCPB", "sh | `claude-plugins-official-main` |
| [build-personal-skill](./build-personal-skill) | Evidence-based creation of a reusable personal course-making Skill from the user's own classroom and chat hist | `OpenMAIC` |
| [building-agents](./building-agents) | Use when building or restructuring an LLM agent — provider adapter, tool calling, structured output, RAG, agen | `ericrisco__rsc-harness` |
| [bulk-ingestion](./bulk-ingestion) | \| | `gbrain` |
| [bulk-rnaseq](./bulk-rnaseq) | End-to-end bulk RNA-seq orchestrator — takes raw FASTQ reads through QC and trimming (FastQC, fastp/Trim Galor | `scientific-agent-skills` |
| [bun-runtime](./bun-runtime) | Bun as runtime, package manager, bundler, and test runner. When to choose Bun vs Node, migration notes, and Ve | `everything-claude-code-main` |
| [business-intelligence](./business-intelligence) | Use when a metric (revenue, MRR, margin) needs defining once in a governed semantic layer so every dashboard,  | `ericrisco__rsc-harness` |
| [bzdesignprompt](./bzdesignprompt) | 为前端或网页设计任务选择并下载合适的 DESIGN.md 模板。当用户需要网页设计模板、UI 风格参考、设计系统、落地页或前端界面设计时，浏览长亭百智云 UI 设计模板库，根据产品场景和视觉偏好匹配模板，并将完整 DES | `MonkeyCode` |
| [ca-btw](./ca-btw) | Lightweight Q&A about the project — answer from context and return, no routing, no state change. | `arbiterForge__codeArbiter` |
| [ca-btw-arbiterForge__codeArbiter](./ca-btw-arbiterForge__codeArbiter) | Lightweight Q&A about the project — answer from context and return, no routing, no state change. | `arbiterForge__codeArbiter` |
| [ca-new-skill](./ca-new-skill) | Author a new codeArbiter skill: prove the gap is real, get the spec approved, then write it. | `arbiterForge__codeArbiter` |
| [ca-new-skill-arbiterForge__codeArbiter](./ca-new-skill-arbiterForge__codeArbiter) | Author a new codeArbiter skill: prove the gap is real, get the spec approved, then write it. | `arbiterForge__codeArbiter` |
| [ca-sprint](./ca-sprint) | Autonomous sprint — one interactive spec gate, then plan-to-PR execution with every auto-decision SMARTS-score | `arbiterForge__codeArbiter` |
| [ca-sprint-arbiterForge__codeArbiter](./ca-sprint-arbiterForge__codeArbiter) | Autonomous sprint — one interactive spec gate, then plan-to-PR execution with every auto-decision SMARTS-score | `arbiterForge__codeArbiter` |
| [cache-efficiency](./cache-efficiency) | > | `Claude-Code-Agent-Monitor` |
| [cal-com-automation](./cal-com-automation) | Automate Cal.com tasks via Rube MCP (Composio): manage bookings, check availability, configure webhooks, and h | `agentic-awesome-skills` |
| [cal-com-automation-antigravity-awesome-skills-main](./cal-com-automation-antigravity-awesome-skills-main) | Automate Cal.com tasks via Rube MCP (Composio): manage bookings, check availability, configure webhooks, and h | `antigravity-awesome-skills-main` |
| [calendly-automation](./calendly-automation) | Automate Calendly scheduling, event management, invitee tracking, availability checks, and organization admini | `agentic-awesome-skills` |
| [calendly-automation-antigravity-awesome-skills-main](./calendly-automation-antigravity-awesome-skills-main) | Automate Calendly scheduling, event management, invitee tracking, availability checks, and organization admini | `antigravity-awesome-skills-main` |
| [canary-watch](./canary-watch) | Use this skill to monitor a deployed URL for regressions after deploys, merges, or dependency upgrades. | `everything-claude-code-main` |
| [canva-automation](./canva-automation) | Automate Canva tasks via Rube MCP (Composio): designs, exports, folders, brand templates, autofill. Always sea | `agentic-awesome-skills` |
| [canva-automation-antigravity-awesome-skills-main](./canva-automation-antigravity-awesome-skills-main) | Automate Canva tasks via Rube MCP (Composio): designs, exports, folders, brand templates, autofill. Always sea | `antigravity-awesome-skills-main` |
| [carrier-relationship-management](./carrier-relationship-management) | Codified expertise for managing carrier portfolios, negotiating freight rates, tracking carrier performance, a | `agentic-awesome-skills` |
| [carrier-relationship-management-antigravity-awesome-skills-main](./carrier-relationship-management-antigravity-awesome-skills-main) | Codified expertise for managing carrier portfolios, negotiating freight rates, tracking carrier performance, a | `antigravity-awesome-skills-main` |
| [carrier-relationship-management-ECC](./carrier-relationship-management-ECC) | > | `ECC` |
| [carrier-relationship-management-everything-claude-code-main](./carrier-relationship-management-everything-claude-code-main) | > | `everything-claude-code-main` |
| [cas-supervisor](./cas-supervisor) | Factory supervisor guide for multi-agent EPIC orchestration. Use when acting as supervisor to plan EPICs, spaw | `cas` |
| [cas-worker](./cas-worker) | Factory worker guide for task execution in CAS multi-agent sessions. Use when acting as a worker to execute as | `cas` |
| [cate-cli](./cate-cli) | Drive Cate browser, terminal, editor, panel, review, and coding-agent orchestration surfaces from a Cate termi | `cate` |
| [cavecrew](./cavecrew) | > | `caveman` |
| [caveman-commit](./caveman-commit) | > | `caveman` |
| [caveman-commit-caveman-main](./caveman-commit-caveman-main) | > | `caveman-main` |
| [caveman-learn](./caveman-learn) | Act on a Caveman learn report - review the ranked token sinks, apply cost-lowering fixes with per-edit consent | `caveman` |
| [caveman-stats](./caveman-stats) | > | `caveman` |
| [cc-skill-continuous-learning](./cc-skill-continuous-learning) | Development skill from everything-claude-code | `antigravity-awesome-skills-main` |
| [cc-skill-strategic-compact](./cc-skill-strategic-compact) | Development skill from everything-claude-code | `antigravity-awesome-skills-main` |
| [ce-optimize](./ce-optimize) | Optimize a named target with a measured loop: attribute a workload's cost, or score variants and keep winners. | `EveryInc__compound-engineering-plugin` |
| [ce-proof](./ce-proof) | Publish, read, comment on, or edit markdown in Proof. Use for Proof links, sharing specs/plans/drafts, or publ | `EveryInc__compound-engineering-plugin` |
| [ce-resolve-pr-feedback](./ce-resolve-pr-feedback) | Resolve PR review feedback. Use when addressing feedback already left on a PR. Not for reviewing the code befo | `EveryInc__compound-engineering-plugin` |
| [ce-skill-work](./ce-skill-work) | Applies this repository's skill-authoring standard as a procedure. Use for any change to, or judgment about, a | `EveryInc__compound-engineering-plugin` |
| [ce-strategy](./ce-strategy) | Create or update STRATEGY.md. Use when starting a product, adding a strategy doc, or changing direction or roa | `EveryInc__compound-engineering-plugin` |
| [cellxgene-census](./cellxgene-census) | Query the CZ CELLxGENE Census programmatically for versioned public single-cell and spatial transcriptomics da | `scientific-agent-skills` |
| [changelog](./changelog) | Auto-generates a changelog from git commits, sprint data, and design documents. Produces both internal and pla | `Claude-Code-Game-Studios-main` |
| [check-identity-pack](./check-identity-pack) | Run an AFP 100-point or AUSTRAC safe-harbour identity check over a set of documents, and report exactly what's | `agentic-awesome-skills` |
| [chrome-cdp](./chrome-cdp) | \| | `glimpse` |
| [chrome-devtools](./chrome-devtools) | Uses Chrome DevTools via MCP for efficient debugging, troubleshooting and browser automation. Use when debuggi | `chrome-devtools-mcp` |
| [ci-cd-and-automation](./ci-cd-and-automation) | Automates CI/CD pipeline setup. Use when setting up or modifying build and deployment pipelines. Use when you  | `agentic-awesome-skills` |
| [circleci-automation](./circleci-automation) | Automate CircleCI tasks via Rube MCP (Composio): trigger pipelines, monitor workflows/jobs, retrieve artifacts | `agentic-awesome-skills` |
| [circleci-automation-antigravity-awesome-skills-main](./circleci-automation-antigravity-awesome-skills-main) | Automate CircleCI tasks via Rube MCP (Composio): trigger pipelines, monitor workflows/jobs, retrieve artifacts | `antigravity-awesome-skills-main` |
| [cirq](./cirq) | Cirq is Google Quantum AI's open-source framework for designing, simulating, and running quantum circuits on q | `agentic-awesome-skills` |
| [cirq-antigravity-awesome-skills-main](./cirq-antigravity-awesome-skills-main) | Cirq is Google Quantum AI's open-source framework for designing, simulating, and running quantum circuits on q | `antigravity-awesome-skills-main` |
| [cirq-scientific-agent-skills](./cirq-scientific-agent-skills) | Google quantum computing framework. Use when targeting Google Quantum AI hardware, designing noise-aware circu | `scientific-agent-skills` |
| [citation-management](./citation-management) | Manage citations systematically throughout the research and writing process. | `agentic-awesome-skills` |
| [citation-management-antigravity-awesome-skills-main](./citation-management-antigravity-awesome-skills-main) | Manage citations systematically throughout the research and writing process. | `antigravity-awesome-skills-main` |
| [citation-management-scientific-agent-skills](./citation-management-scientific-agent-skills) | Comprehensive citation management for academic research. Search OpenAlex, PubMed, and Google Scholar for paper | `scientific-agent-skills` |
| [ck](./ck) | Persistent per-project memory for Claude Code. Auto-loads project context on session start, tracks sessions wi | `ECC` |
| [ck-everything-claude-code-main](./ck-everything-claude-code-main) | Persistent per-project memory for Claude Code. Auto-loads project context on session start, tracks sessions wi | `everything-claude-code-main` |
| [ckw-design](./ckw-design) | Frontend design entry point: direction, design system, visual philosophy. Use whenever building or touching th | `agentic-awesome-skills` |
| [claimable-postgres](./claimable-postgres) | Provision instant temporary Postgres databases via Claimable Postgres by Neon (neon.new) with no login, signup | `agentic-awesome-skills` |
| [claimable-postgres-antigravity-awesome-skills-main](./claimable-postgres-antigravity-awesome-skills-main) | Provision instant temporary Postgres databases via Claimable Postgres by Neon (pg.new). No login or credit car | `antigravity-awesome-skills-main` |
| [claims](./claims) | > | `ruflo` |
| [claims-ruflo-main](./claims-ruflo-main) | > | `ruflo-main` |
| [claims-ruvnet__ruflo](./claims-ruvnet__ruflo) | > | `ruvnet__ruflo` |
| [clarity-gate](./clarity-gate) | > | `agentic-awesome-skills` |
| [clarity-gate-antigravity-awesome-skills-main](./clarity-gate-antigravity-awesome-skills-main) | > | `antigravity-awesome-skills-main` |
| [clarvia-aeo-check](./clarvia-aeo-check) | Score any MCP server, API, or CLI for agent-readiness using Clarvia AEO (Agent Experience Optimization). Searc | `agentic-awesome-skills` |
| [clarvia-aeo-check-antigravity-awesome-skills-main](./clarvia-aeo-check-antigravity-awesome-skills-main) | Score any MCP server, API, or CLI for agent-readiness using Clarvia AEO (Agent Experience Optimization). Searc | `antigravity-awesome-skills-main` |
| [claude](./claude) | Use Claude Code as an independent `claude -p` subagent when the user explicitly asks for Claude, wants a secon | `skills` |
| [claude-api](./claude-api) | Build apps with the Claude API or Anthropic SDK. TRIGGER when: code imports `anthropic`/`@anthropic-ai/sdk`/`c | `agentic-awesome-skills` |
| [claude-api-antigravity-awesome-skills-main](./claude-api-antigravity-awesome-skills-main) | Build apps with the Claude API or Anthropic SDK. TRIGGER when: code imports `anthropic`/`@anthropic-ai/sdk`/`c | `antigravity-awesome-skills-main` |
| [claude-api-everything-claude-code-main](./claude-api-everything-claude-code-main) | Anthropic Claude API patterns for Python and TypeScript. Covers Messages API, streaming, tool use, vision, ext | `everything-claude-code-main` |
| [claude-api-everything-claude-code-main](./claude-api-everything-claude-code-main) | Anthropic Claude API patterns for Python and TypeScript. Covers Messages API, streaming, tool use, vision, ext | `everything-claude-code-main` |
| [claude-automation-recommender](./claude-automation-recommender) | Analyze a codebase and recommend Claude Code automations (hooks, subagents, skills, plugins, MCP servers). Use | `claude-plugins-official-main` |
| [claude-buddy](./claude-buddy) | Coordinate with the Claude Buddy companion (Claude Desktop + Claude Code on the user's Mac) — voice-approve to | `autonomous-os` |
| [claude-code](./claude-code) | Delegate coding to Claude Code CLI (features, PRs). | `hermes-agent` |
| [claude-code-expert](./claude-code-expert) | Especialista profundo em Claude Code - CLI da Anthropic. Maximiza produtividade com atalhos, hooks, MCPs, conf | `antigravity-awesome-skills-main` |
| [claude-code-guide](./claude-code-guide) | To provide a comprehensive reference for configuring and using Claude Code (the agentic coding tool) to its fu | `agentic-awesome-skills` |
| [claude-code-guide-antigravity-awesome-skills-main](./claude-code-guide-antigravity-awesome-skills-main) | To provide a comprehensive reference for configuring and using Claude Code (the agentic coding tool) to its fu | `antigravity-awesome-skills-main` |
| [claude-code-hermes-agent-main](./claude-code-hermes-agent-main) | Delegate coding tasks to Claude Code (Anthropic's CLI agent). Use for building features, refactoring, PR revie | `hermes-agent-main` |
| [claude-code-provider](./claude-code-provider) | Configure or troubleshoot BB-specific Claude Code provider settings and session behavior. | `bb` |
| [claude-code-review](./claude-code-review) | Run a general read-only implementation review with Claude Code as a native background subagent from Claude Cod | `proqi` |
| [claude-delegate](./claude-delegate) | Delegate coding tasks to a separate Claude Code CLI process or Claude | `agentic-awesome-skills` |
| [claude-design](./claude-design) | Design one-off HTML artifacts (landing, deck, prototype). | `hermes-agent` |
| [claude-devfleet](./claude-devfleet) | Orchestrate multi-agent coding tasks via Claude DevFleet — plan projects, dispatch parallel agents in isolated | `ECC` |
| [claude-devfleet-everything-claude-code-main](./claude-devfleet-everything-claude-code-main) | Orchestrate multi-agent coding tasks via Claude DevFleet — plan projects, dispatch parallel agents in isolated | `everything-claude-code-main` |
| [claude-md-doctor](./claude-md-doctor) | Give this repo's CLAUDE.md / AGENTS.md a checkup — size vitals vs official guidance, dead references, dead com | `claude-md-doctor` |
| [claude-md-improver](./claude-md-improver) | Audit and improve CLAUDE.md files in repositories. Use when user asks to check, audit, update, improve, or fix | `claude-plugins-official-main` |
| [claude-md-improver-TORCH](./claude-md-improver-TORCH) | OFFLINE FALLBACK for the claude-md-management plugin - prefer that plugin when it is installed. Audit and impr | `TORCH` |
| [claude-md-review](./claude-md-review) | Audit a CLAUDE.md file for the patterns that actually degrade Claude Code's output — vagueness, unnamed files, | `Claude-Code-Everything-You-Need-to-Know` |
| [claude-monitor](./claude-monitor) | Monitor de performance do Claude Code e sistema local. Diagnostica lentidao, mede CPU/RAM/disco, verifica API  | `agentic-awesome-skills` |
| [claude-monitor-antigravity-awesome-skills-main](./claude-monitor-antigravity-awesome-skills-main) | Monitor de performance do Claude Code e sistema local. Diagnostica lentidao, mede CPU/RAM/disco, verifica API  | `antigravity-awesome-skills-main` |
| [claude-settings-audit](./claude-settings-audit) | Analyze a repository to generate recommended Claude Code settings.json permissions. Use when setting up a new  | `antigravity-awesome-skills-main` |
| [clawbio](./clawbio) | Use ClawBio for local-first bioinformatics agent workflows: | `AREX-Skill` |
| [clawteam-dev](./clawteam-dev) | > | `ClawTeam-OpenClaw` |
| [clean-code-guard](./clean-code-guard) | Review generated or changed production code with Clean Code, SOLID, DRY, KISS, YAGNI, and LLM-specific failure | `agentic-awesome-skills` |
| [cli-anything-browser](./cli-anything-browser) | Browser automation CLI using DOMShell MCP server. Maps Chrome's Accessibility Tree to a virtual filesystem for | `CLI-Anything` |
| [cli-anything-ccswitch](./cli-anything-ccswitch) | CLI interface for CC Switch — manage AI coding tool configurations from the terminal | `CLI-Anything` |
| [cli-anything-dify-workflow](./cli-anything-dify-workflow) | Wrapper for the Dify workflow DSL CLI. Create, inspect, validate, edit, and export Dify workflow files through | `CLI-Anything` |
| [cli-anything-quietshrink](./cli-anything-quietshrink) | Compress macOS screen recordings with zero CPU stress using Apple Silicon's hardware HEVC encoder. Typically r | `CLI-Anything` |
| [cli-main](./cli-main) | Use mmx to generate text, images, video, speech, and music via the MiniMax AI platform. Use when the user want | `cli-main` |
| [cli-reference](./cli-reference) | Claude Code CLI commands, flags, headless mode, and automation patterns | `Continuous-Claude-v3-main` |
| [cli-resilience](./cli-resilience) | Inspect and manage circuit-breaker states, connection cooldowns, quota limits, and backoff levels from the CLI | `diegosouzapw__OmniRoute` |
| [cli-resilience-OmniRoute](./cli-resilience-OmniRoute) | Inspect and manage circuit-breaker states, connection cooldowns, quota limits, and backoff levels from the CLI | `OmniRoute` |
| [click-path-audit](./click-path-audit) | Trace every user-facing button/touchpoint through its full state change sequence to find bugs where functions  | `everything-claude-code-main` |
| [clickhouse-io](./clickhouse-io) | ClickHouse database patterns, query optimization, analytics, and data engineering best practices for high-perf | `everything-claude-code-main` |
| [clickup-automation](./clickup-automation) | Automate ClickUp project management including tasks, spaces, folders, lists, comments, and team operations via | `agentic-awesome-skills` |
| [clickup-automation-antigravity-awesome-skills-main](./clickup-automation-antigravity-awesome-skills-main) | Automate ClickUp project management including tasks, spaces, folders, lists, comments, and team operations via | `antigravity-awesome-skills-main` |
| [cline-delegate](./cline-delegate) | Delegate coding tasks to the Cline CLI (`cline`) only when the user explicitly | `agentic-awesome-skills` |
| [cline-sdk](./cline-sdk) | Comprehensive Cline SDK skill for building AI agents. Covers the Agent runtime, ClineCore sessions, custom too | `cline` |
| [clinical-decision-support](./clinical-decision-support) | Prepare and validate research-only clinical decision-support evaluation, evidence-profile, cohort, survival, b | `scientific-agent-skills` |
| [clone](./clone) | Clone the current conversation so the user can branch off and try a different approach. | `claude-code-tips-main` |
| [cmux-cloud-vm](./cmux-cloud-vm) | Route work to cmux Cloud machines (persistent cloud VMs) from the CLI — `cmux vm route`/`run`/`agent` pick a m | `cmux` |
| [cmux-cloud-vm-manaflow-ai__cmux](./cmux-cloud-vm-manaflow-ai__cmux) | Route work to cmux Cloud machines from the plain `cmux vm` CLI (alias `cmux cloud`): route/run/agent pick a ma | `manaflow-ai__cmux` |
| [cmux-cua](./cmux-cua) | Use only after the user explicitly asks for Computer Use: drive real macOS apps from a cmux agent session via  | `cmux` |
| [coda-automation](./coda-automation) | Automate Coda tasks via Rube MCP (Composio): manage docs, pages, tables, rows, formulas, permissions, and publ | `agentic-awesome-skills` |
| [coda-automation-antigravity-awesome-skills-main](./coda-automation-antigravity-awesome-skills-main) | Automate Coda tasks via Rube MCP (Composio): manage docs, pages, tables, rows, formulas, permissions, and publ | `antigravity-awesome-skills-main` |
| [code-improver](./code-improver) | Runs an autonomous review-and-fix improvement loop over any code target — a skill, plugin, module, or director | `trailofbits__skills` |
| [code-review](./code-review) | Performs an architectural and quality code review on a specified file or set of files. Checks for coding stand | `Claude-Code-Game-Studios-main` |
| [code-review-and-quality](./code-review-and-quality) | Conducts multi-axis code review. Use before merging any change. Use when reviewing code written by yourself, a | `agentic-awesome-skills` |
| [code-review-ericrisco__rsc-harness](./code-review-ericrisco__rsc-harness) | Use to judge a concrete diff, branch, or GitHub PR on its own merits with no rsc-SDD spec/plan chain to key of | `ericrisco__rsc-harness` |
| [code-review-Kangentic__kangentic](./code-review-Kangentic__kangentic) | Review git changes for quality and conventions via parallel reviewer subagents synthesized in the main agent ( | `Kangentic__kangentic` |
| [code-review-pipecat](./code-review-pipecat) | Automated code review for pull requests using multiple specialized agents | `pipecat` |
| [code-showcase-core-components](./code-showcase-core-components) | Core component library and design system patterns. Use when building UI, using design tokens, or working with  | `agentic-awesome-skills` |
| [code-showcase-react-ui-patterns](./code-showcase-react-ui-patterns) | Modern React UI patterns for loading states, error handling, and data fetching. Use when building UI component | `agentic-awesome-skills` |
| [code-showcase-systematic-debugging](./code-showcase-systematic-debugging) | Four-phase debugging methodology with root cause analysis. Use when investigating bugs, fixing test failures,  | `agentic-awesome-skills` |
| [code-showcase-testing-patterns](./code-showcase-testing-patterns) | Jest testing patterns, factory functions, mocking strategies, and TDD workflow. Use when writing unit tests, c | `agentic-awesome-skills` |
| [code-simplification](./code-simplification) | Simplifies code for clarity. Use when refactoring code for clarity without changing behavior. Use when code wo | `agentic-awesome-skills` |
| [codebase-design](./codebase-design) | Shared vocabulary for designing deep modules. Use when the user wants to design or improve a module's interfac | `agentic-awesome-skills` |
| [codebase-inspection](./codebase-inspection) | Inspect codebases w/ pygount: LOC, languages, ratios. | `hermes-agent` |
| [codehealth-mcp](./codehealth-mcp) | Real-time structural Code Health via CodeScene MCP — review before edits, verify score deltas after changes, g | `ECC` |
| [codex](./codex) | Delegate coding to OpenAI Codex CLI (features, PRs). | `hermes-agent` |
| [codex-delegate](./codex-delegate) | Delegate coding tasks to the OpenAI Codex CLI only when the user explicitly | `agentic-awesome-skills` |
| [codex-hermes-agent-main](./codex-hermes-agent-main) | Delegate coding tasks to OpenAI Codex CLI agent. Use for building features, refactoring, PR reviews, and batch | `hermes-agent-main` |
| [codex-profiles](./codex-profiles) | Use codex-profiles to run Codex CLI or Codex Desktop with isolated CODEX_HOME profiles for separate accounts,  | `agentic-awesome-skills` |
| [codex-review](./codex-review) | Professional code review with auto CHANGELOG generation, integrated with Codex AI. Use when you want professio | `agentic-awesome-skills` |
| [codex-review-antigravity-awesome-skills-main](./codex-review-antigravity-awesome-skills-main) | Professional code review with auto CHANGELOG generation, integrated with Codex AI. Use when you want professio | `antigravity-awesome-skills-main` |
| [codex-review-proqi](./codex-review-proqi) | Run a bounded read-only implementation review with Codex as a native background subagent from Codex, or throug | `proqi` |
| [codex-skills](./codex-skills) | Use the local Codex CLI as an independent second agent. Two branches — (1) proactively run `codex review` for  | `skills` |
| [coding-agent](./coding-agent) | Delegate coding work to Codex, Claude Code, or OpenCode as background workers; not simple edits or read-only c | `openclaw` |
| [coding-agent-standards](./coding-agent-standards) | Defines practical standards for implementation-focused coding agents. Use when creating or editing code, espec | `claude-code-prompts-master` |
| [coding-agent-understudy-ai__understudy](./coding-agent-understudy-ai__understudy) | Delegate coding tasks to Codex, Claude Code, or Pi agents via background process. Use when: (1) building/creat | `understudy-ai__understudy` |
| [coding-standards](./coding-standards) | Baseline cross-project coding conventions for naming, readability, immutability, and code-quality review. Use  | `everything-claude-code-main` |
| [comfyui-gateway](./comfyui-gateway) | REST API gateway for ComfyUI servers. Workflow management, job queuing, webhooks, caching, auth, rate limiting | `agentic-awesome-skills` |
| [comfyui-gateway-antigravity-awesome-skills-main](./comfyui-gateway-antigravity-awesome-skills-main) | REST API gateway for ComfyUI servers. Workflow management, job queuing, webhooks, caching, auth, rate limiting | `antigravity-awesome-skills-main` |
| [command-development](./command-development) | This skill should be used when the user asks to "create a slash command", "add a command", "write a custom com | `claude-plugins-official-main` |
| [commandcode-delegate](./commandcode-delegate) | Delegate coding tasks to the Command Code CLI (`cmd`) only when the user | `agentic-awesome-skills` |
| [commerce-architecture](./commerce-architecture) | How the reference commerce agents of either role are structured, covering the loop, where each rule lives, ski | `commerce-agents` |
| [commerce-merchant-operations](./commerce-merchant-operations) | The reference merchant agent, covering its flows, staged changes and host approval, metrics grounding, the ana | `commerce-agents` |
| [compact-guard](./compact-guard) | Smart context compaction with state preservation. Saves critical files, task progress, and working state befor | `pro-workflow` |
| [compact-now](./compact-now) | Operator-triggered proactive compaction — Chrono externalizes load-bearing state (active decisions, open tasks | `claude-vibe-squad` |
| [competitor-analysis](./competitor-analysis) | Research competitors with Browserbase discovery, enrichment lanes, screenshots, matrices, and HTML reports. | `agentic-awesome-skills` |
| [competitor-news-monitor](./competitor-news-monitor) | Watch named companies for material news; cited digests. | `hermes-agent` |
| [complexity-cuts](./complexity-cuts) | Lower Big-O on existing code via a one-transformation-at-a-time playbook with verify-revert-stop. For new code | `agentic-awesome-skills` |
| [compose-multiplatform-patterns](./compose-multiplatform-patterns) | Compose Multiplatform and Jetpack Compose patterns for KMP projects — state management, navigation, theming, p | `everything-claude-code-main` |
| [composition-patterns](./composition-patterns) | Use when working with composition-patterns tasks or workflows | `agentic-awesome-skills` |
| [computer-use](./computer-use) | Drive the desktop background-first; escalate on signal. | `hermes-agent` |
| [concurrency-report](./concurrency-report) | > | `Claude-Code-Agent-Monitor` |
| [config-audit](./config-audit) | > | `Claude-Code-Agent-Monitor` |
| [configure-ecc](./configure-ecc) | Interactive installer for Everything Claude Code — guides users through selecting and installing skills and ru | `everything-claude-code-main` |
| [confluence-automation](./confluence-automation) | Automate Confluence page creation, content search, space management, labels, and hierarchy navigation via Rube | `agentic-awesome-skills` |
| [confluence-automation-antigravity-awesome-skills-main](./confluence-automation-antigravity-awesome-skills-main) | Automate Confluence page creation, content search, space management, labels, and hierarchy navigation via Rube | `antigravity-awesome-skills-main` |
| [consciousness-council](./consciousness-council) | Run a multi-perspective Mind Council deliberation on any question, decision, or creative challenge. Use this s | `scientific-agent-skills` |
| [consistency-check](./consistency-check) | Scan all GDDs against the entity registry to detect cross-document inconsistencies: same entity with different | `Claude-Code-Game-Studios-main` |
| [content-audit](./content-audit) | Audit GDD-specified content counts against implemented content. Identifies what's planned vs built. | `Claude-Code-Game-Studios-main` |
| [content-hash-cache-pattern](./content-hash-cache-pattern) | Cache expensive file processing results using SHA-256 content hashes — path-independent, auto-invalidating, wi | `everything-claude-code-main` |
| [context-agent](./context-agent) | Agente de contexto para continuidade entre sessoes. Salva resumos, decisoes, tarefas pendentes e carrega brief | `antigravity-awesome-skills-main` |
| [context-budget](./context-budget) | Audits Claude Code context window consumption across agents, skills, MCP servers, and rules. Identifies bloat, | `ECC` |
| [context-budget-everything-claude-code-main](./context-budget-everything-claude-code-main) | Audits Claude Code context window consumption across agents, skills, MCP servers, and rules. Identifies bloat, | `everything-claude-code-main` |
| [context-compression](./context-compression) | This skill should be used when the user asks to "compress context", "summarize conversation history", "impleme | `Agent-Skills-for-Context-Engineering-main` |
| [context-degradation](./context-degradation) | This skill should be used when the user asks to "diagnose context problems", "fix lost-in-middle issues", "deb | `Agent-Skills-for-Context-Engineering-main` |
| [context-engineering](./context-engineering) | Optimizes agent context setup. Use when starting a new session, when agent output quality degrades, when switc | `agentic-awesome-skills` |
| [context-fundamentals](./context-fundamentals) | This skill should be used when the user asks to "understand context", "explain context windows", "design agent | `Agent-Skills-for-Context-Engineering-main` |
| [context-guardian](./context-guardian) | Guardiao de contexto que preserva dados criticos antes da compactacao automatica. Snapshots, verificacao de in | `agentic-awesome-skills` |
| [context-guardian-antigravity-awesome-skills-main](./context-guardian-antigravity-awesome-skills-main) | Guardiao de contexto que preserva dados criticos antes da compactacao automatica. Snapshots, verificacao de in | `antigravity-awesome-skills-main` |
| [context-kit](./context-kit) | Evaluate, adapt, and safely install Context Kit personal context artifacts for Claude Code or adjacent agent w | `agentic-awesome-skills` |
| [context-management-context-save](./context-management-context-save) | Use when working with context management context save | `agentic-awesome-skills` |
| [context-management-context-save-antigravity-awesome-skills-main](./context-management-context-save-antigravity-awesome-skills-main) | Use when working with context management context save | `antigravity-awesome-skills-main` |
| [context-manager](./context-manager) | Elite AI context engineering specialist mastering dynamic context management, vector databases, knowledge grap | `agentic-awesome-skills` |
| [context-manager-antigravity-awesome-skills-main](./context-manager-antigravity-awesome-skills-main) | Elite AI context engineering specialist mastering dynamic context management, vector databases, knowledge grap | `antigravity-awesome-skills-main` |
| [context-mode-ops](./context-mode-ops) | Manage context-mode GitHub issues, PRs, releases, and marketing with parallel subagent army. Orchestrates 10-2 | `context-mode` |
| [context-mode-ops-context-mode-main](./context-mode-ops-context-mode-main) | Manage context-mode GitHub issues, PRs, releases, and marketing with parallel subagent army. Orchestrates 10-2 | `context-mode-main` |
| [context-optimization](./context-optimization) | This skill should be used when the user asks to "optimize context", "reduce token costs", "improve context eff | `Agent-Skills-for-Context-Engineering-main` |
| [context-optimizer](./context-optimizer) | Optimize token usage and context management. Use when sessions feel slow, context is degraded, or you're runni | `pro-workflow` |
| [context7-auto-research](./context7-auto-research) | Automatically fetch latest library/framework documentation for Claude Code via Context7 API. Use when you need | `agentic-awesome-skills` |
| [context7-auto-research-antigravity-awesome-skills-main](./context7-auto-research-antigravity-awesome-skills-main) | Automatically fetch latest library/framework documentation for Claude Code via Context7 API. Use when you need | `antigravity-awesome-skills-main` |
| [continuous-agent-loop](./continuous-agent-loop) | Patterns for continuous autonomous agent loops with quality gates, evals, and recovery controls. | `everything-claude-code-main` |
| [continuous-learning](./continuous-learning) | [DEPRECATED - use continuous-learning-v2] Legacy v1 stop-hook skill extractor. v2 is a strict superset with in | `ECC` |
| [continuous-learning-everything-claude-code-main](./continuous-learning-everything-claude-code-main) | Automatically extract reusable patterns from Claude Code sessions and save them as learned skills for future u | `everything-claude-code-main` |
| [continuous-learning-v2](./continuous-learning-v2) | Instinct-based learning system that observes sessions via hooks, creates atomic instincts with confidence scor | `ECC` |
| [continuous-learning-v2-everything-claude-code-main](./continuous-learning-v2-everything-claude-code-main) | Instinct-based learning system that observes sessions via hooks, creates atomic instincts with confidence scor | `everything-claude-code-main` |
| [contributing](./contributing) | Contribute changes to the Feynman repository itself. Use when the task is to add features, fix bugs, update pr | `feynman-main` |
| [control-centre](./control-centre) | > | `vnx-orchestration` |
| [convertkit-automation](./convertkit-automation) | Automate ConvertKit (Kit) tasks via Rube MCP (Composio): manage subscribers, tags, broadcasts, and broadcast s | `agentic-awesome-skills` |
| [convertkit-automation-antigravity-awesome-skills-main](./convertkit-automation-antigravity-awesome-skills-main) | Automate ConvertKit (Kit) tasks via Rube MCP (Composio): manage subscribers, tags, broadcasts, and broadcast s | `antigravity-awesome-skills-main` |
| [convex](./convex) | Convex is the backend agents get right on the first try: an all-TypeScript reactive platform where the databas | `openclaw__clawhub` |
| [copilot-delegate](./copilot-delegate) | Delegate coding tasks to the GitHub Copilot CLI (`copilot`) only when | `agentic-awesome-skills` |
| [copilot-sdk](./copilot-sdk) | Build applications that programmatically interact with GitHub Copilot. The SDK wraps the Copilot CLI via JSON- | `agentic-awesome-skills` |
| [copilot-sdk-antigravity-awesome-skills-main](./copilot-sdk-antigravity-awesome-skills-main) | Build applications that programmatically interact with GitHub Copilot. The SDK wraps the Copilot CLI via JSON- | `antigravity-awesome-skills-main` |
| [copilotkit-agui](./copilotkit-agui) | Use when building custom agent backends, implementing the AG-UI protocol, debugging streaming issues, or under | `copilotkit` |
| [copilotkit-contribute](./copilotkit-contribute) | > | `copilotkit` |
| [copilotkit-develop](./copilotkit-develop) | Use when building AI-powered features with CopilotKit v2 -- adding chat interfaces, registering frontend tools | `copilotkit` |
| [copilotkit-integrations](./copilotkit-integrations) | Use when wiring an external agent framework (LangGraph, CrewAI, PydanticAI, Mastra, ADK, LlamaIndex, Agno, Str | `copilotkit` |
| [copilotkit-self-update](./copilotkit-self-update) | Use when the user wants to update, refresh, or reinstall the CopilotKit agent SKILLS (the SKILL.md files that  | `copilotkit` |
| [copilotkit-setup](./copilotkit-setup) | > | `copilotkit` |
| [cost](./cost) | >- | `Citadel` |
| [cost-alert](./cost-alert) | > | `Claude-Code-Agent-Monitor` |
| [cost-aware-llm-pipeline](./cost-aware-llm-pipeline) | Cost optimization patterns for LLM API usage — model routing by task complexity, budget tracking, retry logic, | `everything-claude-code-main` |
| [cost-breakdown](./cost-breakdown) | > | `Claude-Code-Agent-Monitor` |
| [cost-report](./cost-report) | > | `Claude-Code-Agent-Monitor` |
| [cost-track](./cost-track) | Auto-capture per-session token usage from the Claude Code session jsonl and persist to the cost-tracking names | `ruflo` |
| [cost-tracker](./cost-tracker) | Track session costs, set budget alerts, and optimize token spend. Use to check costs mid-session or set spendi | `pro-workflow` |
| [cost-tracking](./cost-tracking) | Track and report Claude Code token usage, spending, and budgets from the local ECC cost-tracker metrics log. U | `ECC` |
| [cotal-mesh](./cotal-mesh) | Put an AI agent on a Cotal mesh and coordinate with other agents across vendors and machines. Use when a user  | `Cotal-AI__Cotal` |
| [council](./council) | Convene a four-voice council for ambiguous decisions, tradeoffs, and go/no-go calls. Use when multiple valid p | `ECC` |
| [council-everything-claude-code-main](./council-everything-claude-code-main) | Convene a four-voice council for ambiguous decisions, tradeoffs, and go/no-go calls. Use when multiple valid p | `everything-claude-code-main` |
| [council-verdicts](./council-verdicts) | Starter: interpret a council run's verdict artifacts — quorum, dissent, cross-lab validity, and what to do nex | `claude-octopus` |
| [cowart-open-canvas](./cowart-open-canvas) | Open, reopen, or explicitly refresh the native Cowart canvas when the user asks to see it, or when a bare @Cow | `cowart` |
| [create-agent](./create-agent) | > | `openma-ai__open-managed-agents` |
| [create-architecture](./create-architecture) | Guided, section-by-section authoring of the master architecture document for the game. Reads all GDDs, the sys | `Claude-Code-Game-Studios-main` |
| [create-control-manifest](./create-control-manifest) | After architecture is complete, produces a flat actionable rules sheet for programmers — what you must do, wha | `Claude-Code-Game-Studios-main` |
| [create-epics](./create-epics) | Translate approved GDDs + architecture into epics — one epic per architectural module. Defines scope, governin | `Claude-Code-Game-Studios-main` |
| [create-kandev-plugin](./create-kandev-plugin) | Create, modify, debug, test, package, or publish a Kandev runtime plugin in its dedicated repository. Use only | `kandev` |
| [create-plugin](./create-plugin) | Scaffold a new Claude Code plugin with proper directory structure, plugin.json, skills, commands, and agents | `ruflo` |
| [create-skill](./create-skill) | >- | `Citadel` |
| [create-stories](./create-stories) | Break a single epic into implementable story files. Reads the epic, its GDD, governing ADRs, and control manif | `Claude-Code-Game-Studios-main` |
| [cred-omega](./cred-omega) | CISO operacional enterprise para gestao total de credenciais e segredos. | `agentic-awesome-skills` |
| [cred-omega-antigravity-awesome-skills-main](./cred-omega-antigravity-awesome-skills-main) | CISO operacional enterprise para gestao total de credenciais e segredos. | `antigravity-awesome-skills-main` |
| [crewai](./crewai) | Expert in CrewAI - the leading role-based multi-agent framework | `agentic-awesome-skills` |
| [crewai-antigravity-awesome-skills-main](./crewai-antigravity-awesome-skills-main) | Expert in CrewAI - the leading role-based multi-agent framework | `antigravity-awesome-skills-main` |
| [crewai-AREX-Skill](./crewai-AREX-Skill) | Routes agents using or contributing to CrewAI, including crews, | `AREX-Skill` |
| [cross-repo-testing](./cross-repo-testing) | This skill should be used when the user asks to "test a cross-repo feature", "deploy a feature branch to stagi | `OpenHands-main` |
| [crossframe](./crossframe) | Use when the user explicitly invokes CrossFrame or 跨尺度结构诊断 for Chinese-canonical structural diagnosis of compl | `agentic-awesome-skills` |
| [crossframe-casebook](./crossframe-casebook) | Use when CrossFrame Suite routes explicit Chinese casebook work: turning materials into reusable cases, anonym | `agentic-awesome-skills` |
| [crossframe-critical](./crossframe-critical) | Use only when the user explicitly names crossframe-critical for a Chinese structural critique dossier, article | `agentic-awesome-skills` |
| [crossframe-debate](./crossframe-debate) | Use when CrossFrame Suite routes explicit Chinese proposition testing, debate analysis, hidden-premise review, | `agentic-awesome-skills` |
| [crossframe-dialogue](./crossframe-dialogue) | Use when CrossFrame Suite routes explicit Chinese reader replies, editor responses, consultation-style short a | `agentic-awesome-skills` |
| [crossframe-essay](./crossframe-essay) | Use when explicit CrossFrame work needs a Chinese critical insight essay, commentary, concept essay, public pi | `agentic-awesome-skills` |
| [crossframe-notebook](./crossframe-notebook) | Use when CrossFrame Suite routes explicit Chinese notes for books, theories, articles, excerpts, bidirectional | `agentic-awesome-skills` |
| [crossframe-org](./crossframe-org) | Use when CrossFrame Suite routes explicit Chinese analysis of teams, projects, organizations, responsibility c | `agentic-awesome-skills` |
| [crossframe-public](./crossframe-public) | Use when CrossFrame Suite routes explicit Chinese analysis of public issues, platform governance, policy, inst | `agentic-awesome-skills` |
| [crossframe-review](./crossframe-review) | Use when explicit CrossFrame output needs review for reasoning fidelity, evidence boundaries, source anchors,  | `agentic-awesome-skills` |
| [crossframe-suite](./crossframe-suite) | Use when the user explicitly invokes CrossFrame Suite for Chinese structural diagnosis workflows across relati | `agentic-awesome-skills` |
| [crossframe-teach](./crossframe-teach) | Use when CrossFrame Suite routes explicit Chinese teaching of CrossFrame concepts, misreading boundaries, plai | `agentic-awesome-skills` |
| [csharp-testing](./csharp-testing) | C# and .NET testing patterns with xUnit, FluentAssertions, mocking, integration tests, and test organization b | `everything-claude-code-main` |
| [cucumber-skill](./cucumber-skill) | Generates Cucumber BDD tests with Gherkin feature files and step definitions in Java, JavaScript, or Ruby. Use | `agentic-awesome-skills` |
| [customer-billing-ops](./customer-billing-ops) | Operate customer billing workflows such as subscriptions, refunds, churn triage, billing-portal recovery, and  | `everything-claude-code-main` |
| [customs-trade-compliance](./customs-trade-compliance) | Codified expertise for customs documentation, tariff classification, duty optimisation, restricted party scree | `agentic-awesome-skills` |
| [customs-trade-compliance-antigravity-awesome-skills-main](./customs-trade-compliance-antigravity-awesome-skills-main) | Codified expertise for customs documentation, tariff classification, duty optimisation, restricted party scree | `antigravity-awesome-skills-main` |
| [customs-trade-compliance-ECC](./customs-trade-compliance-ECC) | > | `ECC` |
| [customs-trade-compliance-everything-claude-code-main](./customs-trade-compliance-everything-claude-code-main) | > | `everything-claude-code-main` |
| [cypress-skill](./cypress-skill) | Generates production-grade Cypress E2E and component tests in JavaScript or TypeScript. Supports local executi | `agentic-awesome-skills` |
| [daemon](./daemon) | >- | `Citadel` |
| [dag-map](./dag-map) | > | `Claude-Code-Agent-Monitor` |
| [daily-budget-check](./daily-budget-check) | > | `Claude-Code-Agent-Monitor` |
| [daily-news-report](./daily-news-report) | Scrapes content based on a preset URL list, filters high-quality technical information, and generates daily Ma | `agentic-awesome-skills` |
| [daily-news-report-antigravity-awesome-skills-main](./daily-news-report-antigravity-awesome-skills-main) | Scrapes content based on a preset URL list, filters high-quality technical information, and generates daily Ma | `antigravity-awesome-skills-main` |
| [daily-review](./daily-review) | A股每日复盘（专业版）— 基于 QuantDB 本地数据 + 当日新闻情绪 + 模型推理信号 + L1/L2 因子截面 + 板块资金流的盘后复盘：指数、涨跌结构与涨停梯队、量能、行业/概念轮动、资金面、L2 微观结构、当 | `QuantMind` |
| [daily-standup](./daily-standup) | > | `Claude-Code-Agent-Monitor` |
| [dart-flutter-patterns](./dart-flutter-patterns) | Production-ready Dart and Flutter patterns covering null safety, immutable state, async composition, widget ar | `everything-claude-code-main` |
| [dashboard](./dashboard) | Open OwnMem Console, the local dashboard for this repository's memory. Use when the user asks to open the dash | `ownmem` |
| [dashboard-builder](./dashboard-builder) | Build monitoring dashboards that answer real operator questions for Grafana, SigNoz, and similar platforms. Us | `everything-claude-code-main` |
| [dask](./dask) | Distributed computing for larger-than-RAM pandas/NumPy workflows. Use when you need to scale existing pandas/N | `scientific-agent-skills` |
| [data-export](./data-export) | > | `Claude-Code-Agent-Monitor` |
| [data-juicer](./data-juicer) | Data-Juicer repo router for local recipes, Ray recovery, and | `AREX-Skill` |
| [data-scraper-agent](./data-scraper-agent) | Build a fully automated AI-powered data collection agent for any public source — job boards, prices, news, Git | `everything-claude-code-main` |
| [database-lookup](./database-lookup) | Query documented public database APIs with explicit endpoints, filters, pagination, and provenance. Use when a | `scientific-agent-skills` |
| [database-migrations](./database-migrations) | Database migration best practices for schema changes, data migrations, rollbacks, and zero-downtime deployment | `everything-claude-code-main` |
| [datachain](./datachain) | Routes DataChain operating guidance for Python SDK pipelines, | `AREX-Skill` |
| [datadog-automation](./datadog-automation) | Automate Datadog tasks via Rube MCP (Composio): query metrics, search logs, manage monitors/dashboards, create | `agentic-awesome-skills` |
| [datadog-automation-antigravity-awesome-skills-main](./datadog-automation-antigravity-awesome-skills-main) | Automate Datadog tasks via Rube MCP (Composio): query metrics, search logs, manage monitors/dashboards, create | `antigravity-awesome-skills-main` |
| [datamol](./datamol) | Pythonic wrapper around RDKit with simplified interface and sensible defaults. Preferred for standard drug dis | `scientific-agent-skills` |
| [day-one-patch](./day-one-patch) | Prepare a day-one patch for a game launch. Scopes, prioritises, implements, and QA-gates a focused patch addre | `Claude-Code-Game-Studios-main` |
| [debug-hooks](./debug-hooks) | Systematic hook debugging workflow. Use when hooks aren't firing, producing wrong output, or behaving unexpect | `Continuous-Claude-v3-main` |
| [debugging-and-error-recovery](./debugging-and-error-recovery) | Guides systematic root-cause debugging. Use when tests fail, builds break, behavior doesn't match expectations | `agentic-awesome-skills` |
| [debugging-toolkit-smart-debug](./debugging-toolkit-smart-debug) | Use when working with debugging toolkit smart debug | `agentic-awesome-skills` |
| [debugging-toolkit-smart-debug-antigravity-awesome-skills-main](./debugging-toolkit-smart-debug-antigravity-awesome-skills-main) | Use when working with debugging toolkit smart debug | `antigravity-awesome-skills-main` |
| [decompose-gates](./decompose-gates) | Decompose a hard or multi-part task into independently checkable pieces with explicit verification gates and r | `happier` |
| [deep-research](./deep-research) | Multi-source deep research using firecrawl and exa MCPs. Searches the web, synthesizes findings, and delivers  | `everything-claude-code-main` |
| [deepchem](./deepchem) | Molecular ML with diverse featurizers and pre-built datasets. Use for property prediction (ADMET, toxicity) wi | `scientific-agent-skills` |
| [deepke](./deepke) | Route DeepKE knowledge extraction workflows across supervised | `AREX-Skill` |
| [deepspot-m](./deepspot-m) | Generate transcriptome-wide virtual spatial transcriptomics from H&E histology with DeepSpot-M. Use when you n | `scientific-agent-skills` |
| [deeptools](./deeptools) | NGS analysis toolkit. BAM to bigWig conversion, QC (correlation, PCA, fingerprints), heatmaps/profiles (TSS, p | `scientific-agent-skills` |
| [delegate-setup](./delegate-setup) | Configure approved delegation lanes across installed implementer CLIs, | `agentic-awesome-skills` |
| [delegation-audit](./delegation-audit) | > | `Claude-Code-Agent-Monitor` |
| [deploy-to-vercel](./deploy-to-vercel) | Deploy applications and websites to Vercel. Use when the user requests deployment actions like \"deploy my app | `agentic-awesome-skills` |
| [deprecation-and-migration](./deprecation-and-migration) | Manages deprecation and migration. Use when removing old systems, APIs, or features. Use when migrating users  | `agentic-awesome-skills` |
| [deserialize](./deserialize) | Insecure-deserialization playbook — fingerprint the language/format (Java serialized, .NET BinaryFormatter, Py | `agent` |
| [design-md](./design-md) | Analyze Stitch projects and synthesize a semantic design system into DESIGN.md files | `agentic-awesome-skills` |
| [design-md-antigravity-awesome-skills-main](./design-md-antigravity-awesome-skills-main) | Analyze Stitch projects and synthesize a semantic design system into DESIGN.md files | `antigravity-awesome-skills-main` |
| [design-md-hermes-agent](./design-md-hermes-agent) | Author/validate/export Google's DESIGN.md token spec files. | `hermes-agent` |
| [design-orchestration](./design-orchestration) | Orchestrates design workflows by routing work through brainstorming, multi-agent review, and execution readine | `agentic-awesome-skills` |
| [design-orchestration-antigravity-awesome-skills-main](./design-orchestration-antigravity-awesome-skills-main) | Orchestrates design workflows by routing work through brainstorming, multi-agent review, and execution readine | `antigravity-awesome-skills-main` |
| [design-review](./design-review) | Reviews a game design document for completeness, internal consistency, implementability, and adherence to proj | `Claude-Code-Game-Studios-main` |
| [design-spatial](./design-spatial) | Design — spatial composition | `agentic-awesome-skills` |
| [design-system](./design-system) | Guided, section-by-section GDD authoring for a single game system. Gathers context from existing docs, walks t | `Claude-Code-Game-Studios-main` |
| [design-system-everything-claude-code-main](./design-system-everything-claude-code-main) | Use this skill to generate or audit design systems, check visual consistency, and review PRs that touch stylin | `everything-claude-code-main` |
| [design-system-ui-ux-pro-max-skill](./design-system-ui-ux-pro-max-skill) | Token architecture, component specifications, and slide generation. Three-layer tokens (primitive→semantic→com | `ui-ux-pro-max-skill` |
| [destructive_command_guard](./destructive_command_guard) | Destructive Command Guard - High-performance Rust hook for Claude Code that blocks dangerous commands before e | `destructive_command_guard` |
| [detect-ai-text](./detect-ai-text) | Estimate whether a document's prose was written by AI, with the linguistic tells and honest abstention on non- | `agentic-awesome-skills` |
| [deterministic-design](./deterministic-design) | Render the UI and prove it's balanced + usable: a deterministic layout audit (centroid / optical-center / pixe | `agentic-awesome-skills` |
| [dev-story](./dev-story) | Read a story file and implement it. Loads the full context (story, GDD requirement, ADR guidelines, control ma | `Claude-Code-Game-Studios-main` |
| [dev-team](./dev-team) | Simulate a collaborative dev team session where multiple role-based personas (PM, Architect, Developer, QA) re | `ECC` |
| [devops-deploy](./devops-deploy) | DevOps e deploy de aplicacoes — Docker, CI/CD com GitHub Actions, AWS Lambda, SAM, Terraform, infraestrutura c | `agentic-awesome-skills` |
| [devops-deploy-antigravity-awesome-skills-main](./devops-deploy-antigravity-awesome-skills-main) | DevOps e deploy de aplicacoes — Docker, CI/CD com GitHub Actions, AWS Lambda, SAM, Terraform, infraestrutura c | `antigravity-awesome-skills-main` |
| [dg-piagent](./dg-piagent) | \| | `buchidonggua__dg-ai-notes` |
| [dhdna-profiler](./dhdna-profiler) | Extract cognitive patterns and thinking fingerprints from any text. Use this skill when the user wants to anal | `scientific-agent-skills` |
| [diagnosing-bugs](./diagnosing-bugs) | Diagnosis loop for hard bugs and performance regressions. Use when the user says "diagnose"/"debug this", or r | `agentic-awesome-skills` |
| [diffdock](./diffdock) | DiffDock and DiffDock-L molecular docking. Use for protein-small-molecule pose prediction from PDB or sequence | `scientific-agent-skills` |
| [discord-automation](./discord-automation) | Automate Discord tasks via Rube MCP (Composio): messages, channels, roles, webhooks, reactions. Always search  | `agentic-awesome-skills` |
| [discord-automation-antigravity-awesome-skills-main](./discord-automation-antigravity-awesome-skills-main) | Automate Discord tasks via Rube MCP (Composio): messages, channels, roles, webhooks, reactions. Always search  | `antigravity-awesome-skills-main` |
| [discord-bot-architect](./discord-bot-architect) | Specialized skill for building production-ready Discord bots. | `agentic-awesome-skills` |
| [discord-bot-architect-antigravity-awesome-skills-main](./discord-bot-architect-antigravity-awesome-skills-main) | Specialized skill for building production-ready Discord bots. | `antigravity-awesome-skills-main` |
| [discover-agentic](./discover-agentic) | Automatically discover agentic workflow skills when building AI agents, implementing tool use patterns, managi | `cc-polymath` |
| [discover-api](./discover-api) | Automatically discover API design skills when working with REST APIs, GraphQL schemas, API authentication, OAu | `cc-polymath` |
| [discover-cicd](./discover-cicd) | Automatically discover CI/CD and automation skills when working with GitHub Actions, Jenkins, GitLab CI, pipel | `cc-polymath` |
| [discover-cryptography](./discover-cryptography) | Automatically discover cryptography skills when working with encryption, TLS, certificates, PKI, and security | `cc-polymath` |
| [discover-data](./discover-data) | Automatically discover data pipeline and ETL skills when working with ETL, data pipelines, streaming, batch pr | `cc-polymath` |
| [discover-database](./discover-database) | Automatically discover database skills when working with SQL, PostgreSQL, MongoDB, Redis, database schema desi | `cc-polymath` |
| [discover-debugging](./discover-debugging) | Automatically discover debugging and profiling skills when working with GDB, LLDB, breakpoints, profiling, sta | `cc-polymath` |
| [discover-engineering](./discover-engineering) | Automatically discover software engineering practice skills when working with code review, documentation, pair | `cc-polymath` |
| [discover-frontend](./discover-frontend) | Automatically discover frontend development skills when working with React, Next.js, UI components, state mana | `cc-polymath` |
| [discover-math](./discover-math) | Automatically discover mathematics and algorithm skills when working with linear algebra, calculus, optimizati | `cc-polymath` |
| [discover-mcp](./discover-mcp) | Automatically discover MCP (Model Context Protocol) skills when building MCP servers, designing tools, impleme | `cc-polymath` |
| [discover-ml](./discover-ml) | Automatically discover machine learning and AI skills when working with machine learning, PyTorch, training, i | `cc-polymath` |
| [discover-mobile](./discover-mobile) | Automatically discover mobile development skills when working with iOS, Android, Swift, SwiftUI, React Native, | `cc-polymath` |
| [discover-networking](./discover-networking) | Automatically discover networking and connectivity skills when working with TCP, UDP, DNS, mTLS, NAT traversal | `cc-polymath` |
| [discover-product](./discover-product) | Automatically discover product management skills when working with product management, roadmap, user stories,  | `cc-polymath` |
| [discover-research](./discover-research) | Automatically discover research methodology skills when working with research methodology, literature review,  | `cc-polymath` |
| [discover-testing](./discover-testing) | Automatically discover testing skills when working with unit testing, integration testing, e2e testing, TDD, t | `cc-polymath` |
| [discover-wasm](./discover-wasm) | Automatically discover WebAssembly skills when working with WebAssembly, WASM, WASI, wasm-bindgen, Rust to WAS | `cc-polymath` |
| [discover-zig](./discover-zig) | Automatically discover Zig programming skills when working with Zig, comptime, allocators, build.zig, safety,  | `cc-polymath` |
| [distribute-skill-to-all-agents](./distribute-skill-to-all-agents) | Distribute a skill across configured agent skill folders while respecting local symlink layouts. | `agentic-awesome-skills` |
| [ditto](./ditto) | Use when a user asks to mine or update a private, evidence-backed work profile from local Claude Code, Codex,  | `agentic-awesome-skills` |
| [django-patterns](./django-patterns) | Django architecture patterns, REST API design with DRF, ORM best practices, caching, signals, middleware, and  | `everything-claude-code-main` |
| [django-tdd](./django-tdd) | Django testing strategies with pytest-django, TDD methodology, factory_boy, mocking, coverage, and testing Dja | `everything-claude-code-main` |
| [django-verification](./django-verification) | Verification loop for Django projects: migrations, linting, tests with coverage, security scans, and deploymen | `everything-claude-code-main` |
| [dmux-workflows](./dmux-workflows) | Multi-agent orchestration using dmux (tmux pane manager for AI agents). Patterns for parallel agent workflows  | `ECC` |
| [dmux-workflows-ECC](./dmux-workflows-ECC) | Multi-agent orchestration using dmux (tmux pane manager for AI agents). Patterns for parallel agent workflows  | `ECC` |
| [dmux-workflows-everything-claude-code-main](./dmux-workflows-everything-claude-code-main) | Multi-agent orchestration using dmux (tmux pane manager for AI agents). Patterns for parallel agent workflows  | `everything-claude-code-main` |
| [dmux-workflows-everything-claude-code-main](./dmux-workflows-everything-claude-code-main) | Multi-agent orchestration using dmux (tmux pane manager for AI agents). Patterns for parallel agent workflows  | `everything-claude-code-main` |
| [dnanexus-integration](./dnanexus-integration) | Build and operate reproducible genomics workloads on DNAnexus with the dx CLI, dxpy, apps/applets, native work | `scientific-agent-skills` |
| [docker-patterns](./docker-patterns) | Docker and Docker Compose patterns for local development, container security, networking, volume strategies, a | `everything-claude-code-main` |
| [docs-planner](./docs-planner) | Identify documentation gaps and prioritize the docs backlog. Use when planning a docs improvement sprint, afte | `harness-sdk` |
| [doctor](./doctor) | 环境检查与安装向导。检查数学建模工作流所需的全部依赖是否已安装，对缺失项提供安装命令，并在用户确认后执行安装。手动触发。 | `MathModelAgent` |
| [document-to-action-items](./document-to-action-items) | Extract cited obligations, deadlines, tasks from documents. | `hermes-agent` |
| [documentation-and-adrs](./documentation-and-adrs) | Records decisions and documentation. Use when making architectural decisions, changing public APIs, shipping f | `agentic-awesome-skills` |
| [documentation-lookup](./documentation-lookup) | Use up-to-date library and framework docs via Context7 MCP instead of training data. Activates for setup quest | `ECC` |
| [documentation-lookup-ECC](./documentation-lookup-ECC) | Use up-to-date library and framework docs via Context7 MCP instead of training data. Activates for setup quest | `ECC` |
| [documentation-lookup-everything-claude-code-main](./documentation-lookup-everything-claude-code-main) | Use up-to-date library and framework docs via Context7 MCP instead of training data. Activates for setup quest | `everything-claude-code-main` |
| [docusign-automation](./docusign-automation) | Automate DocuSign tasks via Rube MCP (Composio): templates, envelopes, signatures, document management. Always | `agentic-awesome-skills` |
| [docusign-automation-antigravity-awesome-skills-main](./docusign-automation-antigravity-awesome-skills-main) | Automate DocuSign tasks via Rube MCP (Composio): templates, envelopes, signatures, document management. Always | `antigravity-awesome-skills-main` |
| [docx](./docx) | Create, read, edit, template, and review Word .docx files. | `hermes-agent` |
| [docx-scientific-agent-skills](./docx-scientific-agent-skills) | Use this skill whenever the user wants to create, read, edit, or manipulate Word documents (.docx files) or Wo | `scientific-agent-skills` |
| [dogfood](./dogfood) | Exploratory QA of web apps: find bugs, evidence, reports. | `hermes-agent` |
| [domain-agent-coaching](./domain-agent-coaching) | Coach the design and ongoing reflection of long-lived domain Agents and Agent organizations. Use when defining | `codexloom` |
| [domain-modeling](./domain-modeling) | Build and sharpen a project's domain model. Use when the user wants to pin down domain terminology or a ubiqui | `agentic-awesome-skills` |
| [doubt-driven-development](./doubt-driven-development) | Subjects every non-trivial decision to a fresh-context adversarial review before it stands. | `agentic-awesome-skills` |
| [drizzle-migration-conflict](./drizzle-migration-conflict) | Diagnose, repair, and prevent Drizzle Kit migration conflicts involving generated SQL, snapshots, journals, me | `agentic-awesome-skills` |
| [dropbox-automation](./dropbox-automation) | Automate Dropbox file management, sharing, search, uploads, downloads, and folder operations via Rube MCP (Com | `agentic-awesome-skills` |
| [dropbox-automation-antigravity-awesome-skills-main](./dropbox-automation-antigravity-awesome-skills-main) | Automate Dropbox file management, sharing, search, uploads, downloads, and folder operations via Rube MCP (Com | `antigravity-awesome-skills-main` |
| [dsh-upgrade-audit](./dsh-upgrade-audit) | Audit external compatibility between two DSH (DeepSeek Harness) versions and detect reverts, producing an upgr | `dsh-agent-teams` |
| [dynamic-workflow-mode](./dynamic-workflow-mode) | Design task-local harnesses, eval gates, and reusable skill extraction for Claude dynamic workflow mode and ot | `ECC` |
| [e2e-testing](./e2e-testing) | Playwright E2E testing patterns, Page Object Model, configuration, CI/CD integration, artifact management, and | `everything-claude-code-main` |
| [earllm-build](./earllm-build) | Build, maintain, and extend the EarLLM One Android project — a Kotlin/Compose app that connects Bluetooth earb | `agentic-awesome-skills` |
| [earllm-build-antigravity-awesome-skills-main](./earllm-build-antigravity-awesome-skills-main) | Build, maintain, and extend the EarLLM One Android project — a Kotlin/Compose app that connects Bluetooth earb | `antigravity-awesome-skills-main` |
| [ecc-guide](./ecc-guide) | Guide users through ECC's current agents, skills, commands, hooks, rules, install profiles, and project onboar | `ECC` |
| [ecc-recipes](./ecc-recipes) | Map a described workflow to the right ECC command-GROUP with run-order and stop condition, and browse all comm | `ECC` |
| [effective-agent-skills](./effective-agent-skills) | Author and review high-quality agent skills with triggers, progressive disclosure, and safety notes. | `agentic-awesome-skills` |
| [elon-musk](./elon-musk) | Agente que simula Elon Musk com profundidade psicologica e comunicacional de alta fidelidade. Ativado para: \" | `agentic-awesome-skills` |
| [elon-musk-antigravity-awesome-skills-main](./elon-musk-antigravity-awesome-skills-main) | Agente que simula Elon Musk com profundidade psicologica e comunicacional de alta fidelidade. Ativado para: \" | `antigravity-awesome-skills-main` |
| [email-inbox-triage](./email-inbox-triage) | Triage an inbox: prioritize threads, draft replies safely. | `hermes-agent` |
| [email-ops](./email-ops) | Evidence-first mailbox triage, drafting, send verification, and sent-mail-safe follow-up workflow for ECC. Use | `everything-claude-code-main` |
| [emblemai-crypto-wallet](./emblemai-crypto-wallet) | Crypto wallet management across 7 blockchains via EmblemAI Agent Hustle API. Balance checks, token swaps, port | `agentic-awesome-skills` |
| [emblemai-crypto-wallet-antigravity-awesome-skills-main](./emblemai-crypto-wallet-antigravity-awesome-skills-main) | Crypto wallet management across 7 blockchains via EmblemAI Agent Hustle API. Balance checks, token swaps, port | `antigravity-awesome-skills-main` |
| [endpoint-probe](./endpoint-probe) | > | `Claude-Code-Agent-Monitor` |
| [energy-procurement](./energy-procurement) | Codified expertise for electricity and gas procurement, tariff optimisation, demand charge management, renewab | `agentic-awesome-skills` |
| [energy-procurement-antigravity-awesome-skills-main](./energy-procurement-antigravity-awesome-skills-main) | Codified expertise for electricity and gas procurement, tariff optimisation, demand charge management, renewab | `antigravity-awesome-skills-main` |
| [energy-procurement-ECC](./energy-procurement-ECC) | > | `ECC` |
| [energy-procurement-everything-claude-code-main](./energy-procurement-everything-claude-code-main) | > | `everything-claude-code-main` |
| [entropy-box](./entropy-box) | Entropy Box knowledge-compiler for embodied-AI: turns bounded requirements into grounded workflows via Solutio | `agentic-awesome-skills` |
| [error-debugging-multi-agent-review](./error-debugging-multi-agent-review) | Use when working with error debugging multi agent review | `agentic-awesome-skills` |
| [error-debugging-multi-agent-review-antigravity-awesome-skills-main](./error-debugging-multi-agent-review-antigravity-awesome-skills-main) | Use when working with error debugging multi agent review | `antigravity-awesome-skills-main` |
| [error-diagnostics-smart-debug](./error-diagnostics-smart-debug) | Use when working with error diagnostics smart debug | `agentic-awesome-skills` |
| [error-diagnostics-smart-debug-antigravity-awesome-skills-main](./error-diagnostics-smart-debug-antigravity-awesome-skills-main) | Use when working with error diagnostics smart debug | `antigravity-awesome-skills-main` |
| [error-propagation](./error-propagation) | > | `Claude-Code-Agent-Monitor` |
| [error-scan](./error-scan) | > | `Claude-Code-Agent-Monitor` |
| [esm](./esm) | Use when working directly with the `esm` Python SDK, ESM3 or ESMC model IDs, Forge/Biohub inference clients, o | `scientific-agent-skills` |
| [estimate](./estimate) | Estimates task effort by analyzing complexity, dependencies, historical velocity, and risk factors. Produces a | `Claude-Code-Game-Studios-main` |
| [eval-harness](./eval-harness) | Formal evaluation framework for Claude Code sessions implementing eval-driven development (EDD) principles. Us | `ECC` |
| [eval-harness-ECC](./eval-harness-ECC) | Formal evaluation framework for Claude Code sessions implementing eval-driven development (EDD) principles. Us | `ECC` |
| [eval-harness-everything-claude-code-main](./eval-harness-everything-claude-code-main) | Formal evaluation framework for Claude Code sessions implementing eval-driven development (EDD) principles | `everything-claude-code-main` |
| [eval-harness-everything-claude-code-main](./eval-harness-everything-claude-code-main) | Formal evaluation framework for Claude Code sessions implementing eval-driven development (EDD) principles | `everything-claude-code-main` |
| [eval-skills](./eval-skills) | Eval and improve a skill against golden cases — run the target skill blind in a fresh, context-free subagent o | `skills` |
| [evaluation](./evaluation) | This skill should be used when the user asks to "evaluate agent performance", "build test framework", "measure | `Agent-Skills-for-Context-Engineering-main` |
| [evaluation-agentic-awesome-skills](./evaluation-agentic-awesome-skills) | Build evaluation frameworks for agent systems. Use when testing agent performance systematically, validating c | `agentic-awesome-skills` |
| [evaluation-antigravity-awesome-skills-main](./evaluation-antigravity-awesome-skills-main) | Build evaluation frameworks for agent systems. Use when testing agent performance systematically, validating c | `antigravity-awesome-skills-main` |
| [eve](./eve) | Build durable backend AI agents with the eve framework. Use when creating, editing, or debugging an eve projec | `vercel__eve` |
| [event-staffing-ordering](./event-staffing-ordering) | Order W-2 compliant temporary event staff for conventions, trade shows, festivals, concerts, sporting events,  | `agentic-awesome-skills` |
| [event-trace](./event-trace) | > | `Claude-Code-Agent-Monitor` |
| [everything-claude-code](./everything-claude-code) | Development conventions and patterns for everything-claude-code. JavaScript project with conventional commits. | `ECC` |
| [everything-claude-code-everything-claude-code-main](./everything-claude-code-everything-claude-code-main) | Development conventions and patterns for everything-claude-code. JavaScript project with conventional commits. | `everything-claude-code-main` |
| [everything-claude-code-everything-claude-code-main](./everything-claude-code-everything-claude-code-main) | Development conventions and patterns for everything-claude-code. JavaScript project with conventional commits. | `everything-claude-code-main` |
| [evm-token-decimals](./evm-token-decimals) | Prevent silent decimal mismatch bugs across EVM chains. Covers runtime decimal lookup, chain-aware caching, br | `everything-claude-code-main` |
| [evolution](./evolution) | This skill enables makepad-skills to self-improve continuously during development. | `agentic-awesome-skills` |
| [evolution-antigravity-awesome-skills-main](./evolution-antigravity-awesome-skills-main) | This skill enables makepad-skills to self-improve continuously during development. | `antigravity-awesome-skills-main` |
| [exa-search](./exa-search) | Semantic search, similar content discovery, and structured research using Exa API. Use when you need semantic/ | `agentic-awesome-skills` |
| [exa-search-antigravity-awesome-skills-main](./exa-search-antigravity-awesome-skills-main) | Semantic search, similar content discovery, and structured research using Exa API. Use when you need semantic/ | `antigravity-awesome-skills-main` |
| [exa-search-ECC](./exa-search-ECC) | Neural search via Exa MCP for web, code, and company research. Use when the user needs web search, code exampl | `ECC` |
| [exa-search-everything-claude-code-main](./exa-search-everything-claude-code-main) | Neural search via Exa MCP for web, code, and company research. Use when the user needs web search, code exampl | `everything-claude-code-main` |
| [exa-search-scientific-agent-skills](./exa-search-scientific-agent-skills) | Web toolkit powered by Exa, tuned for scientific and technical content. Use this skill when the user needs to  | `scientific-agent-skills` |
| [example-command](./example-command) | An example user-invoked skill that demonstrates frontmatter options and the skills/<name>/SKILL.md layout | `claude-plugins-official-main` |
| [example-skill](./example-skill) | This skill should be used when the user asks to "demonstrate skills", "show skill format", "create a skill tem | `claude-plugins-official-main` |
| [executing-plans](./executing-plans) | The checkpoint coordinator for /feature. Routed to by /feature once a writing-plans plan exists. Groups tasks  | `arbiterForge__codeArbiter` |
| [executing-plans-superpowers-main](./executing-plans-superpowers-main) | Use when you have a written implementation plan to execute in a separate session with review checkpoints | `superpowers-main` |
| [executive-report](./executive-report) | > | `Claude-Code-Agent-Monitor` |
| [experimental-design](./experimental-design) | Design experiments and studies BEFORE data is collected — choosing a design, randomizing, blocking, and laying | `scientific-agent-skills` |
| [explicit-identity](./explicit-identity) | Explicit Identity Across Boundaries | `Continuous-Claude-v3-main` |
| [exploratory-data-analysis](./exploratory-data-analysis) | Perform bounded, local exploratory analysis of explicitly supported scientific files. Use for redacted CSV/TSV | `scientific-agent-skills` |
| [extract-document-data](./extract-document-data) | Extract structured, grounded fields from documents — values cite their page, missing values abstain instead of | `agentic-awesome-skills` |
| [faf-expert](./faf-expert) | Advanced .faf (Foundational AI-context Format) specialist. IANA-registered format, MCP server config, champion | `agentic-awesome-skills` |
| [faf-expert-antigravity-awesome-skills-main](./faf-expert-antigravity-awesome-skills-main) | Advanced .faf (Foundational AI-context Format) specialist. IANA-registered format, MCP server config, champion | `antigravity-awesome-skills-main` |
| [faf-go](./faf-go) | Guided interview to Gold Code (100% AI-Readiness). Use when helping users improve their .faf file through ques | `agentic-awesome-skills` |
| [fal-ai-media](./fal-ai-media) | Unified media generation via fal.ai MCP — image, video, and audio. Covers text-to-image (Nano Banana), text/im | `ECC` |
| [fal-ai-media-ECC](./fal-ai-media-ECC) | Unified media generation via fal.ai MCP — image, video, and audio. Covers text-to-image (Nano Banana), text/im | `ECC` |
| [fal-ai-media-everything-claude-code-main](./fal-ai-media-everything-claude-code-main) | Unified media generation via fal.ai MCP — image, video, and audio. Covers text-to-image (Nano Banana), text/im | `everything-claude-code-main` |
| [fal-ai-media-everything-claude-code-main](./fal-ai-media-everything-claude-code-main) | Unified media generation via fal.ai MCP — image, video, and audio. Covers text-to-image (Nano Banana), text/im | `everything-claude-code-main` |
| [falsify](./falsify) | The scientific thinking protocol for AI agents. Use when facing complex, ambiguous, or high-stakes questions w | `agentic-awesome-skills` |
| [famulor-skill](./famulor-skill) | Operate Famulor assistants, communication history, campaigns, knowledge, automations, telephony, and workspace | `agentic-awesome-skills` |
| [figma-automation](./figma-automation) | Automate Figma tasks via Rube MCP (Composio): files, components, design tokens, comments, exports. Always sear | `agentic-awesome-skills` |
| [figma-automation-antigravity-awesome-skills-main](./figma-automation-antigravity-awesome-skills-main) | Automate Figma tasks via Rube MCP (Composio): files, components, design tokens, comments, exports. Always sear | `antigravity-awesome-skills-main` |
| [figma-code-connect-components](./figma-code-connect-components) | \| | `abingyyds__open-design` |
| [figma-code-connect-components-open-design](./figma-code-connect-components-open-design) | \| | `open-design` |
| [figma-create-design-system-rules](./figma-create-design-system-rules) | \| | `abingyyds__open-design` |
| [figma-create-design-system-rules-open-design](./figma-create-design-system-rules-open-design) | \| | `open-design` |
| [figma-create-new-file](./figma-create-new-file) | \| | `abingyyds__open-design` |
| [figma-create-new-file-open-design](./figma-create-new-file-open-design) | \| | `open-design` |
| [figma-generate-design](./figma-generate-design) | \| | `abingyyds__open-design` |
| [figma-generate-design-open-design](./figma-generate-design-open-design) | \| | `open-design` |
| [figma-generate-library](./figma-generate-library) | \| | `abingyyds__open-design` |
| [figma-generate-library-open-design](./figma-generate-library-open-design) | \| | `open-design` |
| [figma-implement-design](./figma-implement-design) | \| | `abingyyds__open-design` |
| [figma-implement-design-open-design](./figma-implement-design-open-design) | \| | `open-design` |
| [figma-use](./figma-use) | \| | `abingyyds__open-design` |
| [figma-use-open-design](./figma-use-open-design) | \| | `open-design` |
| [file-headers](./file-headers) | MANDATORY for every coding agent (Claude Code, Codex, or any other) on every change-set — every applicable sou | `Claude-Code-Agent-Monitor` |
| [filesystem-context](./filesystem-context) | This skill should be used when the user asks to "offload context to files", "implement dynamic context discove | `Agent-Skills-for-Context-Engineering-main` |
| [finance-billing-ops](./finance-billing-ops) | Evidence-first revenue, pricing, refunds, team-billing, and billing-model truth workflow for ECC. Use when the | `everything-claude-code-main` |
| [find-complementary-founders](./find-complementary-founders) | Use when an owner explicitly asks for a cofounder or project partner, or explicitly says they need a complemen | `agentic-awesome-skills` |
| [find-matching-tenders](./find-matching-tenders) | Find open AU/NZ government tenders matching what a company does, ranked by fit with why and gap analysis. Use  | `agentic-awesome-skills` |
| [find-me-skills](./find-me-skills) | Find Agent Skills for a goal the user cannot name yet, then export an installable bundle. Use when they ask wh | `luongnv89__asm` |
| [find-skills](./find-skills) | Helps users discover and install agent skills when they ask questions like "how do I do X", "find a skill for  | `Auto-Company` |
| [find-skills-deer-flow](./find-skills-deer-flow) | Helps users discover and install agent skills when they ask questions like "how do I do X", "find a skill for  | `deer-flow` |
| [find-skills-fastclaw](./find-skills-fastclaw) | \| | `fastclaw` |
| [find-skills-grok-cli-main](./find-skills-grok-cli-main) | Helps users discover and install agent skills when they ask questions like "how do I do X", "find a skill for  | `grok-cli-main` |
| [find-skills-Leon-Drq__openagentskill](./find-skills-Leon-Drq__openagentskill) | Find, compare, audit, and safely install reusable AI agent skills from OpenAgentSkill. Use when a user asks fo | `Leon-Drq__openagentskill` |
| [find-skills-OpenBMB__PilotDeck](./find-skills-OpenBMB__PilotDeck) | Helps users discover and install agent skills when they ask questions like "how do I do X", "find a skill for  | `OpenBMB__PilotDeck` |
| [findmy](./findmy) | Track Apple devices/AirTags via FindMy.app on macOS. | `hermes-agent` |
| [firecrawl-scrape](./firecrawl-scrape) | Scrape web pages and extract content via Firecrawl MCP | `Continuous-Claude-v3-main` |
| [firecrawl-scraper](./firecrawl-scraper) | Deep web scraping, screenshots, PDF parsing, and website crawling using Firecrawl API. Use when you need deep  | `agentic-awesome-skills` |
| [firecrawl-scraper-antigravity-awesome-skills-main](./firecrawl-scraper-antigravity-awesome-skills-main) | Deep web scraping, screenshots, PDF parsing, and website crawling using Firecrawl API. Use when you need deep  | `antigravity-awesome-skills-main` |
| [fix](./fix) | Meta-skill workflow orchestrator for bug investigation and resolution. Routes to debug, implement, test, and c | `Continuous-Claude-v3-main` |
| [fleet-auditor](./fleet-auditor) | Cross-system agent token/cost audit (Claude Code, Codex, OpenClaw, Hermes, OpenCode): idle burns, model misrou | `token-optimizer` |
| [flow-define](./flow-define) | Multi-AI requirements scoping using available external providers (Double Diamond Define phase). Priority trigg | `claude-octopus` |
| [flow-define-claude-octopus](./flow-define-claude-octopus) | Multi-AI requirements scoping using available external providers (Double Diamond Define phase). Priority trigg | `claude-octopus` |
| [flow-deliver](./flow-deliver) | Multi-AI validation, scoring, and review using available external providers (Double Diamond Deliver phase) | `claude-octopus` |
| [flow-deliver-claude-octopus](./flow-deliver-claude-octopus) | Multi-AI validation, scoring, and review using available external providers (Double Diamond Deliver phase) | `claude-octopus` |
| [flow-develop](./flow-develop) | Multi-AI implementation using available external providers (Double Diamond Develop phase). DO NOT use for simp | `claude-octopus` |
| [flow-develop-claude-octopus](./flow-develop-claude-octopus) | Multi-AI implementation using available external providers (Double Diamond Develop phase). DO NOT use for simp | `claude-octopus` |
| [flow-discover](./flow-discover) | Multi-AI research using available external providers (Double Diamond Discover phase) | `claude-octopus` |
| [flow-discover-claude-octopus](./flow-discover-claude-octopus) | Multi-AI research using available external providers (Double Diamond Discover phase) | `claude-octopus` |
| [flow-nexus-neural](./flow-nexus-neural) | Train and deploy neural networks in distributed E2B sandboxes with Flow Nexus | `ruflo` |
| [flow-nexus-neural-ruflo](./flow-nexus-neural-ruflo) | Train and deploy neural networks in distributed E2B sandboxes with Flow Nexus | `ruflo` |
| [flow-nexus-neural-ruflo-main](./flow-nexus-neural-ruflo-main) | Train and deploy neural networks in distributed E2B sandboxes with Flow Nexus | `ruflo-main` |
| [flow-nexus-neural-ruflo-main](./flow-nexus-neural-ruflo-main) | Train and deploy neural networks in distributed E2B sandboxes with Flow Nexus | `ruflo-main` |
| [flow-nexus-swarm](./flow-nexus-swarm) | Cloud-based AI swarm deployment and event-driven workflow automation with Flow Nexus platform | `ruflo` |
| [flow-nexus-swarm-ruflo](./flow-nexus-swarm-ruflo) | Cloud-based AI swarm deployment and event-driven workflow automation with Flow Nexus platform | `ruflo` |
| [flow-nexus-swarm-ruflo-main](./flow-nexus-swarm-ruflo-main) | Cloud-based AI swarm deployment and event-driven workflow automation with Flow Nexus platform | `ruflo-main` |
| [flow-nexus-swarm-ruflo-main](./flow-nexus-swarm-ruflo-main) | Cloud-based AI swarm deployment and event-driven workflow automation with Flow Nexus platform | `ruflo-main` |
| [flow-parallel](./flow-parallel) | Decompose and execute large changes, migrations, or multi-issue fixes in parallel with quality gates | `claude-octopus` |
| [flow-parallel-claude-octopus](./flow-parallel-claude-octopus) | Decompose and execute large changes, migrations, or multi-issue fixes in parallel with quality gates | `claude-octopus` |
| [flow-spec](./flow-spec) | NLSpec authoring — use when you need a structured specification from multi-AI research and consensus | `claude-octopus` |
| [flowhunt-skill](./flowhunt-skill) | Automation discovery audit skill. Walks through a 5-question workflow intake, then audits Gmail/Calendar/Slack | `agentic-awesome-skills` |
| [flowio](./flowio) | Read, inspect, and write Flow Cytometry Standard (FCS) 2.0, 3.0, and 3.1 files with FlowIO. Use for low-level  | `scientific-agent-skills` |
| [fluidsim](./fluidsim) | Plan, configure, inspect, restart, and analyze bounded FluidSim computational-fluid-dynamics simulations with  | `scientific-agent-skills` |
| [flutter-dart-code-review](./flutter-dart-code-review) | Library-agnostic Flutter/Dart code review checklist covering widget best practices, state management patterns  | `everything-claude-code-main` |
| [folder-specific-claude-and-agents-md](./folder-specific-claude-and-agents-md) | Create folder-scoped CLAUDE.md and AGENTS.md guidance for future agents working in that area. | `agentic-awesome-skills` |
| [formik-patterns](./formik-patterns) | Formik form handling with validation patterns. Use when building forms, implementing validation, or handling f | `agentic-awesome-skills` |
| [freshdesk-automation](./freshdesk-automation) | Automate Freshdesk helpdesk operations including tickets, contacts, companies, notes, and replies via Rube MCP | `agentic-awesome-skills` |
| [freshdesk-automation-antigravity-awesome-skills-main](./freshdesk-automation-antigravity-awesome-skills-main) | Automate Freshdesk helpdesk operations including tickets, contacts, companies, notes, and replies via Rube MCP | `antigravity-awesome-skills-main` |
| [freshservice-automation](./freshservice-automation) | Automate Freshservice ITSM tasks via Rube MCP (Composio): create/update tickets, bulk operations, service requ | `agentic-awesome-skills` |
| [freshservice-automation-antigravity-awesome-skills-main](./freshservice-automation-antigravity-awesome-skills-main) | Automate Freshservice ITSM tasks via Rube MCP (Composio): create/update tickets, bulk operations, service requ | `antigravity-awesome-skills-main` |
| [frontend-architecture](./frontend-architecture) | A portable, framework-agnostic architecture style for any React or React Native frontend. Organizes apps into  | `agentic-awesome-skills` |
| [frontend-data-contracts](./frontend-data-contracts) | A portable, framework-agnostic discipline for type safety at the network edge of any React or React Native app | `agentic-awesome-skills` |
| [frontend-design](./frontend-design) | Create distinctive, production-grade frontend interfaces with high design quality. Use when the user asks to b | `everything-claude-code-main` |
| [frontend-lighthouse](./frontend-lighthouse) | Add a portable Lighthouse CI gate for production frontend builds with Core Web Vitals budgets, category floors | `agentic-awesome-skills` |
| [frontend-module-standards](./frontend-module-standards) | Enforce this repository's React and TypeScript frontend module architecture standards. Use whenever creating,  | `claudecodeui` |
| [frontend-observability](./frontend-observability) | A portable, framework-agnostic field-side observability system for any React or React Native app. | `agentic-awesome-skills` |
| [frontend-optimistic-mutations](./frontend-optimistic-mutations) | A portable, framework-agnostic discipline for the write path of any React or React Native app using a query/ca | `agentic-awesome-skills` |
| [frontend-patterns](./frontend-patterns) | Frontend development patterns for React, Next.js, state management, performance optimization, and UI best prac | `everything-claude-code-main` |
| [frontend-seo](./frontend-seo) | A portable, framework-agnostic SEO system for any React or React Native-for-web frontend. | `agentic-awesome-skills` |
| [frontend-ui-engineering](./frontend-ui-engineering) | Builds production-quality UIs. Use when building or modifying user-facing interfaces. Use when creating compon | `agentic-awesome-skills` |
| [gaia-submission](./gaia-submission) | Walk through a complete GAIA benchmark→submit flow — from key resolution through HAL-compatible package genera | `ruflo` |
| [gan-style-harness](./gan-style-harness) | GAN-inspired Generator-Evaluator agent harness for building high-quality applications autonomously. Based on A | `ECC` |
| [gan-style-harness-everything-claude-code-main](./gan-style-harness-everything-claude-code-main) | GAN-inspired Generator-Evaluator agent harness for building high-quality applications autonomously. Based on A | `everything-claude-code-main` |
| [gate-check](./gate-check) | Validate readiness to advance between development phases. Produces a PASS/CONCERNS/FAIL verdict with specific  | `Claude-Code-Game-Studios-main` |
| [gdb-cli](./gdb-cli) | GDB debugging assistant for AI agents - analyze core dumps, debug live processes, investigate crashes and dead | `agentic-awesome-skills` |
| [gdb-cli-antigravity-awesome-skills-main](./gdb-cli-antigravity-awesome-skills-main) | GDB debugging assistant for AI agents - analyze core dumps, debug live processes, investigate crashes and dead | `antigravity-awesome-skills-main` |
| [Geek-skills-keqian-method](./Geek-skills-keqian-method) | 胥克谦式AI-Native产品开发方法论。适用于：(1) 使用AI Agent（Claude Code、Codex、Cursor等）进行产品级软件开发，(2) 设计和优化Harness/Skill体系，(3) 文档驱动开 | `ClaudeSkills` |
| [generate-image](./generate-image) | Generate or edit images with AI models through the OpenRouter Image API (Gemini, Seedream, Recraft, GPT-Image, | `scientific-agent-skills` |
| [geniml](./geniml) | Use Geniml for audited local genomic-interval workflows: validate BED and universe contracts, plan Region2Vec  | `scientific-agent-skills` |
| [genomic-intelligence](./genomic-intelligence) | Predict regulatory features, gene structure, and expression directly from DNA sequence using Genomic Intellige | `scientific-agent-skills` |
| [geoffrey-hinton](./geoffrey-hinton) | Agente que simula Geoffrey Hinton — Godfather of Deep Learning, Prêmio Turing 2018, criador do backpropagation | `agentic-awesome-skills` |
| [geoffrey-hinton-antigravity-awesome-skills-main](./geoffrey-hinton-antigravity-awesome-skills-main) | Agente que simula Geoffrey Hinton — Godfather of Deep Learning, Prêmio Turing 2018, criador do backpropagation | `antigravity-awesome-skills-main` |
| [geomaster](./geomaster) | Comprehensive geospatial science skill covering remote sensing, GIS, spatial analysis, machine learning for ea | `scientific-agent-skills` |
| [geopandas](./geopandas) | Guidance and local audit tools for Python workflows that directly use GeoPandas GeoSeries, GeoDataFrame, spati | `scientific-agent-skills` |
| [get-available-resources](./get-available-resources) | Detect host inventory and effective CPU, memory, disk, scheduler, container, and accelerator limits when a use | `scientific-agent-skills` |
| [gha](./gha) | Analyze GitHub Actions failures and identify root causes | `claude-code-tips-main` |
| [gif-search](./gif-search) | Search/download GIFs from Tenor via curl + jq. | `hermes-agent` |
| [git-commits](./git-commits) | Git Commit Rules | `Continuous-Claude-v3-main` |
| [git-guardrails-claude-code](./git-guardrails-claude-code) | Set up Claude Code hooks to block dangerous git commands (push, reset --hard, clean, branch -D, etc.) before t | `mattpocock__skills` |
| [git-pr-workflows-git-workflow](./git-pr-workflows-git-workflow) | Orchestrate a comprehensive git workflow from code review through PR creation, leveraging specialized agents f | `antigravity-awesome-skills-main` |
| [git-workflow](./git-workflow) | Git workflow patterns including branching strategies, commit conventions, merge vs rebase, conflict resolution | `everything-claude-code-main` |
| [git-workflow-and-versioning](./git-workflow-and-versioning) | Structures git workflow practices. Use when making any code change. Use when committing, branching, resolving  | `agentic-awesome-skills` |
| [github](./github) | GitHub via gh CLI: PRs, issues, reviews, repos, auth. | `hermes-agent` |
| [github-automation](./github-automation) | Automate GitHub repositories, issues, pull requests, branches, CI/CD, and permissions via Rube MCP (Composio). | `antigravity-awesome-skills-main` |
| [github-code-review](./github-code-review) | Comprehensive GitHub code review with AI-powered swarm coordination | `ruflo` |
| [github-code-review-ruflo-main](./github-code-review-ruflo-main) | Comprehensive GitHub code review with AI-powered swarm coordination | `ruflo-main` |
| [github-code-review-RuView](./github-code-review-RuView) | Comprehensive GitHub code review with AI-powered swarm coordination | `RuView` |
| [github-ops](./github-ops) | GitHub repository operations, automation, and management. Issue triage, PR management, CI/CD operations, relea | `everything-claude-code-main` |
| [github-search](./github-search) | Search GitHub code, repositories, issues, and PRs via MCP | `Continuous-Claude-v3-main` |
| [github-triage](./github-triage) | Triages a repository's open GitHub issues and pull requests via the gh CLI. Optionally reviews and merges read | `trailofbits__skills` |
| [gitlab-automation](./gitlab-automation) | Automate GitLab project management, issues, merge requests, pipelines, branches, and user operations via Rube  | `agentic-awesome-skills` |
| [gitlab-automation-antigravity-awesome-skills-main](./gitlab-automation-antigravity-awesome-skills-main) | Automate GitLab project management, issues, merge requests, pipelines, branches, and user operations via Rube  | `antigravity-awesome-skills-main` |
| [gitnexus-cli](./gitnexus-cli) | Use when the user needs to run GitNexus CLI commands like analyze/index a repo, check status, clean the index, | `GitNexus-main` |
| [gitnexus-impact-analysis](./gitnexus-impact-analysis) | Use when the user wants to know what will break if they change something, or needs safety analysis before edit | `GitNexus` |
| [gitnexus-pr-swarm-review](./gitnexus-pr-swarm-review) | Run a GitNexus production-readiness pull request review using a coordinated reviewer swarm. | `GitNexus` |
| [gitnexus-refactoring](./gitnexus-refactoring) | Use when the user wants to rename, extract, split, move, or restructure code safely. Examples: \"Rename this f | `GitNexus` |
| [global-chat-agent-discovery](./global-chat-agent-discovery) | Discover and search 18K+ MCP servers and AI agents across 6+ registries using Global Chat's cross-protocol dir | `agentic-awesome-skills` |
| [global-chat-agent-discovery-antigravity-awesome-skills-main](./global-chat-agent-discovery-antigravity-awesome-skills-main) | Discover and search 18K+ MCP servers and AI agents across 6+ registries using Global Chat's cross-protocol dir | `antigravity-awesome-skills-main` |
| [glycoengineering](./glycoengineering) | Analyze and engineer protein glycosylation. Scan sequences for N-glycosylation sequons (N-X-S/T), predict O-gl | `scientific-agent-skills` |
| [gmail-automation](./gmail-automation) | Lightweight Gmail integration with standalone OAuth authentication. No MCP server required. | `agentic-awesome-skills` |
| [gmail-automation-antigravity-awesome-skills-main](./gmail-automation-antigravity-awesome-skills-main) | Lightweight Gmail integration with standalone OAuth authentication. No MCP server required. | `antigravity-awesome-skills-main` |
| [gnhf](./gnhf) | Use when the user asks to run GNHF, says they are going to sleep or leaving and wants an agent-managed coding  | `kunchenguid__gnhf` |
| [goal-loop](./goal-loop) | Draft and explain persistent goal-loop prompts for long-running agent work with clear stop conditions. | `agentic-awesome-skills` |
| [goal-prompt](./goal-prompt) | Drafts copy-paste-ready /goal commands for goal mode in Claude Code and Codex. Use when the user asks to creat | `trailofbits__skills` |
| [google-analytics-automation](./google-analytics-automation) | Automate Google Analytics tasks via Rube MCP (Composio): run reports, list accounts/properties, funnels, pivot | `agentic-awesome-skills` |
| [google-analytics-automation-antigravity-awesome-skills-main](./google-analytics-automation-antigravity-awesome-skills-main) | Automate Google Analytics tasks via Rube MCP (Composio): run reports, list accounts/properties, funnels, pivot | `antigravity-awesome-skills-main` |
| [google-calendar-automation](./google-calendar-automation) | Lightweight Google Calendar integration with standalone OAuth authentication. No MCP server required. | `agentic-awesome-skills` |
| [google-calendar-automation-antigravity-awesome-skills-main](./google-calendar-automation-antigravity-awesome-skills-main) | Lightweight Google Calendar integration with standalone OAuth authentication. No MCP server required. | `antigravity-awesome-skills-main` |
| [google-docs-automation](./google-docs-automation) | Lightweight Google Docs integration with standalone OAuth authentication. No MCP server required. | `agentic-awesome-skills` |
| [google-docs-automation-antigravity-awesome-skills-main](./google-docs-automation-antigravity-awesome-skills-main) | Lightweight Google Docs integration with standalone OAuth authentication. No MCP server required. | `antigravity-awesome-skills-main` |
| [google-drive-automation](./google-drive-automation) | Lightweight Google Drive integration with standalone OAuth authentication. No MCP server required. Full read/w | `agentic-awesome-skills` |
| [google-drive-automation-antigravity-awesome-skills-main](./google-drive-automation-antigravity-awesome-skills-main) | Lightweight Google Drive integration with standalone OAuth authentication. No MCP server required. Full read/w | `antigravity-awesome-skills-main` |
| [google-sheets-automation](./google-sheets-automation) | Lightweight Google Sheets integration with standalone OAuth authentication. No MCP server required. Full read/ | `agentic-awesome-skills` |
| [google-sheets-automation-antigravity-awesome-skills-main](./google-sheets-automation-antigravity-awesome-skills-main) | Lightweight Google Sheets integration with standalone OAuth authentication. No MCP server required. Full read/ | `antigravity-awesome-skills-main` |
| [google-slides-automation](./google-slides-automation) | Lightweight Google Slides integration with standalone OAuth authentication. No MCP server required. Full read/ | `agentic-awesome-skills` |
| [google-slides-automation-antigravity-awesome-skills-main](./google-slides-automation-antigravity-awesome-skills-main) | Lightweight Google Slides integration with standalone OAuth authentication. No MCP server required. Full read/ | `antigravity-awesome-skills-main` |
| [google-workspace](./google-workspace) | Gmail, Calendar, Drive, Docs, Sheets via gws CLI or Python. | `hermes-agent` |
| [google-workspace-ops](./google-workspace-ops) | Operate across Google Drive, Docs, Sheets, and Slides as one workflow surface for plans, trackers, decks, and  | `everything-claude-code-main` |
| [googlesheets-automation](./googlesheets-automation) | Automate Google Sheets operations (read, write, format, filter, manage spreadsheets) via Rube MCP (Composio).  | `agentic-awesome-skills` |
| [googlesheets-automation-antigravity-awesome-skills-main](./googlesheets-automation-antigravity-awesome-skills-main) | Automate Google Sheets operations (read, write, format, filter, manage spreadsheets) via Rube MCP (Composio).  | `antigravity-awesome-skills-main` |
| [gpt-image-cli](./gpt-image-cli) | Use GPT-Image2-Skill and the gpt-image-cli package for OpenAI GPT | `AREX-Skill` |
| [graphiti](./graphiti) | Routes Graphiti SDK, REST service, and MCP server workflows. | `AREX-Skill` |
| [graphql-schema](./graphql-schema) | GraphQL queries, mutations, and code generation patterns. Use when creating GraphQL operations, working with A | `agentic-awesome-skills` |
| [greptimedb-release-note](./greptimedb-release-note) | Generate a GreptimeDB release changelog with git cliff (correct range, subtract already-released patch PRs, re | `greptimedb` |
| [grill-me](./grill-me) | A relentless interview to sharpen a plan or design. | `agentic-awesome-skills` |
| [grill-with-docs](./grill-with-docs) | A relentless interview to sharpen a plan or design, which also creates docs (ADR's and glossary) as we go. | `agentic-awesome-skills` |
| [grilling](./grilling) | Interview the user relentlessly about a plan or design. Use when the user wants to stress-test a plan before b | `agentic-awesome-skills` |
| [grok-build](./grok-build) | Delegate well-specified implementation tasks to xAI's Grok Build CLI running headlessly while the orchestratin | `agentic-awesome-skills` |
| [grok-delegate](./grok-delegate) | Delegate coding tasks to the Grok Build CLI only when the user explicitly | `agentic-awesome-skills` |
| [grounded-citations](./grounded-citations) | Ground answers and documents in cited, verifiable sources. | `hermes-agent` |
| [growth-engine](./growth-engine) | Motor de crescimento para produtos digitais -- growth hacking, SEO, ASO, viral loops, email marketing, CRM, re | `agentic-awesome-skills` |
| [growth-engine-antigravity-awesome-skills-main](./growth-engine-antigravity-awesome-skills-main) | Motor de crescimento para produtos digitais -- growth hacking, SEO, ASO, viral loops, email marketing, CRM, re | `antigravity-awesome-skills-main` |
| [gtars](./gtars) | Use Gtars for local genomic interval models and set algebra, overlaps and counts, consensus and coverage, toke | `scientific-agent-skills` |
| [half-clone](./half-clone) | Clone the later half of the current conversation, discarding earlier context to reduce token usage while prese | `claude-code-tips-main` |
| [hand-drawn-diagrams](./hand-drawn-diagrams) | \| | `abingyyds__open-design` |
| [hand-drawn-diagrams-open-design](./hand-drawn-diagrams-open-design) | \| | `open-design` |
| [handoff](./handoff) | Compact the current conversation into a handoff document for another agent to pick up. | `agentic-awesome-skills` |
| [handoff-claude-code-tips-main](./handoff-claude-code-tips-main) | Write or update a handoff document so the next agent with fresh context can continue this work. | `claude-code-tips-main` |
| [happier-diagnose](./happier-diagnose) | Diagnose and explain a Happier runtime, session, daemon, provider (Claude/Codex/OpenCode), authentication, or  | `happier` |
| [happier-issue-triage](./happier-issue-triage) | Triage one or many Happier GitHub issues before deep diagnosis: retrieve the requested corpus, treat public co | `happier` |
| [happier-session-control](./happier-session-control) | Manage Happier sessions and execution runs through the CLI JSON contract, and create independent Happier diagn | `happier` |
| [harness-creator](./harness-creator) | >- | `learn-harness-engineering` |
| [harness-improvement](./harness-improvement) | Improve Kandev's AI harness from session learnings or explicit requests. Use when the user asks to record lear | `kandev` |
| [harness-mcp-scan](./harness-mcp-scan) | Static security scan of a harness's declared MCP surface via `harness mcp-scan <path>`. Reads `.mcp/servers.js | `ruflo` |
| [harness-mint](./harness-mint) | Scaffold a custom AI agent harness via `metaharness new <name> --template <id> --host <id>`. Defaults to DRY-R | `ruflo` |
| [harness-score](./harness-score) | 5-dimension harness readiness scorecard from `metaharness score <path>`. Returns harnessFit / compileConfidenc | `ruflo` |
| [health](./health) | Invoke when Claude ignores instructions, behaves inconsistently, hooks malfunction, or MCP servers need auditi | `Waza-main` |
| [healthcare-cdss-patterns](./healthcare-cdss-patterns) | Clinical Decision Support System (CDSS) development patterns. Drug interaction checking, dose validation, clin | `everything-claude-code-main` |
| [healthcare-emr-patterns](./healthcare-emr-patterns) | EMR/EHR development patterns for healthcare applications. Clinical safety, encounter workflows, prescription g | `everything-claude-code-main` |
| [healthcare-eval-harness](./healthcare-eval-harness) | Patient safety evaluation harness for healthcare application deployments. Automated test suites for CDSS accur | `everything-claude-code-main` |
| [hello-world](./hello-world) | A minimal test skill that greets the user and demonstrates the ASM publish workflow. | `luongnv89__asm` |
| [help](./help) | Analyzes what is done and the users query and offers advice on what to do next. Use if user says what should I | `Claude-Code-Game-Studios-main` |
| [helpdesk-automation](./helpdesk-automation) | Automate HelpDesk tasks via Rube MCP (Composio): list tickets, manage views, use canned responses, and configu | `agentic-awesome-skills` |
| [helpdesk-automation-antigravity-awesome-skills-main](./helpdesk-automation-antigravity-awesome-skills-main) | Automate HelpDesk tasks via Rube MCP (Composio): list tickets, manage views, use canned responses, and configu | `antigravity-awesome-skills-main` |
| [hermes](./hermes) | Multi-agent swarm orchestration. USE THIS (not delegate_task) when the user says team/swarm/multi-agent/clawte | `ClawTeam-OpenClaw` |
| [hermes-agent](./hermes-agent) | Use, configure, theme, extend, and orchestrate Hermes Agent. | `hermes-agent` |
| [hermes-agent-hermes-agent-main](./hermes-agent-hermes-agent-main) | Complete guide to using and extending Hermes Agent — CLI usage, setup, configuration, spawning additional agen | `hermes-agent-main` |
| [hermes-agent-skill-authoring](./hermes-agent-skill-authoring) | Author in-repo SKILL.md files: frontmatter and structure. | `hermes-agent` |
| [hermes-code-bridge](./hermes-code-bridge) | Use when connecting Hermes Agent to local coding CLIs such as Codex, Kimi Code, Claude Code, OpenCode, Gemini  | `hermes-code-bridge` |
| [hexagonal-architecture](./hexagonal-architecture) | Design, implement, and refactor Ports & Adapters systems with clear domain boundaries, dependency inversion, a | `everything-claude-code-main` |
| [hf-mcp](./hf-mcp) | Use Hugging Face Hub via MCP server tools. Search models, datasets, Spaces, papers. Get repo details, fetch do | `agentic-awesome-skills` |
| [hierarchical-agent-memory](./hierarchical-agent-memory) | Scoped CLAUDE.md memory system that reduces context token spend. Creates directory-level context files, tracks | `agentic-awesome-skills` |
| [hierarchical-agent-memory-antigravity-awesome-skills-main](./hierarchical-agent-memory-antigravity-awesome-skills-main) | Scoped CLAUDE.md memory system that reduces context token spend. Creates directory-level context files, tracks | `antigravity-awesome-skills-main` |
| [high-end-visual-design](./high-end-visual-design) | Use when designing expensive agency-grade interfaces with premium fonts, spatial rhythm, soft depth, and fluid | `agentic-awesome-skills` |
| [himalaya](./himalaya) | Himalaya CLI: IMAP/SMTP email from terminal. | `hermes-agent` |
| [histolab](./histolab) | Lightweight WSI tile extraction and preprocessing. Use for basic slide processing, tissue detection, tile extr | `scientific-agent-skills` |
| [history-portability](./history-portability) | > | `Claude-Code-Agent-Monitor` |
| [hive-mind](./hive-mind) | > | `ruflo` |
| [hive-mind-advanced](./hive-mind-advanced) | Advanced Hive Mind collective intelligence system for queen-led multi-agent coordination with consensus mechan | `ruflo` |
| [hive-mind-advanced-ruflo-main](./hive-mind-advanced-ruflo-main) | Advanced Hive Mind collective intelligence system for queen-led multi-agent coordination with consensus mechan | `ruflo-main` |
| [hive-mind-ruflo-main](./hive-mind-ruflo-main) | > | `ruflo-main` |
| [hono](./hono) | Use when building Hono web applications or when the user asks about Hono APIs, routing, middleware, JSX, valid | `K-Vault` |
| [hook-developer](./hook-developer) | Complete Claude Code hooks reference - input/output schemas, registration, testing patterns | `Continuous-Claude-v3-main` |
| [hook-development](./hook-development) | This skill should be used when the user asks to "create a hook", "add a PreToolUse/PostToolUse/Stop hook", "va | `claude-plugins-official-main` |
| [hook-failure-audit](./hook-failure-audit) | > | `Claude-Code-Agent-Monitor` |
| [hook-inventory](./hook-inventory) | > | `Claude-Code-Agent-Monitor` |
| [hook-setup](./hook-setup) | > | `Claude-Code-Agent-Monitor` |
| [hooks-automation](./hooks-automation) | Automated coordination, formatting, and learning from Claude Code operations using intelligent hooks with MCP  | `ruflo` |
| [hooks-automation-ruflo](./hooks-automation-ruflo) | Automated coordination, formatting, and learning from Claude Code operations using intelligent hooks with MCP  | `ruflo` |
| [hooks-automation-ruflo-main](./hooks-automation-ruflo-main) | Automated coordination, formatting, and learning from Claude Code operations using intelligent hooks with MCP  | `ruflo-main` |
| [hooks-automation-ruflo-main](./hooks-automation-ruflo-main) | Automated coordination, formatting, and learning from Claude Code operations using intelligent hooks with MCP  | `ruflo-main` |
| [hosted-agents](./hosted-agents) | This skill should be used when the user asks to "build background agent", "create hosted coding agent", "set u | `Agent-Skills-for-Context-Engineering-main` |
| [hr-pro](./hr-pro) | Professional, ethical HR partner for hiring, onboarding/offboarding, PTO and leave, performance, compliant pol | `agentic-awesome-skills` |
| [hr-pro-antigravity-awesome-skills-main](./hr-pro-antigravity-awesome-skills-main) | Professional, ethical HR partner for hiring, onboarding/offboarding, PTO and leave, performance, compliant pol | `antigravity-awesome-skills-main` |
| [hubspot-automation](./hubspot-automation) | Automate HubSpot CRM operations (contacts, companies, deals, tickets, properties) via Rube MCP using Composio  | `agentic-awesome-skills` |
| [hubspot-automation-antigravity-awesome-skills-main](./hubspot-automation-antigravity-awesome-skills-main) | Automate HubSpot CRM operations (contacts, companies, deals, tickets, properties) via Rube MCP using Composio  | `antigravity-awesome-skills-main` |
| [hugging-face-datasets](./hugging-face-datasets) | Create and manage datasets on Hugging Face Hub. Supports initializing repos, defining configs/system prompts,  | `agentic-awesome-skills` |
| [hugging-face-datasets-antigravity-awesome-skills-main](./hugging-face-datasets-antigravity-awesome-skills-main) | Create and manage datasets on Hugging Face Hub. Supports initializing repos, defining configs/system prompts,  | `antigravity-awesome-skills-main` |
| [hugging-science](./hugging-science) | Use when the user is doing AI/ML work in a scientific domain such as biology, chemistry, physics, astronomy, c | `scientific-agent-skills` |
| [huggingface-tool-builder](./huggingface-tool-builder) | Use this skill when the user wants to build tool/scripts or achieve a task where using data from the Hugging F | `agentic-awesome-skills` |
| [hugo-to-markdown](./hugo-to-markdown) | Convert Hugo documentation sites and Hugo-managed content into standard Markdown. | `agentic-awesome-skills` |
| [humanizer](./humanizer) | Humanize text: strip AI-isms and add real voice. | `hermes-agent` |
| [hunt-burp](./hunt-burp) | Drive Burp Suite over its MCP server as an AI triage + attack layer - review proxy history for signals, replay | `TORCH` |
| [hyperexecute-skill](./hyperexecute-skill) | Operates HyperExecute end-to-end for TestMu AI/LambdaTest cloud test execution: analyze projects, create YAML, | `agentic-awesome-skills` |
| [hypogenic](./hypogenic) | Plans and audits use of ChicagoHAI HypoGeniC/HypoRefine for LLM-assisted hypothesis generation from labeled te | `scientific-agent-skills` |
| [i18n-parity](./i18n-parity) | MANDATORY for every coding agent and contributor touching localized content — keep all five localization surfa | `Claude-Code-Agent-Monitor` |
| [i18n-parity-Claude-Code-Agent-Monitor](./i18n-parity-Claude-Code-Agent-Monitor) | MANDATORY for every coding agent and contributor touching localized content — keep all five localization surfa | `Claude-Code-Agent-Monitor` |
| [idea-refine](./idea-refine) | Refines raw ideas into sharp, actionable concepts through structured divergent and convergent thinking. Use wh | `agentic-awesome-skills` |
| [ii-commons](./ii-commons) | Deterministic search across arXiv, PubMed/PMC, and US policy corpora with daily freshness cutoffs. | `agentic-awesome-skills` |
| [ilya-sutskever](./ilya-sutskever) | Agente que simula Ilya Sutskever — co-fundador da OpenAI, ex-Chief Scientist, fundador da SSI. Use quando quis | `agentic-awesome-skills` |
| [ilya-sutskever-antigravity-awesome-skills-main](./ilya-sutskever-antigravity-awesome-skills-main) | Agente que simula Ilya Sutskever — co-fundador da OpenAI, ex-Chief Scientist, fundador da SSI. Use quando quis | `antigravity-awesome-skills-main` |
| [image-generation](./image-generation) | Generate or edit images from text prompts. Use when the user asks to create, draw, design, or edit an image, i | `CowAgent` |
| [image-generator](./image-generator) | Generate and edit images using Gemini's Nano Banana Pro model (gemini-3-pro-image-preview). Use this skill whe | `agentic-awesome-skills` |
| [image-studio](./image-studio) | Studio de geracao de imagens inteligente — roteamento automatico entre ai-studio-image (fotos humanizadas/infl | `agentic-awesome-skills` |
| [image-studio-antigravity-awesome-skills-main](./image-studio-antigravity-awesome-skills-main) | Studio de geracao de imagens inteligente — roteamento automatico entre ai-studio-image (fotos humanizadas/infl | `antigravity-awesome-skills-main` |
| [imagen](./imagen) | AI image generation skill powered by Google Gemini, enabling seamless visual content creation for UI placehold | `agentic-awesome-skills` |
| [imagen-antigravity-awesome-skills-main](./imagen-antigravity-awesome-skills-main) | AI image generation skill powered by Google Gemini, enabling seamless visual content creation for UI placehold | `antigravity-awesome-skills-main` |
| [imaging-data-commons](./imaging-data-commons) | Query and download public cancer imaging data from NCI Imaging Data Commons. Invoke for any question about IDC | `scientific-agent-skills` |
| [imessage](./imessage) | Send and receive iMessages/SMS via the imsg CLI on macOS. | `hermes-agent` |
| [implement_plan](./implement_plan) | Implement technical plans from thoughts/shared/plans with verification | `Continuous-Claude-v3-main` |
| [implement-in-worktree](./implement-in-worktree) | Orchestrate one implementation ticket in one isolated Git worktree with one Herdr-managed coding agent, one go | `proqi` |
| [implement-spec](./implement-spec) | Implement an existing spec through committed passes. Use for long or multi-pass specs that need maintenance ch | `skills` |
| [improve-codebase-architecture](./improve-codebase-architecture) | Scan a codebase for deepening opportunities, present them as a visual HTML report, then grill through whicheve | `agentic-awesome-skills` |
| [incremental-implementation](./incremental-implementation) | Delivers changes incrementally. Use when implementing any feature or change that touches more than one file. U | `agentic-awesome-skills` |
| [infinite-gratitude](./infinite-gratitude) | Multi-agent research skill for parallel research execution (10 agents, battle-tested with real case studies). | `agentic-awesome-skills` |
| [infinite-gratitude-antigravity-awesome-skills-main](./infinite-gratitude-antigravity-awesome-skills-main) | Multi-agent research skill for parallel research execution (10 agents, battle-tested with real case studies). | `antigravity-awesome-skills-main` |
| [infographics](./infographics) | Create professional infographics using Nano Banana Pro AI with smart iterative refinement. Uses Gemini 3.6 Fla | `scientific-agent-skills` |
| [init](./init) | Install or update OwnMem in the current repository. Use when the user asks to set up OwnMem, add local project | `ownmem` |
| [init-project](./init-project) | Initialize a new Ruflo project with MCP tools, hooks, and agent configuration. Use when setting up Ruflo in a  | `ruflo` |
| [inspecting-hermes-desktop-dom](./inspecting-hermes-desktop-dom) | Read the live Hermes desktop DOM/CSS over CDP. | `hermes-agent` |
| [instagram-automation](./instagram-automation) | Automate Instagram tasks via Rube MCP (Composio): create posts, carousels, manage media, get insights, and pub | `agentic-awesome-skills` |
| [instagram-automation-antigravity-awesome-skills-main](./instagram-automation-antigravity-awesome-skills-main) | Automate Instagram tasks via Rube MCP (Composio): create posts, carousels, manage media, get insights, and pub | `antigravity-awesome-skills-main` |
| [instructree](./instructree) | Map, explain, and lint repository-scoped coding-agent instructions before changing code. | `agentic-awesome-skills` |
| [intercom-automation](./intercom-automation) | Automate Intercom tasks via Rube MCP (Composio): conversations, contacts, companies, segments, admins. Always  | `agentic-awesome-skills` |
| [intercom-automation-antigravity-awesome-skills-main](./intercom-automation-antigravity-awesome-skills-main) | Automate Intercom tasks via Rube MCP (Composio): conversations, contacts, companies, segments, admins. Always  | `antigravity-awesome-skills-main` |
| [invariant-guard](./invariant-guard) | Correctness-first: forces writing the function contract, loop invariant, termination argument, and edge cases  | `agentic-awesome-skills` |
| [inventory-demand-planning](./inventory-demand-planning) | Codified expertise for demand forecasting, safety stock optimisation, replenishment planning, and promotional  | `agentic-awesome-skills` |
| [inventory-demand-planning-antigravity-awesome-skills-main](./inventory-demand-planning-antigravity-awesome-skills-main) | Codified expertise for demand forecasting, safety stock optimisation, replenishment planning, and promotional  | `antigravity-awesome-skills-main` |
| [inventory-demand-planning-ECC](./inventory-demand-planning-ECC) | > | `ECC` |
| [inventory-demand-planning-everything-claude-code-main](./inventory-demand-planning-everything-claude-code-main) | > | `everything-claude-code-main` |
| [investor-materials](./investor-materials) | Create and update pitch decks, one-pagers, investor memos, accelerator applications, financial models, and fun | `everything-claude-code-main` |
| [investor-outreach](./investor-outreach) | Draft cold emails, warm intro blurbs, follow-ups, update emails, and investor communications for fundraising.  | `everything-claude-code-main` |
| [ios-debugger-agent](./ios-debugger-agent) | Build, launch, inspect, and drive iOS apps with the repository-configured XcodeBuildMCP server. Use on macOS f | `t3code` |
| [iso-standards-readiness](./iso-standards-readiness) | Prepares and structurally reviews readiness evidence for ISO management-system and laboratory-competence stand | `scientific-agent-skills` |
| [iterative-retrieval](./iterative-retrieval) | Pattern for progressively refining context retrieval to solve the subagent context problem. Use when a subagen | `ECC` |
| [iterative-retrieval-everything-claude-code-main](./iterative-retrieval-everything-claude-code-main) | Pattern for progressively refining context retrieval to solve the subagent context problem | `everything-claude-code-main` |
| [ito-compute](./ito-compute) | Query live GPU inventory, submit an authenticated Itô fixed-rate RFQ, inspect RFQ or procurement status, revok | `ECC` |
| [java-coding-standards](./java-coding-standards) | Java coding standards for Spring Boot services: naming, immutability, Optional usage, streams, exceptions, gen | `everything-claude-code-main` |
| [jest-skill](./jest-skill) | Generates Jest unit and integration tests in JavaScript or TypeScript. Covers mocking, snapshots, async testin | `agentic-awesome-skills` |
| [jira-automation](./jira-automation) | Automate Jira tasks via Rube MCP (Composio): issues, projects, sprints, boards, comments, users. Always search | `agentic-awesome-skills` |
| [jira-automation-antigravity-awesome-skills-main](./jira-automation-antigravity-awesome-skills-main) | Automate Jira tasks via Rube MCP (Composio): issues, projects, sprints, boards, comments, users. Always search | `antigravity-awesome-skills-main` |
| [jira-integration](./jira-integration) | Use this skill when retrieving Jira tickets, analyzing requirements, updating ticket status, adding comments,  | `ECC` |
| [jira-integration-everything-claude-code-main](./jira-integration-everything-claude-code-main) | Use this skill when retrieving Jira tickets, analyzing requirements, updating ticket status, adding comments,  | `everything-claude-code-main` |
| [jobgpt](./jobgpt) | Job search automation, auto apply, resume generation, application tracking, salary intelligence, and recruiter | `agentic-awesome-skills` |
| [jobgpt-antigravity-awesome-skills-main](./jobgpt-antigravity-awesome-skills-main) | Job search automation, auto apply, resume generation, application tracking, salary intelligence, and recruiter | `antigravity-awesome-skills-main` |
| [jobs](./jobs) | Inspect active background research work including running processes, scheduled follow-ups, and pending tasks.  | `feynman-main` |
| [jpa-patterns](./jpa-patterns) | JPA/Hibernate patterns for entity design, relationships, query optimization, transactions, auditing, indexing, | `everything-claude-code-main` |
| [junit-5-skill](./junit-5-skill) | Generates production-grade JUnit 5 unit and integration tests in Java. Covers assertions, parameterized tests, | `agentic-awesome-skills` |
| [kiln](./kiln) | Use and maintain Kiln's AI development monorepo: Python library, | `AREX-Skill` |
| [kimi-delegate](./kimi-delegate) | Delegate coding tasks to the Kimi Code CLI (`kimi`) only when the user | `agentic-awesome-skills` |
| [klaviyo-automation](./klaviyo-automation) | Automate Klaviyo tasks via Rube MCP (Composio): manage email/SMS campaigns, inspect campaign messages, track t | `agentic-awesome-skills` |
| [klaviyo-automation-antigravity-awesome-skills-main](./klaviyo-automation-antigravity-awesome-skills-main) | Automate Klaviyo tasks via Rube MCP (Composio): manage email/SMS campaigns, inspect campaign messages, track t | `antigravity-awesome-skills-main` |
| [knowledge-ops](./knowledge-ops) | Knowledge base management, ingestion, sync, and retrieval across multiple storage layers (local files, MCP mem | `everything-claude-code-main` |
| [knowledge-wiki](./knowledge-wiki) | Manage the personal knowledge wiki. Use when the user shares articles, documents, or asks to organize knowledg | `CowAgent` |
| [kobe](./kobe) | Use when controlling Rove tasks, parallel coding attempts, hosted agent sessions, task lifecycle, or the daemo | `rove` |
| [kotlin-coroutines-flows](./kotlin-coroutines-flows) | Kotlin Coroutines and Flow patterns for Android and KMP — structured concurrency, Flow operators, StateFlow, e | `everything-claude-code-main` |
| [kotlin-patterns](./kotlin-patterns) | Idiomatic Kotlin patterns, best practices, and conventions for building robust, efficient, and maintainable Ko | `everything-claude-code-main` |
| [kotlin-testing](./kotlin-testing) | Kotlin testing patterns with Kotest, MockK, coroutine testing, property-based testing, and Kover coverage. Fol | `everything-claude-code-main` |
| [kubestellar-console](./kubestellar-console) | Multi-cluster Kubernetes dashboard with AI-powered operations via MCP server and 10+ built-in agent skills | `agentic-awesome-skills` |
| [la-vague](./la-vague) | Operate LaVague browser-agent, model-context, driver, | `AREX-Skill` |
| [lab-hardware-cad](./lab-hardware-cad) | Design custom laboratory hardware as parametric build123d models and export fabrication-ready STEP, STL, and D | `scientific-agent-skills` |
| [labarchive-integration](./labarchive-integration) | Securely integrate with the official LabArchives ELN REST-like API and Inventory API v1. Use for regional endp | `scientific-agent-skills` |
| [lambda-lang](./lambda-lang) | Native agent-to-agent language for compact multi-agent messaging. A shared tongue agents speak directly, not a | `agentic-awesome-skills` |
| [lambdatest-agent-skills](./lambdatest-agent-skills) | Production-grade test automation skills for 46 frameworks across E2E, unit, mobile, BDD, visual, and cloud tes | `agentic-awesome-skills` |
| [langchain-architecture](./langchain-architecture) | Design LLM applications using LangChain 1.x and LangGraph for agents, memory, and tool integration. Use when b | `agents-main` |
| [langgraph](./langgraph) | Expert in LangGraph - the production-grade framework for building | `agentic-awesome-skills` |
| [langgraph-antigravity-awesome-skills-main](./langgraph-antigravity-awesome-skills-main) | Expert in LangGraph - the production-grade framework for building | `antigravity-awesome-skills-main` |
| [laravel-patterns](./laravel-patterns) | Laravel architecture patterns, routing/controllers, Eloquent ORM, service layers, queues, events, caching, and | `everything-claude-code-main` |
| [laravel-plugin-discovery](./laravel-plugin-discovery) | Discover and evaluate Laravel packages via LaraPlugins.io MCP. Use when the user wants to find plugins, check  | `ECC` |
| [laravel-plugin-discovery-everything-claude-code-main](./laravel-plugin-discovery-everything-claude-code-main) | Discover and evaluate Laravel packages via LaraPlugins.io MCP. Use when the user wants to find plugins, check  | `everything-claude-code-main` |
| [laravel-tdd](./laravel-tdd) | Test-driven development for Laravel with PHPUnit and Pest, factories, database testing, fakes, and coverage ta | `everything-claude-code-main` |
| [laravel-verification](./laravel-verification) | Verification loop for Laravel projects: env checks, linting, static analysis, tests with coverage, security sc | `everything-claude-code-main` |
| [latchbio-integration](./latchbio-integration) | Build, register, debug, and operate bioinformatics workflows on Latch using the Python SDK, CLI, Latch Data an | `scientific-agent-skills` |
| [launch-checklist](./launch-checklist) | Complete launch readiness validation covering every department: code, content, store, marketing, community, in | `Claude-Code-Game-Studios-main` |
| [lazy-llm](./lazy-llm) | Guides LazyLLM low-code LLM application workflows including | `AREX-Skill` |
| [lean-ctx-review](./lean-ctx-review) | Review how the lean-ctx ctx_* MCP tools performed in the current session and file upstream issues for confirme | `lean-ctx` |
| [leann-search](./leann-search) | Semantic search across codebase using LEANN vector index | `Continuous-Claude-v3-main` |
| [learn](./learn) | Help a user learn a topic through adaptive tutoring, lesson planning, practice, retrieval checks, explanations | `agentic-awesome-skills` |
| [leiloeiro-avaliacao](./leiloeiro-avaliacao) | Avaliacao pericial de imoveis em leilao. Valor de mercado, liquidacao forcada, ABNT NBR 14653, metodos compara | `agentic-awesome-skills` |
| [leiloeiro-avaliacao-antigravity-awesome-skills-main](./leiloeiro-avaliacao-antigravity-awesome-skills-main) | Avaliacao pericial de imoveis em leilao. Valor de mercado, liquidacao forcada, ABNT NBR 14653, metodos compara | `antigravity-awesome-skills-main` |
| [leiloeiro-edital](./leiloeiro-edital) | Analise e auditoria de editais de leilao judicial e extrajudicial. Riscos ocultos, clausulas perigosas, debito | `agentic-awesome-skills` |
| [leiloeiro-edital-antigravity-awesome-skills-main](./leiloeiro-edital-antigravity-awesome-skills-main) | Analise e auditoria de editais de leilao judicial e extrajudicial. Riscos ocultos, clausulas perigosas, debito | `antigravity-awesome-skills-main` |
| [leiloeiro-ia](./leiloeiro-ia) | Especialista em leiloes judiciais e extrajudiciais de imoveis. Analise juridica, pericial e de mercado integra | `agentic-awesome-skills` |
| [leiloeiro-ia-antigravity-awesome-skills-main](./leiloeiro-ia-antigravity-awesome-skills-main) | Especialista em leiloes judiciais e extrajudiciais de imoveis. Analise juridica, pericial e de mercado integra | `antigravity-awesome-skills-main` |
| [leiloeiro-juridico](./leiloeiro-juridico) | Analise juridica de leiloes: nulidades, bem de familia, alienacao fiduciaria, CPC arts 829-903, Lei 9514/97, o | `agentic-awesome-skills` |
| [leiloeiro-juridico-antigravity-awesome-skills-main](./leiloeiro-juridico-antigravity-awesome-skills-main) | Analise juridica de leiloes: nulidades, bem de familia, alienacao fiduciaria, CPC arts 829-903, Lei 9514/97, o | `antigravity-awesome-skills-main` |
| [leiloeiro-mercado](./leiloeiro-mercado) | Analise de mercado imobiliario para leiloes. Liquidez, desagio tipico, ROI, estrategias de saida (flip/reforma | `agentic-awesome-skills` |
| [leiloeiro-mercado-antigravity-awesome-skills-main](./leiloeiro-mercado-antigravity-awesome-skills-main) | Analise de mercado imobiliario para leiloes. Liquidez, desagio tipico, ROI, estrategias de saida (flip/reforma | `antigravity-awesome-skills-main` |
| [leiloeiro-risco](./leiloeiro-risco) | Analise de risco em leiloes de imoveis. Score 36 pontos, riscos juridicos/financeiros/operacionais, stress tes | `agentic-awesome-skills` |
| [leiloeiro-risco-antigravity-awesome-skills-main](./leiloeiro-risco-antigravity-awesome-skills-main) | Analise de risco em leiloes de imoveis. Score 36 pontos, riscos juridicos/financeiros/operacionais, stress tes | `antigravity-awesome-skills-main` |
| [lemmaly](./lemmaly) | Algorithm-first discipline: state Big-O, data structure, and algorithm family BEFORE writing loops, queries, o | `agentic-awesome-skills` |
| [lesson-generator](./lesson-generator) | Build compact, standalone multi-lesson course artifacts with lesson navigation, objectives, flashcards, quizze | `agentic-awesome-skills` |
| [lieflat-charts](./lieflat-charts) | 一套模板驱动的数据可视化与报告生成 skill，既能严格从 Lupi、Basics、Glance、Maps 与 Interactive gallery 的真实实现生成 HTML 图表，也能从 12 套中英文整页报告模板生 | `lieflat-charts` |
| [linear](./linear) | Manage Linear issues, projects, and teams via the GraphQL API. Create, update, search, and organize issues. Us | `hermes-agent-main` |
| [linear-automation](./linear-automation) | Automate Linear tasks via Rube MCP (Composio): issues, projects, cycles, teams, labels. Always search tools fi | `agentic-awesome-skills` |
| [linear-automation-antigravity-awesome-skills-main](./linear-automation-antigravity-awesome-skills-main) | Automate Linear tasks via Rube MCP (Composio): issues, projects, cycles, teams, labels. Always search tools fi | `antigravity-awesome-skills-main` |
| [linkedin-automation](./linkedin-automation) | Automate LinkedIn tasks via Rube MCP (Composio): create posts, manage profile, company info, comments, and ima | `agentic-awesome-skills` |
| [linkedin-automation-antigravity-awesome-skills-main](./linkedin-automation-antigravity-awesome-skills-main) | Automate LinkedIn tasks via Rube MCP (Composio): create posts, manage profile, company info, comments, and ima | `antigravity-awesome-skills-main` |
| [liquid-glass-design](./liquid-glass-design) | iOS 26 Liquid Glass design system — dynamic glass material with blur, reflection, and interactive morphing for | `everything-claude-code-main` |
| [liteparse](./liteparse) | Local document and PDF parsing that returns spatial text with bounding boxes. Use for extracting text from PDF | `scientific-agent-skills` |
| [literature-review](./literature-review) | Conduct comprehensive, systematic literature reviews using multiple academic databases (PubMed, arXiv, bioRxiv | `scientific-agent-skills` |
| [live-watch](./live-watch) | > | `Claude-Code-Agent-Monitor` |
| [llm-app-patterns](./llm-app-patterns) | Architecture and integration sketches for LLM applications, with explicit retrieval, tool, privacy and verific | `agentic-awesome-skills` |
| [llm-app-patterns-antigravity-awesome-skills-main](./llm-app-patterns-antigravity-awesome-skills-main) | Production-ready patterns for building LLM applications, inspired by [Dify](https://github.com/langgenius/dify | `antigravity-awesome-skills-main` |
| [llm-council](./llm-council) | Run Fireworks-hosted open-weight model councils that compare responses and synthesize a final answer. | `agentic-awesome-skills` |
| [llm-council-codex-skills-main](./llm-council-codex-skills-main) | > | `codex-skills-main` |
| [llm-gate](./llm-gate) | LLM-powered quality verification using prompt hooks. Validates commit messages, code patterns, and conventions | `pro-workflow` |
| [llm-ops](./llm-ops) | LLM Operations -- RAG, embeddings, vector databases, fine-tuning, prompt engineering avancado, custos de LLM,  | `agentic-awesome-skills` |
| [llm-ops-antigravity-awesome-skills-main](./llm-ops-antigravity-awesome-skills-main) | LLM Operations -- RAG, embeddings, vector databases, fine-tuning, prompt engineering avancado, custos de LLM,  | `antigravity-awesome-skills-main` |
| [llm-wiki](./llm-wiki) | Karpathy's LLM Wiki: build/query interlinked markdown KB. | `hermes-agent` |
| [lmops](./lmops) | Route LMOps paper-code workflows for prompt optimization, | `AREX-Skill` |
| [localize](./localize) | Full localization pipeline: scan for hardcoded strings, extract and manage string tables, validate translation | `Claude-Code-Game-Studios-main` |
| [logistics-exception-management](./logistics-exception-management) | Codified expertise for handling freight exceptions, shipment delays, damages, losses, and carrier disputes. In | `agentic-awesome-skills` |
| [logistics-exception-management-antigravity-awesome-skills-main](./logistics-exception-management-antigravity-awesome-skills-main) | Codified expertise for handling freight exceptions, shipment delays, damages, losses, and carrier disputes. In | `antigravity-awesome-skills-main` |
| [logistics-exception-management-ECC](./logistics-exception-management-ECC) | > | `ECC` |
| [logistics-exception-management-everything-claude-code-main](./logistics-exception-management-everything-claude-code-main) | > | `everything-claude-code-main` |
| [loki-mode](./loki-mode) | Version 2.35.0 \| PRD to Production \| Zero Human Intervention > Research-enhanced: OpenAI SDK, DeepMind, Anth | `agentic-awesome-skills` |
| [loki-mode-antigravity-awesome-skills-main](./loki-mode-antigravity-awesome-skills-main) | Version 2.35.0 \| PRD to Production \| Zero Human Intervention > Research-enhanced: OpenAI SDK, DeepMind, Anth | `antigravity-awesome-skills-main` |
| [longbridge-content](./longbridge-content) | Latest news articles, regulatory filings, community discussion topics for listed stocks, and SEC EDGAR filing  | `agentic-awesome-skills` |
| [longbridge-fundamentals](./longbridge-fundamentals) | Financial statements, business segments, dividends, valuation multiples (PE/PB/PS), industry comparison, opera | `agentic-awesome-skills` |
| [longbridge-market-data](./longbridge-market-data) | Real-time quotes, K-line charts, order book, trade ticks, intraday capital flow, market sentiment temperature, | `agentic-awesome-skills` |
| [lookdev](./lookdev) | Human-in-the-loop web studio to tune AI-generated output by eye. Stand up a local interactive studio (sliders, | `agentic-awesome-skills` |
| [lookdev-auto](./lookdev-auto) | Automated visual tuning: a vision or video model rates rendered variants in a loop. Render several labeled var | `agentic-awesome-skills` |
| [loop-library](./loop-library) | Find, compare, adapt, and design bounded AI-agent feedback loops with explicit checks, stop rules, guardrails, | `agentic-awesome-skills` |
| [loop-worker](./loop-worker) | Run Ruflo background workers using Claude Code native /loop scheduling | `ruflo` |
| [loopx-change-quality](./loopx-change-quality) | Qualify the exact final diff for a LoopX-managed goal. Use when goal policy enables change_quality_qualificati | `loopx` |
| [loopx-material](./loopx-material) | Operate an explicitly activated LoopX Material Lifecycle for a connected project. Use for material-store inven | `loopx` |
| [loopx-project](./loopx-project) | Use when connecting a repository or project goal document to LoopX, maintaining project-local goal state, refr | `loopx` |
| [lore](./lore) | Markdown project memory for AI agents. Use for decisions, architecture, conventions, monorepo scopes, `.lore/` | `agentic-awesome-skills` |
| [m-flow](./m-flow) | Operate M-flow memory, graph-routed retrieval, ingestion | `AREX-Skill` |
| [machine-learning-ops-ml-pipeline](./machine-learning-ops-ml-pipeline) | Design and implement a complete ML pipeline for: $ARGUMENTS | `agentic-awesome-skills` |
| [machine-learning-ops-ml-pipeline-antigravity-awesome-skills-main](./machine-learning-ops-ml-pipeline-antigravity-awesome-skills-main) | Design and implement a complete ML pipeline for: $ARGUMENTS | `antigravity-awesome-skills-main` |
| [magic-ui-generator](./magic-ui-generator) | Utilizes Magic by 21st.dev to generate, compare, and integrate multiple production-ready UI component variatio | `agentic-awesome-skills` |
| [magic-ui-generator-antigravity-awesome-skills-main](./magic-ui-generator-antigravity-awesome-skills-main) | Leverage [Magic by 21st.dev](https://21st.dev/magic) to build modern, responsive UI components using an AI-nat | `antigravity-awesome-skills-main` |
| [mailchimp-automation](./mailchimp-automation) | Automate Mailchimp email marketing including campaigns, audiences, subscribers, segments, and analytics via Ru | `agentic-awesome-skills` |
| [mailchimp-automation-antigravity-awesome-skills-main](./mailchimp-automation-antigravity-awesome-skills-main) | Automate Mailchimp email marketing including campaigns, audiences, subscribers, segments, and analytics via Ru | `antigravity-awesome-skills-main` |
| [make](./make) | Use when operating Make.com (formerly Integromat) programmatically — driving its REST API v2 or the Make MCP s | `ericrisco__rsc-harness` |
| [make-automation](./make-automation) | Automate Make (Integromat) tasks via Rube MCP (Composio): operations, enums, language and timezone lookups. Al | `agentic-awesome-skills` |
| [make-automation-antigravity-awesome-skills-main](./make-automation-antigravity-awesome-skills-main) | Automate Make (Integromat) tasks via Rube MCP (Composio): operations, enums, language and timezone lookups. Al | `antigravity-awesome-skills-main` |
| [manage-skills](./manage-skills) | Manage the user's shared agent-skill library via skills-manager-cli — install, update, remove, deploy or undep | `skills-manager` |
| [managed-agent](./managed-agent) | Run an Anthropic Claude Managed Agent — a cloud agent harness (container + filesystem + tools), the cloud coun | `ruflo` |
| [manim-video](./manim-video) | Build reusable Manim explainers for technical concepts, graphs, system diagrams, and product walkthroughs, the | `everything-claude-code-main` |
| [manim-video-hermes-agent](./manim-video-hermes-agent) | Manim CE animations: 3Blue1Brown math/algo videos. | `hermes-agent` |
| [map-systems](./map-systems) | Decompose a game concept into individual systems, map dependencies, prioritize design order, and create the sy | `Claude-Code-Game-Studios-main` |
| [maps](./maps) | Geocode, POIs, routes, timezones via OpenStreetMap/OSRM. | `hermes-agent` |
| [markdown-mermaid-writing](./markdown-mermaid-writing) | Comprehensive markdown and Mermaid diagram writing skill. Use when creating any scientific document, report, a | `scientific-agent-skills` |
| [market-analysis](./market-analysis) | 市场分析报告（大盘快照版）— 复用市场分析页面全部数据能力：核心指数、市场广度与情绪温度、行业板块/热门概念热力图、行业多日涨跌幅对比（1/3/5日）、板块资金流（1日/5日/10日）、个股主力资金 Top20、标签体系 | `QuantMind` |
| [market-research](./market-research) | Conduct market research, competitive analysis, investor due diligence, and industry intelligence with source a | `everything-claude-code-main` |
| [markitdown](./markitdown) | Use MarkItDown to convert documents to Markdown, configure | `AREX-Skill` |
| [markitdown-scientific-agent-skills](./markitdown-scientific-agent-skills) | Convert heterogeneous documents and selected URIs to Markdown with Microsoft MarkItDown for text analysis, sea | `scientific-agent-skills` |
| [matchms](./matchms) | Process, clean, compare, and search tandem mass spectra with matchms. Use for MS/MS file I/O, metadata harmoni | `scientific-agent-skills` |
| [matematico-tao](./matematico-tao) | Matemático ultra-avançado inspirado em Terence Tao. Análise rigorosa de código e arquitetura com teoria matemá | `agentic-awesome-skills` |
| [matematico-tao-antigravity-awesome-skills-main](./matematico-tao-antigravity-awesome-skills-main) | Matemático ultra-avançado inspirado em Terence Tao. Análise rigorosa de código e arquitetura com teoria matemá | `antigravity-awesome-skills-main` |
| [matlab](./matlab) | Build, review, migrate, and safely plan MATLAB or GNU Octave numerical workflows, including arrays, tabular/ti | `scientific-agent-skills` |
| [matplotlib](./matplotlib) | Matplotlib is Python's foundational visualization library for creating static, animated, and interactive plots | `agentic-awesome-skills` |
| [matplotlib-antigravity-awesome-skills-main](./matplotlib-antigravity-awesome-skills-main) | Matplotlib is Python's foundational visualization library for creating static, animated, and interactive plots | `antigravity-awesome-skills-main` |
| [matplotlib-scientific-agent-skills](./matplotlib-scientific-agent-skills) | Low-level plotting library for full customization. Use when you need fine-grained control over every plot elem | `scientific-agent-skills` |
| [mcp](./mcp) | MCP Apps integration for json-render. Use when building MCP servers that render interactive UIs in Claude, Cha | `vercel-labs__json-render` |
| [mcp-agent](./mcp-agent) | Build, compose, serve, operate, and troubleshoot mcp-agent | `AREX-Skill` |
| [mcp-audit](./mcp-audit) | Audit connected MCP servers for token overhead, redundancy, and security. Use when sessions feel slow or befor | `pro-workflow` |
| [mcp-builder](./mcp-builder) | Create MCP (Model Context Protocol) servers that enable LLMs to interact with external services through well-d | `agentic-awesome-skills` |
| [mcp-builder-aiox-core-main](./mcp-builder-aiox-core-main) | Guide for creating high-quality MCP (Model Context Protocol) servers that enable LLMs to interact with externa | `aiox-core-main` |
| [mcp-builder-anthropics__skills](./mcp-builder-anthropics__skills) | Guide for creating high-quality MCP (Model Context Protocol) servers that enable LLMs to interact with externa | `anthropics__skills` |
| [mcp-builder-antigravity-awesome-skills-main](./mcp-builder-antigravity-awesome-skills-main) | Create MCP (Model Context Protocol) servers that enable LLMs to interact with external services through well-d | `antigravity-awesome-skills-main` |
| [mcp-builder-librarium](./mcp-builder-librarium) | Guide for creating high-quality MCP (Model Context Protocol) servers that enable LLMs to interact with externa | `librarium` |
| [mcp-builder-ms](./mcp-builder-ms) | Use this skill when building MCP servers to integrate external APIs or services, whether in Python (FastMCP) o | `agentic-awesome-skills` |
| [mcp-builder-ms-antigravity-awesome-skills-main](./mcp-builder-ms-antigravity-awesome-skills-main) | Use this skill when building MCP servers to integrate external APIs or services, whether in Python (FastMCP) o | `antigravity-awesome-skills-main` |
| [mcp-builder-shareAI-lab__learn-claude-code](./mcp-builder-shareAI-lab__learn-claude-code) | Build MCP (Model Context Protocol) servers that give Claude new capabilities. Use when user wants to create an | `shareAI-lab__learn-claude-code` |
| [mcp-chaining](./mcp-chaining) | Research-to-implement pipeline chaining 5 MCP tools with graceful degradation | `Continuous-Claude-v3-main` |
| [mcp-client](./mcp-client) | Universal MCP client for connecting to any MCP server with progressive disclosure. Wraps MCP servers as skills | `coleam00__second-brain-skills` |
| [mcp-context-forge](./mcp-context-forge) | Operate ContextForge (mcp-context-forge), the FastAPI | `AREX-Skill` |
| [mcp-maintainer](./mcp-maintainer) | Operate and maintain the local MCP server for this repository. Use for MCP tool updates, policy-guard changes, | `Claude-Code-Agent-Monitor` |
| [mcp-operations](./mcp-operations) | Operate and maintain the local MCP server for this project. Use when creating MCP host config, troubleshooting | `Claude-Code-Agent-Monitor` |
| [mcp-server](./mcp-server) | > | `Claude-Code-Agent-Monitor` |
| [mcp-server-patterns](./mcp-server-patterns) | Build MCP servers with Node/TypeScript SDK — tools, resources, prompts, Zod validation, stdio vs Streamable HT | `ECC` |
| [mcp-server-patterns-ECC](./mcp-server-patterns-ECC) | Build MCP servers with Node/TypeScript SDK — tools, resources, prompts, Zod validation, stdio vs Streamable HT | `ECC` |
| [mcp-server-patterns-everything-claude-code-main](./mcp-server-patterns-everything-claude-code-main) | Build MCP servers with Node/TypeScript SDK — tools, resources, prompts, Zod validation, stdio vs Streamable HT | `everything-claude-code-main` |
| [mcp-server-patterns-everything-claude-code-main](./mcp-server-patterns-everything-claude-code-main) | Build MCP servers with Node/TypeScript SDK — tools, resources, prompts, Zod validation, stdio vs Streamable HT | `everything-claude-code-main` |
| [mcp-tool-developer](./mcp-tool-developer) | Build Model Context Protocol (MCP) servers and tools from scratch. Full-stack MCP development with TypeScript/ | `agentic-awesome-skills` |
| [mcporter](./mcporter) | Use the mcporter CLI to list, configure, auth, and call MCP servers/tools directly (HTTP or stdio), including  | `hermes-agent-main` |
| [mcporter-openclaw](./mcporter-openclaw) | List, configure, authenticate, call, and inspect MCP servers/tools with mcporter over HTTP or stdio. | `openclaw` |
| [mcporter-understudy-ai__understudy](./mcporter-understudy-ai__understudy) | Use the mcporter CLI to list, configure, auth, and call MCP servers/tools directly (HTTP or stdio), including  | `understudy-ai__understudy` |
| [medchem](./medchem) | Medicinal chemistry filters for compound triage. Apply drug-likeness rules (Lipinski, Veber, CNS), structural  | `scientific-agent-skills` |
| [medrax](./medrax) | Use MedRAX for chest-X-ray reasoning workflows, selective | `AREX-Skill` |
| [memory-bridge](./memory-bridge) | Bridge Claude Code auto-memory into AgentDB with ONNX embeddings, deduplicate, and enable unified cross-projec | `ruflo` |
| [memory-review](./memory-review) | > | `Claude-Code-Agent-Monitor` |
| [memory-systems](./memory-systems) | > | `Agent-Skills-for-Context-Engineering-main` |
| [memory-systems-agentic-awesome-skills](./memory-systems-agentic-awesome-skills) | Design short-term, long-term, and graph-based memory architectures. Use when building agents that must persist | `agentic-awesome-skills` |
| [memory-systems-antigravity-awesome-skills-main](./memory-systems-antigravity-awesome-skills-main) | Design short-term, long-term, and graph-based memory architectures. Use when building agents that must persist | `antigravity-awesome-skills-main` |
| [mesh-memory](./mesh-memory) | Self-hosted semantic memory for AI agents via MCP. Save worklogs, decisions, and notes, then recall them acros | `agentic-awesome-skills` |
| [meta-gpt](./meta-gpt) | Use MetaGPT for multi-agent software-company workflows, Data | `AREX-Skill` |
| [microsoft-teams-automation](./microsoft-teams-automation) | Automate Microsoft Teams tasks via Rube MCP (Composio): send messages, manage channels, create meetings, handl | `agentic-awesome-skills` |
| [microsoft-teams-automation-antigravity-awesome-skills-main](./microsoft-teams-automation-antigravity-awesome-skills-main) | Automate Microsoft Teams tasks via Rube MCP (Composio): send messages, manage channels, create meetings, handl | `antigravity-awesome-skills-main` |
| [milestone-review](./milestone-review) | Generates a comprehensive milestone progress review including feature completeness, quality metrics, risk asse | `Claude-Code-Game-Studios-main` |
| [minion-orchestrator](./minion-orchestrator) | \| | `gbrain` |
| [mintlify](./mintlify) | Build and maintain documentation sites with Mintlify. Use when | `FailproofAI__failproofai` |
| [miro-automation](./miro-automation) | Automate Miro tasks via Rube MCP (Composio): boards, items, sticky notes, frames, sharing, connectors. Always  | `agentic-awesome-skills` |
| [miro-automation-antigravity-awesome-skills-main](./miro-automation-antigravity-awesome-skills-main) | Automate Miro tasks via Rube MCP (Composio): boards, items, sticky notes, frames, sharing, connectors. Always  | `antigravity-awesome-skills-main` |
| [mixpanel-automation](./mixpanel-automation) | Automate Mixpanel tasks via Rube MCP (Composio): events, segmentation, funnels, cohorts, user profiles, JQL qu | `agentic-awesome-skills` |
| [mixpanel-automation-antigravity-awesome-skills-main](./mixpanel-automation-antigravity-awesome-skills-main) | Automate Mixpanel tasks via Rube MCP (Composio): events, segmentation, funnels, cohorts, user profiles, JQL qu | `antigravity-awesome-skills-main` |
| [mkl-review-source-change](./mkl-review-source-change) | Review an upstream documentation change against the skills, agent instructions, or runbooks that cite it. Iden | `00200200__maintainer-skills-lab` |
| [mmx-cli](./mmx-cli) | Use mmx to generate text, images, video, speech, and music via the MiniMax AI platform. Use when the user want | `agentic-awesome-skills` |
| [modal](./modal) | Modal is a serverless cloud platform for running Python on demand, including on-demand GPUs. Use when deployin | `scientific-agent-skills` |
| [model-mix](./model-mix) | > | `Claude-Code-Agent-Monitor` |
| [model-savings](./model-savings) | > | `Claude-Code-Agent-Monitor` |
| [model-train-infer-backtest-report](./model-train-infer-backtest-report) | 模型训练-推理-组合回测-专业报告 全流程 — 提交T+N周期模型训练（13种模型类型：lightgbm/xgboost/catboost/random_forest/linear/mlp/gru/lstm/alstm/ | `QuantMind` |
| [molecular-dynamics](./molecular-dynamics) | Run and analyze molecular dynamics simulations with OpenMM and MDAnalysis. Set up protein/small molecule syste | `scientific-agent-skills` |
| [molfeat](./molfeat) | Molecular featurization for ML (100+ featurizers). ECFP, MACCS, descriptors, pretrained models (ChemBERTa), co | `scientific-agent-skills` |
| [monday-automation](./monday-automation) | Automate Monday.com work management including boards, items, columns, groups, subitems, and updates via Rube M | `agentic-awesome-skills` |
| [monday-automation-antigravity-awesome-skills-main](./monday-automation-antigravity-awesome-skills-main) | Automate Monday.com work management including boards, items, columns, groups, subitems, and updates via Rube M | `antigravity-awesome-skills-main` |
| [monetization](./monetization) | Estrategia e implementacao de monetizacao para produtos digitais - Stripe, subscriptions, pricing experiments, | `agentic-awesome-skills` |
| [monetization-antigravity-awesome-skills-main](./monetization-antigravity-awesome-skills-main) | Estrategia e implementacao de monetizacao para produtos digitais - Stripe, subscriptions, pricing experiments, | `antigravity-awesome-skills-main` |
| [monorepo-architect](./monorepo-architect) | Expert in monorepo architecture, build systems, and dependency management at scale. Masters Nx, Turborepo, Baz | `agentic-awesome-skills` |
| [monte-carlo-prevent](./monte-carlo-prevent) | Surfaces Monte Carlo data observability context (table health, alerts, lineage, blast radius) before SQL/dbt e | `agentic-awesome-skills` |
| [monthly-review](./monthly-review) | > | `Claude-Code-Agent-Monitor` |
| [mot](./mot) | System health check (MOT) for skills, agents, hooks, and memory | `Continuous-Claude-v3-main` |
| [multi-advisor](./multi-advisor) | Conselho de especialistas — consulta multiplos agentes do ecossistema em paralelo para analise multi-perspecti | `agentic-awesome-skills` |
| [multi-advisor-antigravity-awesome-skills-main](./multi-advisor-antigravity-awesome-skills-main) | Conselho de especialistas — consulta multiplos agentes do ecossistema em paralelo para analise multi-perspecti | `antigravity-awesome-skills-main` |
| [multi-agent-architect](./multi-agent-architect) | Design and optimize production-grade multi-agent systems with LangGraph, LangChain, and DeepAgents for complex | `agentic-awesome-skills` |
| [multi-agent-brainstorming](./multi-agent-brainstorming) | Simulate a structured peer-review process using multiple specialized agents to validate designs, surface hidde | `agentic-awesome-skills` |
| [multi-agent-brainstorming-antigravity-awesome-skills-main](./multi-agent-brainstorming-antigravity-awesome-skills-main) | Simulate a structured peer-review process using multiple specialized agents to validate designs, surface hidde | `antigravity-awesome-skills-main` |
| [multi-agent-patterns](./multi-agent-patterns) | This skill should be used when the user asks to "design multi-agent system", "implement supervisor pattern", " | `Agent-Skills-for-Context-Engineering-main` |
| [multi-agent-patterns-agentic-awesome-skills](./multi-agent-patterns-agentic-awesome-skills) | This skill should be used when the user asks to "design multi-agent system", "implement supervisor pattern", " | `agentic-awesome-skills` |
| [multi-agent-patterns-antigravity-awesome-skills-main](./multi-agent-patterns-antigravity-awesome-skills-main) | This skill should be used when the user asks to "design multi-agent system", "implement supervisor pattern", " | `antigravity-awesome-skills-main` |
| [multi-agent-task-orchestrator](./multi-agent-task-orchestrator) | Route tasks to specialized AI agents with anti-duplication, quality gates, and 30-minute heartbeat monitoring | `agentic-awesome-skills` |
| [multi-agent-task-orchestrator-antigravity-awesome-skills-main](./multi-agent-task-orchestrator-antigravity-awesome-skills-main) | Route tasks to specialized AI agents with anti-duplication, quality gates, and 30-minute heartbeat monitoring | `antigravity-awesome-skills-main` |
| [music-connect](./music-connect) | One-time setup — mint a Cognitum Music personal access token and register the cogmusic MCP server with Claude  | `ruflo` |
| [muzic](./muzic) | Route Microsoft Muzic research workflows for music understanding, | `AREX-Skill` |
| [n8n](./n8n) | Use when operating a live n8n instance programmatically — its REST API (`{host}/api/v1`, `X-N8N-API-KEY`) or t | `ericrisco__rsc-harness` |
| [n8n-mcp-tools-expert](./n8n-mcp-tools-expert) | Expert guide for using n8n-mcp MCP tools effectively. Use when searching for nodes, validating configurations, | `agentic-awesome-skills` |
| [n8n-mcp-tools-expert-antigravity-awesome-skills-main](./n8n-mcp-tools-expert-antigravity-awesome-skills-main) | Expert guide for using n8n-mcp MCP tools effectively. Use when searching for nodes, validating configurations, | `antigravity-awesome-skills-main` |
| [nanoclaw-repl](./nanoclaw-repl) | Operate and extend NanoClaw v2, ECC's zero-dependency session-aware REPL built on claude -p. | `everything-claude-code-main` |
| [ncats-arax](./ncats-arax) | Queries the NCATS Translator ARAX production API for bounded, typed, provenance-rich one-hop and endpoint-pinn | `scientific-agent-skills` |
| [neon-ai-gateway](./neon-ai-gateway) | One API and one credential for frontier and open-source LLMs, built into your Neon branch and powered by Datab | `agentic-awesome-skills` |
| [neon-functions](./neon-functions) | Long-running, serverless Node.js HTTP functions deployed onto your Neon branch, with DATABASE_URL injected aut | `agentic-awesome-skills` |
| [neon-object-storage](./neon-object-storage) | S3-compatible object storage that branches with your Neon project, so files and the database stay in sync acro | `agentic-awesome-skills` |
| [neon-postgres](./neon-postgres) | Guides and best practices for working with Neon Serverless Postgres. Covers setup, connection methods, branchi | `agentic-awesome-skills` |
| [neon-postgres-branches](./neon-postgres-branches) | Choose and create the right Neon branch type for testing and development. Use when users ask about Neon branch | `agentic-awesome-skills` |
| [neon-postgres-egress-optimizer](./neon-postgres-egress-optimizer) | Diagnose and fix excessive Postgres egress (network data transfer) in a codebase. | `agentic-awesome-skills` |
| [nested-subagents](./nested-subagents) | Spawn nested sub-agents (agents that spawn sub-agents, up to depth=5) via Claude Code's native Task tool — for | `ruflo` |
| [nestjs-patterns](./nestjs-patterns) | NestJS architecture patterns for modules, controllers, providers, DTO validation, guards, interceptors, config | `everything-claude-code-main` |
| [neurokit2](./neurokit2) | Use NeuroKit2 to build or audit reproducible research workflows for physiological time-series preprocessing, e | `scientific-agent-skills` |
| [neuropixels-analysis](./neuropixels-analysis) | Analyze Neuropixels extracellular recordings end-to-end with SpikeInterface. Covers loading SpikeGLX/Open Ephy | `scientific-agent-skills` |
| [new-command-docs](./new-command-docs) | This skill should be used when a new ArcKit command has been added and documentation needs updating across the | `arc-kit-main` |
| [newman-cicd-integration](./newman-cicd-integration) | Generate ready-to-use CI/CD pipeline configurations that install and run Newman for automated API testing. | `agentic-awesome-skills` |
| [news-sentiment-finbert](./news-sentiment-finbert) | RSS 新闻情绪识别（FinBERT 中文金融情感）安装与运维 — 情绪管线架构、transformers 安装、FinBERT 权重下载、字典法扩充、全量重算、情绪筛选/条形图/个股资讯标签的使用。在 QuantBot | `QuantMind` |
| [news-sentiment-research](./news-sentiment-research) | 新闻情绪研究方法论 — Huntly RSS 42万篇历史新闻 → FinBERT+词典双引擎情绪 → 事件研究/来源预测力/时段特征/信号强度/首日动量/情绪反转/事件标签 七维深度分析 → 融合规律优化策略回测（来源 | `QuantMind` |
| [nextflow](./nextflow) | Build, run, and debug Nextflow data pipelines and nf-core workflows end to end. Use whenever the user mentions | `scientific-agent-skills` |
| [nextjs-seo-indexing](./nextjs-seo-indexing) | Fix SEO indexing issues, crawl budget problems, and Search Console coverage errors for Next.js apps. Covers ca | `agentic-awesome-skills` |
| [nextjs-turbopack](./nextjs-turbopack) | Next.js 16+ and Turbopack — incremental bundling, FS caching, dev speed, and when to use Turbopack vs webpack. | `everything-claude-code-main` |
| [nika](./nika) | Runs repeatable AI work as checked, budgeted workflow files. | `agentic-awesome-skills` |
| [node-inspect-debugger](./node-inspect-debugger) | Debug Node.js via --inspect + Chrome DevTools Protocol CLI. | `hermes-agent` |
| [not-human-search-mcp](./not-human-search-mcp) | Search AI-ready websites, inspect indexed site details, verify MCP endpoints, and discover tools and APIs usin | `agentic-awesome-skills` |
| [notion](./notion) | Notion API + ntn CLI: pages, databases, markdown, Workers. | `hermes-agent` |
| [notion-automation](./notion-automation) | Automate Notion tasks via Rube MCP (Composio): pages, databases, blocks, comments, users. Always search tools  | `agentic-awesome-skills` |
| [notion-automation-antigravity-awesome-skills-main](./notion-automation-antigravity-awesome-skills-main) | Automate Notion tasks via Rube MCP (Composio): pages, databases, blocks, comments, users. Always search tools  | `antigravity-awesome-skills-main` |
| [novel-art](./novel-art) | \| | `shuohao-skills` |
| [novel-characters](./novel-characters) | \| | `shuohao-skills` |
| [novel-outline](./novel-outline) | \| | `shuohao-skills` |
| [novel-script](./novel-script) | \| | `shuohao-skills` |
| [novel-storyboard](./novel-storyboard) | \| | `shuohao-skills` |
| [nutrient-document-processing](./nutrient-document-processing) | Process, convert, OCR, extract, redact, sign, and fill documents using the Nutrient DWS API. Works with PDFs,  | `everything-claude-code-main` |
| [nuxt4-patterns](./nuxt4-patterns) | Nuxt 4 app patterns for hydration safety, performance, route rules, lazy loading, and SSR-safe data fetching w | `everything-claude-code-main` |
| [nvidia-skills-catalog-maintenance](./nvidia-skills-catalog-maintenance) | Maintain the NVIDIA skills catalog repository: components.d | `AREX-Skill` |
| [observability-and-instrumentation](./observability-and-instrumentation) | Instruments code so production behavior is visible and diagnosable. Use when adding logging, metrics, tracing, | `agentic-awesome-skills` |
| [obsidian](./obsidian) | Read, search, create, and edit notes in the Obsidian vault. | `hermes-agent` |
| [obsidian-cli](./obsidian-cli) | Interact with Obsidian vaults using the Obsidian CLI to read, create, search, and manage notes, tasks, propert | `obsidian-mind` |
| [octopus-architecture](./octopus-architecture) | Review system architecture, simplify boundaries, or compare interface designs from repository evidence | `claude-octopus` |
| [octopus-quick](./octopus-quick) | Quick execution for ad-hoc tasks without full workflow overhead — use for small, self-contained requests | `claude-octopus` |
| [octopus-research](./octopus-research) | Thorough research across multiple sources — use for complex topics needing broad synthesis | `claude-octopus` |
| [octopus-ui-ux-design](./octopus-ui-ux-design) | Design UI/UX systems with style guides, palettes, typography, and component specs for new interfaces | `claude-octopus` |
| [omero-integration](./omero-integration) | Securely inspect and automate microscopy data workflows against OMERO.server with omero-py, BlitzGateway, OMER | `scientific-agent-skills` |
| [omh-deep-research](./omh-deep-research) | parallel web research; subagents→synthesis→cite-verify | `witt3rd__oh-my-hermes` |
| [omh-ralph-driver](./omh-ralph-driver) | Drive an omh-ralph run: dispatch, evidence, commit hygiene. | `witt3rd__oh-my-hermes` |
| [omh-ralph-task](./omh-ralph-task) | Execute one omh-ralph task: file-scope, commit, report. | `witt3rd__oh-my-hermes` |
| [omh-ralplan](./omh-ralplan) | Planner+Architect+Critic→consensus impl plan (≤3 rounds) | `witt3rd__oh-my-hermes` |
| [omh-ralplan-driver](./omh-ralplan-driver) | Drive omh-ralplan: context package, rounds, distillation. | `witt3rd__oh-my-hermes` |
| [omh-triage](./omh-triage) | Multi-role consensus triage of an issue backlog. | `witt3rd__oh-my-hermes` |
| [omh-triage-driver](./omh-triage-driver) | Drive omh-triage: when to invoke, how to run rounds. | `witt3rd__oh-my-hermes` |
| [omp-delegate](./omp-delegate) | Delegate coding tasks to Oh My Pi (`omp`) only when the user explicitly | `agentic-awesome-skills` |
| [onboard](./onboard) | Generates a contextual onboarding document for a new contributor or agent joining the project. Summarizes proj | `Claude-Code-Game-Studios-main` |
| [onboard-Continuous-Claude-v3-main](./onboard-Continuous-Claude-v3-main) | Analyze brownfield codebase and create initial continuity ledger | `Continuous-Claude-v3-main` |
| [onboard-repo](./onboard-repo) | Index an unfamiliar codebase into the knowledge graph, then produce a first orientation map -- entry points, m | `n24q02m__crg` |
| [one-drive-automation](./one-drive-automation) | Automate OneDrive file management, search, uploads, downloads, sharing, permissions, and folder operations via | `agentic-awesome-skills` |
| [one-drive-automation-antigravity-awesome-skills-main](./one-drive-automation-antigravity-awesome-skills-main) | Automate OneDrive file management, search, uploads, downloads, sharing, permissions, and folder operations via | `antigravity-awesome-skills-main` |
| [onekgpd](./onekgpd) | > | `scientific-agent-skills` |
| [ontology-term-resolution](./ontology-term-resolution) | Resolve free-text scientific labels to ontology term IDs and validate existing CURIEs against the EBI Ontology | `scientific-agent-skills` |
| [ontoly-software-graph](./ontoly-software-graph) | Use Ontoly's deterministic Software Graph, MCP server, and agent skills for architecture review, request traci | `agentic-awesome-skills` |
| [opc-architecture](./opc-architecture) | OPC Architecture Understanding | `Continuous-Claude-v3-main` |
| [open-dynamic-workflows](./open-dynamic-workflows) | Plan, orchestrate, and adversarially verify parallel AI coding agents with a dynamic multi-agent workflow engi | `agentic-awesome-skills` |
| [open-notebook](./open-notebook) | Self-hosted, open-source alternative to Google NotebookLM for AI-powered research and document analysis. Use w | `scientific-agent-skills` |
| [open-source](./open-source) | > | `browser-use` |
| [open-wearables](./open-wearables) | Operating router for the Open Wearables FastAPI backend, wearable | `AREX-Skill` |
| [openai-docs-skill](./openai-docs-skill) | Query the OpenAI developer documentation via the OpenAI Docs MCP server using CLI (curl/jq). Use whenever a ta | `codex-skills-main` |
| [openapi-spec-generator](./openapi-spec-generator) | Generate complete, production-ready OpenAPI 3.x and Swagger 2.0 specifications from natural language descripti | `agentic-awesome-skills` |
| [openclaw](./openclaw) | Multi-agent swarm coordination via the ClawTeam CLI. Use when the user wants to create agent teams, spawn mult | `ClawTeam-OpenClaw` |
| [openclaw-persona-forge](./openclaw-persona-forge) | \|- | `everything-claude-code-main` |
| [openclaw-pr-maintainer](./openclaw-pr-maintainer) | Use immediately for any pasted OpenClaw GitHub issue or PR URL/number, and for OpenClaw issue/PR orchestration | `openclaw` |
| [openclaw-refactor-docs](./openclaw-refactor-docs) | Refactor an existing OpenClaw docs page with source-audited preservation, restructuring, and verification. | `openclaw` |
| [openclaw-repair-sweep](./openclaw-repair-sweep) | Orchestrate worker fleets over OpenClaw issues and PRs: prove root causes, prefer clean refactors over quick p | `openclaw` |
| [opencode](./opencode) | Delegate coding to OpenCode CLI (features, PR review). | `hermes-agent` |
| [opencode-delegate](./opencode-delegate) | Delegate coding tasks to the OpenCode CLI only when the user explicitly | `agentic-awesome-skills` |
| [opencode-hermes-agent-main](./opencode-hermes-agent-main) | Delegate coding tasks to OpenCode CLI agent for feature implementation, refactoring, PR review, and long-runni | `hermes-agent-main` |
| [openma](./openma) | > | `openma-ai__open-managed-agents` |
| [openmaic](./openmaic) | OpenMAIC assistant for setting up, generating, and extending OpenMAIC. Use when the user wants to use OpenMAIC | `OpenMAIC` |
| [openpiv](./openpiv) | Particle Image Velocimetry (PIV) analysis with OpenPIV. Use when extracting velocity fields from PIV image pai | `scientific-agent-skills` |
| [opensource-pipeline](./opensource-pipeline) | Open-source pipeline: fork, sanitize, and package private projects for safe public release. Chains 3 agents (f | `everything-claude-code-main` |
| [openstoryline-use](./openstoryline-use) | Use this skill when OpenStoryline is already installed and the user wants to start the local MCP/Web services, | `FireRed-OpenStoryline` |
| [opentrons-integration](./opentrons-integration) | Author, review, migrate, simulate, and troubleshoot official Opentrons Python Protocol API v2 protocols for Fl | `scientific-agent-skills` |
| [optimization-suggest](./optimization-suggest) | > | `Claude-Code-Agent-Monitor` |
| [optimize](./optimize) | USE ONLY WHEN A HUMAN EXPLICITLY INVOKES IT — never auto-select or run this proactively: not after writing cod | `daintree` |
| [optimize-for-gpu](./optimize-for-gpu) | GPU-accelerates scientific Python on NVIDIA hardware and verifies that the result is correct and faster. Use f | `scientific-agent-skills` |
| [orca-replay](./orca-replay) | Answers questions about a past agent run from its recording rather than from memory, and replays or forks that | `agentic-awesome-skills` |
| [orchestrate](./orchestrate) | Coordinate focused subagents on substantial work, keep their ownership non-overlapping, and integrate verified | `agentic-awesome-skills` |
| [orchestrate-batch-refactor](./orchestrate-batch-refactor) | Plan and execute large refactors with dependency-aware work packets and parallel analysis. | `agentic-awesome-skills` |
| [orchestrate-batch-refactor-antigravity-awesome-skills-main](./orchestrate-batch-refactor-antigravity-awesome-skills-main) | Plan and execute large refactors with dependency-aware work packets and parallel analysis. | `antigravity-awesome-skills-main` |
| [orchestration](./orchestration) | >- | `orca` |
| [outlook-automation](./outlook-automation) | Automate Outlook tasks via Rube MCP (Composio): emails, calendar, contacts, folders, attachments. Always searc | `agentic-awesome-skills` |
| [outlook-automation-antigravity-awesome-skills-main](./outlook-automation-antigravity-awesome-skills-main) | Automate Outlook tasks via Rube MCP (Composio): emails, calendar, contacts, folders, attachments. Always searc | `antigravity-awesome-skills-main` |
| [outlook-calendar-automation](./outlook-calendar-automation) | Automate Outlook Calendar tasks via Rube MCP (Composio): create events, manage attendees, find meeting times,  | `agentic-awesome-skills` |
| [outlook-calendar-automation-antigravity-awesome-skills-main](./outlook-calendar-automation-antigravity-awesome-skills-main) | Automate Outlook Calendar tasks via Rube MCP (Composio): create events, manage attendees, find meeting times,  | `antigravity-awesome-skills-main` |
| [owl](./owl) | Guides the OWL multi-agent task-automation package, CAMEL | `AREX-Skill` |
| [ownmem-dashboard](./ownmem-dashboard) | Open OwnMem Console, the local dashboard for this repository's memory. Use when the user asks to open the dash | `ownmem` |
| [ownmem-init](./ownmem-init) | Install or update OwnMem in the current repository. Use when the user asks to set up OwnMem, add local project | `ownmem` |
| [p5js](./p5js) | p5.js sketches: gen art, shaders, interactive, 3D. | `hermes-agent` |
| [pacsomatic](./pacsomatic) | Operator toolkit for nf-core/pacsomatic matched tumor-normal workflows from BAM inputs. Use this skill when th | `scientific-agent-skills` |
| [pagerduty-automation](./pagerduty-automation) | Automate PagerDuty tasks via Rube MCP (Composio): manage incidents, services, schedules, escalation policies,  | `agentic-awesome-skills` |
| [pagerduty-automation-antigravity-awesome-skills-main](./pagerduty-automation-antigravity-awesome-skills-main) | Automate PagerDuty tasks via Rube MCP (Composio): manage incidents, services, schedules, escalation policies,  | `antigravity-awesome-skills-main` |
| [paper-lookup](./paper-lookup) | Search 11 academic literature APIs for papers, preprints, citations, and open-access full text, and return res | `scientific-agent-skills` |
| [paperclip](./paperclip) | Search and read full-text biomedical papers, FDA/PMDA/EMA regulatory documents, clinical trial registries, and | `scientific-agent-skills` |
| [papers-skill](./papers-skill) | Skill for academic research workflows: search Semantic Scholar (200M+ papers), inspect citations, download arX | `agentic-awesome-skills` |
| [paperzilla](./paperzilla) | Chat with your agent about projects, recommendations, and canonical papers in Paperzilla. Use when users ask f | `scientific-agent-skills` |
| [parallel](./parallel) | > | `codex-skills-main` |
| [parallel-agents](./parallel-agents) | Parallel Agent Orchestration | `Continuous-Claude-v3-main` |
| [parallel-feature-development](./parallel-feature-development) | Coordinate parallel feature development with file ownership strategies, conflict avoidance rules, and integrat | `agents-main` |
| [parallel-task](./parallel-task) | > | `codex-skills-main` |
| [parallel-task-spark](./parallel-task-spark) | > | `codex-skills-main` |
| [parallel-web](./parallel-web) | Use Parallel CLI for web search, URL extraction, deep research, structured data enrichment, entity discovery,  | `scientific-agent-skills` |
| [parallel-worktrees](./parallel-worktrees) | Create and manage git worktrees for parallel coding sessions with zero dead time. Use when blocked on tests, b | `pro-workflow` |
| [paseo-plugin](./paseo-plugin) | Build and manage trusted local Paseo plugins. Use when the user asks to create, edit, install, reload, enable, | `getpaseo__paseo` |
| [patch-notes](./patch-notes) | Generate player-facing patch notes from git history, sprint data, and internal changelogs. Translates develope | `Claude-Code-Game-Studios-main` |
| [pathml](./pathml) | Use PathML for local, research-only computational pathology workflows: load and tile slides, build preprocessi | `scientific-agent-skills` |
| [pathogen-variant-surveillance](./pathogen-variant-surveillance) | Query live pathogen genomic surveillance data through the GenSpectrum LAPIS API to find which viral lineages a | `scientific-agent-skills` |
| [pathway-enrichment](./pathway-enrichment) | Run pathway and gene-set enrichment analysis on gene lists or ranked gene data, then interpret the results. Us | `scientific-agent-skills` |
| [pattern-detect](./pattern-detect) | > | `Claude-Code-Agent-Monitor` |
| [pdf](./pdf) | PDF files: create, read, merge, fill, OCR, edit text. | `hermes-agent` |
| [pdf-scientific-agent-skills](./pdf-scientific-agent-skills) | Use this skill whenever the user wants to do anything with PDF files. This includes reading or extracting text | `scientific-agent-skills` |
| [pdf-shareAI-lab__learn-claude-code](./pdf-shareAI-lab__learn-claude-code) | Process PDF files - extract text, create PDFs, merge documents. Use when user asks to read PDF, create PDF, or | `shareAI-lab__learn-claude-code` |
| [peer-review](./peer-review) | Prepare evidence-bounded, constructive peer-review drafts and structured manuscript assessments. Use for autho | `scientific-agent-skills` |
| [penguin-harness-dev](./penguin-harness-dev) | Use when developing PenguinHarness itself — changing packages/{core,server,web,cli,desktop,landing,docs,skills | `penguin-harness` |
| [penguin-sdk](./penguin-sdk) | Use whenever the user wants to build an agent application — their own program with an embedded agent, such as  | `penguin-harness` |
| [pennylane](./pennylane) | Hardware-agnostic quantum ML framework with automatic differentiation. Use when training quantum circuits via  | `scientific-agent-skills` |
| [people-data](./people-data) | Research LinkedIn professional profiles and public business-contact data, including email/phone lookup, people | `agentic-awesome-skills` |
| [perf-profile](./perf-profile) | Structured performance profiling workflow. Identifies bottlenecks, measures against budgets, and generates opt | `Claude-Code-Game-Studios-main` |
| [performance-optimization](./performance-optimization) | Optimizes application performance. Use when performance requirements exist, when you suspect performance regre | `agentic-awesome-skills` |
| [performance-testing-review-multi-agent-review](./performance-testing-review-multi-agent-review) | Use when working with performance testing review multi agent review | `agentic-awesome-skills` |
| [performance-testing-review-multi-agent-review-antigravity-awesome-skills-main](./performance-testing-review-multi-agent-review-antigravity-awesome-skills-main) | Use when working with performance testing review multi agent review | `antigravity-awesome-skills-main` |
| [perl-patterns](./perl-patterns) | Modern Perl 5.36+ idioms, best practices, and conventions for building robust, maintainable Perl applications. | `everything-claude-code-main` |
| [perl-testing](./perl-testing) | Perl testing patterns using Test2::V0, Test::More, prove runner, mocking, coverage with Devel::Cover, and TDD  | `everything-claude-code-main` |
| [permission-manager](./permission-manager) | Manage opencode permissions: review always-allow lists, suggest safe read-only commands, configure permission  | `agentic-awesome-skills` |
| [pettingzoo](./pettingzoo) | Use, select, author, validate, wrap, and integrate PettingZoo | `AREX-Skill` |
| [pi-agent](./pi-agent) | Build with and use Pi, the minimal terminal coding harness. Use for installing Pi, configuring providers/model | `scientific-agent-skills` |
| [pi-delegate](./pi-delegate) | Delegate coding tasks to the Pi coding agent CLI (`pi`) only when the | `agentic-awesome-skills` |
| [pi-interactive-shell](./pi-interactive-shell) | Cheat sheet + workflow for launching interactive coding-agent CLIs (Claude Code, Gemini CLI, Codex CLI, Cursor | `nicobailon__pi-interactive-shell` |
| [pilotdeck-skills-migration](./pilotdeck-skills-migration) | >- | `OpenBMB__PilotDeck` |
| [pipedrive-automation](./pipedrive-automation) | Automate Pipedrive CRM operations including deals, contacts, organizations, activities, notes, and pipeline ma | `agentic-awesome-skills` |
| [pipedrive-automation-antigravity-awesome-skills-main](./pipedrive-automation-antigravity-awesome-skills-main) | Automate Pipedrive CRM operations including deals, contacts, organizations, activities, notes, and pipeline ma | `antigravity-awesome-skills-main` |
| [pixel-face](./pixel-face) | Create or improve a 32×32 pixel-art persona face for the Frontier Faces demo (examples/04-frontier-faces/perso | `Cotal-AI__Cotal` |
| [pixijs-application](./pixijs-application) | Use this skill when creating and configuring a PixiJS v8 Application. Covers new Application() + async app.ini | `cohub` |
| [pkpd-modeling](./pkpd-modeling) | Pharmacokinetic and pharmacodynamic modelling and simulation - non-compartmental analysis, compartmental and p | `scientific-agent-skills` |
| [plan](./plan) | Plan mode for Hermes — inspect context, write a markdown plan into the active workspace's `.hermes/plans/` dir | `hermes-agent-main` |
| [planner-orchestration](./planner-orchestration) | Enforce Kandev's single-session, user-controlled model workflow for feature, fix, debug, review, verification, | `kandev` |
| [planning-and-task-breakdown](./planning-and-task-breakdown) | Breaks work into ordered tasks. Use when you have a spec or clear requirements and need to break work into imp | `agentic-awesome-skills` |
| [playtest-report](./playtest-report) | Generates a structured playtest report template or analyzes existing playtest notes into a structured format.  | `Claude-Code-Game-Studios-main` |
| [playwright-skill](./playwright-skill) | IMPORTANT - Path Resolution: This skill can be installed in different locations (plugin system, manual install | `agentic-awesome-skills` |
| [playwright-skill-antigravity-awesome-skills-main](./playwright-skill-antigravity-awesome-skills-main) | IMPORTANT - Path Resolution: This skill can be installed in different locations (plugin system, manual install | `antigravity-awesome-skills-main` |
| [plotly](./plotly) | Interactive visualization library. Use when you need hover info, zoom, pan, or web-embeddable charts. Best for | `agentic-awesome-skills` |
| [plotly-antigravity-awesome-skills-main](./plotly-antigravity-awesome-skills-main) | Interactive visualization library. Use when you need hover info, zoom, pan, or web-embeddable charts. Best for | `antigravity-awesome-skills-main` |
| [plugin-settings](./plugin-settings) | This skill should be used when the user asks about "plugin settings", "store plugin configuration", "user-conf | `claude-plugins-official-main` |
| [plugin-settings-yoink-main](./plugin-settings-yoink-main) | This skill should be used when the user asks about "plugin settings", "store plugin configuration", "user-conf | `yoink-main` |
| [plugin-structure](./plugin-structure) | This skill should be used when the user asks to "create a plugin", "scaffold a plugin", "understand plugin str | `claude-plugins-official-main` |
| [plugin-structure-yoink-main](./plugin-structure-yoink-main) | This skill should be used when the user asks to "create a plugin", "scaffold a plugin", "understand plugin str | `yoink-main` |
| [pluginstaller](./pluginstaller) | Add a Codex plugin from a GitHub repo to a repo or personal marketplace using the current Codex plugin layout  | `codex-skills-main` |
| [polars](./polars) | Fast in-memory DataFrame library for datasets that fit in RAM. Use when pandas is too slow but data still fits | `agentic-awesome-skills` |
| [polars-antigravity-awesome-skills-main](./polars-antigravity-awesome-skills-main) | Fast in-memory DataFrame library for datasets that fit in RAM. Use when pandas is too slow but data still fits | `antigravity-awesome-skills-main` |
| [polars-bio](./polars-bio) | High-performance genomic interval operations and bioinformatics file I/O on Polars DataFrames. Overlap, neares | `scientific-agent-skills` |
| [polars-scientific-agent-skills](./polars-scientific-agent-skills) | High-performance DataFrame library for Python ETL, analytics, and pandas migration. Use for expression-based d | `scientific-agent-skills` |
| [popular-web-designs](./popular-web-designs) | 54 real design systems (Stripe, Linear, Vercel) as HTML/CSS. | `hermes-agent` |
| [postgres-patterns](./postgres-patterns) | PostgreSQL database patterns for query optimization, schema design, indexing, and security. Based on Supabase  | `everything-claude-code-main` |
| [postgresql-cli](./postgresql-cli) | PostgreSQL interactive terminal (psql) reference and usage guide. | `agentic-awesome-skills` |
| [posthog-automation](./posthog-automation) | Automate PostHog tasks via Rube MCP (Composio): events, feature flags, projects, user profiles, annotations. A | `agentic-awesome-skills` |
| [posthog-automation-antigravity-awesome-skills-main](./posthog-automation-antigravity-awesome-skills-main) | Automate PostHog tasks via Rube MCP (Composio): events, feature flags, projects, user profiles, annotations. A | `antigravity-awesome-skills-main` |
| [postman-collection-generator](./postman-collection-generator) | Generate complete, import-ready Postman Collection v2.1 JSON files from natural language API descriptions or c | `agentic-awesome-skills` |
| [postman-newman-automation](./postman-newman-automation) | Generate Newman CLI commands, configuration files, Jenkins pipeline scripts, and shell automation for running  | `agentic-awesome-skills` |
| [postman-openapi-converter](./postman-openapi-converter) | Convert OpenAPI 3.x or Swagger 2.0 specs (YAML or JSON) into complete, import-ready Postman Collection v2.1 JS | `agentic-awesome-skills` |
| [postmark-automation](./postmark-automation) | Automate Postmark email delivery tasks via Rube MCP (Composio): send templated emails, manage templates, monit | `agentic-awesome-skills` |
| [postmark-automation-antigravity-awesome-skills-main](./postmark-automation-antigravity-awesome-skills-main) | Automate Postmark email delivery tasks via Rube MCP (Composio): send templated emails, manage templates, monit | `antigravity-awesome-skills-main` |
| [potpie](./potpie) | Operate Potpie's CLI, daemon, context graph, source bindings, auth | `AREX-Skill` |
| [powerpoint](./powerpoint) | Create, read, edit .pptx decks with python-pptx. | `hermes-agent` |
| [ppt-agent-skills](./ppt-agent-skills) | 专业 PPT 演示文稿全流程 AI 生成助手。模拟顶级 PPT 设计公司的完整工作流（需求调研到资料搜集到大纲策划到策划稿到设计稿），输出高质量 HTML 格式演示文稿。当用户提到制作 PPT、做演示文稿、做 slide | `ppt-agent-skills` |
| [pptx](./pptx) | Use this skill any time a .pptx or .potx file is involved in any way — as input, output, or both. This include | `scientific-agent-skills` |
| [pptx-posters](./pptx-posters) | Create and audit editable scientific posters in macro-free PowerPoint (.pptx) from author-approved local conte | `scientific-agent-skills` |
| [pr-fixup](./pr-fixup) | Wait for CI and automated reviews on a PR, fix valid failures and comments, disposition every review thread, v | `kandev` |
| [pr-improver](./pr-improver) | Runs an autonomous review-and-fix improvement loop over the current branch's changes until a PR review comes b | `trailofbits__skills` |
| [pr-watch](./pr-watch) | >- | `Citadel` |
| [pre-release-review](./pre-release-review) | Run a read-only pre-release review for deploy readiness, migrations, config, secrets, rollout order, rollback  | `agentic-awesome-skills` |
| [presentation-structure](./presentation-structure) | Knowledge about the presentation slide format, weight system, navigation, and section structure | `claude-code-best-practice` |
| [presentation-structure-claude-code-best-practice-main](./presentation-structure-claude-code-best-practice-main) | Knowledge about the presentation slide format, weight system, navigation, and section structure | `claude-code-best-practice-main` |
| [presentation-styling](./presentation-styling) | Knowledge about CSS classes, component patterns, and syntax highlighting in the presentation | `claude-code-best-practice` |
| [presentation-styling-claude-code-best-practice-main](./presentation-styling-claude-code-best-practice-main) | Knowledge about CSS classes, component patterns, and syntax highlighting in the presentation | `claude-code-best-practice-main` |
| [printing-press-amend](./printing-press-amend) | > | `mvanhorn__cli-printing-press` |
| [prism](./prism) | > | `irfndi__prism-liquidity-agent` |
| [prism-hermes](./prism-hermes) | > | `irfndi__prism-liquidity-agent` |
| [prism-maintain](./prism-maintain) | > | `irfndi__prism-liquidity-agent` |
| [prism-openclaw](./prism-openclaw) | > | `irfndi__prism-liquidity-agent` |
| [pro-workflow](./pro-workflow) | Complete AI coding workflow system. Orchestration patterns, 18 hook events, 8 agents, cross-agent support, ref | `pro-workflow` |
| [product-capability](./product-capability) | Translate PRD intent, roadmap asks, or product discussions into an implementation-ready capability plan that e | `everything-claude-code-main` |
| [product-design](./product-design) | Design de produto nivel Apple — sistemas visuais, UX flows, acessibilidade, linguagem visual proprietaria, des | `agentic-awesome-skills` |
| [product-design-antigravity-awesome-skills-main](./product-design-antigravity-awesome-skills-main) | Design de produto nivel Apple — sistemas visuais, UX flows, acessibilidade, linguagem visual proprietaria, des | `antigravity-awesome-skills-main` |
| [product-inventor](./product-inventor) | Product Inventor e Design Alchemist de nivel maximo — combina Product Thinking, Design Systems, UI Engineering | `agentic-awesome-skills` |
| [product-inventor-antigravity-awesome-skills-main](./product-inventor-antigravity-awesome-skills-main) | Product Inventor e Design Alchemist de nivel maximo — combina Product Thinking, Design Systems, UI Engineering | `antigravity-awesome-skills-main` |
| [product-lens](./product-lens) | Use this skill to validate the "why" before building, run product diagnostics, and pressure-test product direc | `everything-claude-code-main` |
| [product-marketing](./product-marketing) | > | `marketing-skills` |
| [product-price-monitor](./product-price-monitor) | Watch product, flight, or listing prices; alert on target. | `hermes-agent` |
| [production-scheduling](./production-scheduling) | Codified expertise for production scheduling, job sequencing, line balancing, changeover optimisation, and bot | `agentic-awesome-skills` |
| [production-scheduling-antigravity-awesome-skills-main](./production-scheduling-antigravity-awesome-skills-main) | Codified expertise for production scheduling, job sequencing, line balancing, changeover optimisation, and bot | `antigravity-awesome-skills-main` |
| [production-scheduling-ECC](./production-scheduling-ECC) | > | `ECC` |
| [production-scheduling-everything-claude-code-main](./production-scheduling-everything-claude-code-main) | > | `everything-claude-code-main` |
| [productivity-score](./productivity-score) | > | `Claude-Code-Agent-Monitor` |
| [project-development](./project-development) | This skill should be used when the user asks to "start an LLM project", "design batch pipeline", "evaluate tas | `Agent-Skills-for-Context-Engineering-main` |
| [project-development-agentic-awesome-skills](./project-development-agentic-awesome-skills) | This skill covers the principles for identifying tasks suited to LLM processing, designing effective project a | `agentic-awesome-skills` |
| [project-development-antigravity-awesome-skills-main](./project-development-antigravity-awesome-skills-main) | This skill covers the principles for identifying tasks suited to LLM processing, designing effective project a | `antigravity-awesome-skills-main` |
| [project-flow-ops](./project-flow-ops) | Operate execution flow across GitHub and Linear by triaging issues and pull requests, linking active work, and | `everything-claude-code-main` |
| [project-stage-detect](./project-stage-detect) | Automatically analyze project state, detect stage, identify gaps, and recommend next steps based on existing a | `Claude-Code-Game-Studios-main` |
| [project-state-governor](./project-state-governor) | Govern evidence-backed canonical project state across sessions, branches, reviews, and research cycles without | `agentic-awesome-skills` |
| [prompt](./prompt) | Use when the user asks to create, refine, or optimize a prompt ('프롬프트 만들어/생성해/다듬어') — AI 프롬프트 생성기. 사용자의 아이디어를  | `treylom__prompt-engineering-skills` |
| [prompt-architect](./prompt-architect) | Designs clear, testable prompts for agent workflows. Use when creating new prompts, refining weak prompts, or  | `claude-code-prompts-master` |
| [prompt-optimizer](./prompt-optimizer) | >- | `ECC` |
| [prompt-optimizer-everything-claude-code-main](./prompt-optimizer-everything-claude-code-main) | >- | `everything-claude-code-main` |
| [propagate-design-change](./propagate-design-change) | When a GDD is revised, scans all ADRs and the traceability index to identify which architectural decisions are | `Claude-Code-Game-Studios-main` |
| [protocolsio-integration](./protocolsio-integration) | Read, validate, and safely export protocols.io data with current official REST/MCP contracts, or create non-ex | `scientific-agent-skills` |
| [prototype](./prototype) | Build a throwaway prototype to flesh out a design — a runnable terminal app for state/business-logic questions | `agentic-awesome-skills` |
| [prototype-Claude-Code-Game-Studios-main](./prototype-Claude-Code-Game-Studios-main) | Rapid prototyping workflow. Skips normal standards to quickly validate a game concept or mechanic. Produces th | `Claude-Code-Game-Studios-main` |
| [provider-health](./provider-health) | Starter: one-screen provider health summary — availability, auth method, version drift, and cost posture for e | `claude-octopus` |
| [provider-research](./provider-research) | Research every provider behind Pipecat's services for new models and API affordances, writing per-service repo | `pipecat` |
| [provider-retry](./provider-retry) | Diagnose automatic retries of BB turns after provider subscription-window limits. | `bb` |
| [pstack](./pstack) | Rigorous engineering mode for nontrivial work in this repo — a set of named principles plus the leaf skills th | `rove` |
| [pubmed-database](./pubmed-database) | Direct REST API access to PubMed. Advanced Boolean/MeSH queries, E-utilities API, batch processing, citation m | `agentic-awesome-skills` |
| [pubmed-database-antigravity-awesome-skills-main](./pubmed-database-antigravity-awesome-skills-main) | Direct REST API access to PubMed. Advanced Boolean/MeSH queries, E-utilities API, batch processing, citation m | `antigravity-awesome-skills-main` |
| [pufferlib](./pufferlib) | Version-aware guidance for PufferLib reinforcement-learning environments, vectorization, policies, PuffeRL tra | `scientific-agent-skills` |
| [puppeteer-skill](./puppeteer-skill) | Generates Puppeteer scripts for browser automation, scraping, and PDF generation. Triggers on: "Puppeteer", "h | `agentic-awesome-skills` |
| [push-skill-to-github](./push-skill-to-github) | Commit and push skill changes to the configured skills repository after review and validation. | `agentic-awesome-skills` |
| [push-to-forked-pr](./push-to-forked-pr) | Push the current working tree directly to a GitHub PR whose head lives on a **fork**, without creating a new b | `Claude-Code-Agent-Monitor` |
| [pydeseq2](./pydeseq2) | Differential gene expression analysis for bulk RNA-seq with PyDESeq2, including formulaic designs, Wald tests, | `scientific-agent-skills` |
| [pyhealth](./pyhealth) | Build clinical/healthcare deep-learning pipelines with PyHealth — loading EHR/signal/imaging datasets (MIMIC-I | `scientific-agent-skills` |
| [pylabrobot](./pylabrobot) | Develop and review PyLabRobot lab-automation resources, liquid-handling plans, offline simulations, and suppor | `scientific-agent-skills` |
| [pymatgen](./pymatgen) | Analyze, validate, convert, and transform materials structures and computed materials data with current pymatg | `scientific-agent-skills` |
| [pymc](./pymc) | Bayesian modeling with PyMC. Build hierarchical models, MCMC (NUTS), variational inference, LOO/WAIC compariso | `scientific-agent-skills` |
| [pymoo](./pymoo) | Multi-objective optimization framework. NSGA-II, NSGA-III, MOEA/D, Pareto fronts, constraint handling, benchma | `scientific-agent-skills` |
| [pyod](./pyod) | Operate PyOD anomaly-detection workflows across classic detectors, | `AREX-Skill` |
| [pyopenms](./pyopenms) | Complete mass spectrometry analysis platform. Use for proteomics and metabolomics workflows—feature detection, | `scientific-agent-skills` |
| [pysam](./pysam) | Python/HTSlib workflows for genomic files. Use when reading, querying, filtering, or writing SAM/BAM/CRAM, VCF | `scientific-agent-skills` |
| [pyserini](./pyserini) | Use Pyserini for reproducible information retrieval: | `AREX-Skill` |
| [pytdc](./pytdc) | Use Therapeutics Data Commons through the PyTDC Python package for registry discovery, approved dataset access | `scientific-agent-skills` |
| [pytest-skill](./pytest-skill) | Generates production-grade pytest tests in Python with fixtures, parametrize, markers, mocking, and conftest p | `agentic-awesome-skills` |
| [python-debugpy](./python-debugpy) | Debug Python: pdb REPL + debugpy remote (DAP). | `hermes-agent` |
| [python-patterns](./python-patterns) | Pythonic idioms, PEP 8 standards, type hints, and best practices for building robust, efficient, and maintaina | `everything-claude-code-main` |
| [python-testing](./python-testing) | Python testing strategies using pytest, TDD methodology, fixtures, mocking, parametrization, and coverage requ | `everything-claude-code-main` |
| [pytorch-lightning](./pytorch-lightning) | Deep learning framework (PyTorch Lightning / lightning package). Organize PyTorch code into LightningModules,  | `scientific-agent-skills` |
| [pytorch-patterns](./pytorch-patterns) | PyTorch deep learning patterns and best practices for building robust, efficient, and reproducible training pi | `everything-claude-code-main` |
| [pyzotero](./pyzotero) | Interact with Zotero reference management libraries using the pyzotero Python client. Retrieve, create, update | `scientific-agent-skills` |
| [qa-plan](./qa-plan) | Generate a QA test plan for a sprint or feature. Reads GDDs and story files, classifies stories by test type ( | `Claude-Code-Game-Studios-main` |
| [qiskit](./qiskit) | Qiskit is the world's most popular open-source quantum computing framework with 13M+ downloads. Build quantum  | `agentic-awesome-skills` |
| [qiskit-antigravity-awesome-skills-main](./qiskit-antigravity-awesome-skills-main) | Qiskit is the world's most popular open-source quantum computing framework with 13M+ downloads. Build quantum  | `antigravity-awesome-skills-main` |
| [qiskit-scientific-agent-skills](./qiskit-scientific-agent-skills) | Build, simulate, transpile, and execute quantum circuits with Qiskit and IBM Quantum Runtime. Use for Qiskit 2 | `scientific-agent-skills` |
| [qoder-delegate](./qoder-delegate) | Delegate coding tasks to the Qoder CLI (`qodercli`) only when the user | `agentic-awesome-skills` |
| [quality-nonconformance](./quality-nonconformance) | Codified expertise for quality control, non-conformance investigation, root cause analysis, corrective action, | `agentic-awesome-skills` |
| [quality-nonconformance-antigravity-awesome-skills-main](./quality-nonconformance-antigravity-awesome-skills-main) | Codified expertise for quality control, non-conformance investigation, root cause analysis, corrective action, | `antigravity-awesome-skills-main` |
| [quality-nonconformance-ECC](./quality-nonconformance-ECC) | > | `ECC` |
| [quality-nonconformance-everything-claude-code-main](./quality-nonconformance-everything-claude-code-main) | > | `everything-claude-code-main` |
| [quantdb-fields](./quantdb-fields) | QuantDB 字段单位速查手册 — 全部数据集实测验证的单位、口径与陷阱（个股 volume=股/amount=万元、指数 volume=手、L2原始逐笔 l2_data/tick_data、technical % v | `QuantMind` |
| [quantdb-sdk](./quantdb-sdk) | QuantDB 数据 SDK — API Key 配置、数据集目录、字段查询。在 QuantBot / Claude Code 中查询 QuantDB 数据、配置 API Key、预览/同步数据集、远程查询 K 线/财务 | `QuantMind` |
| [quantmind-deploy](./quantmind-deploy) | QuantMind 部署运维完整指南。覆盖**部署前准备 → 一键/手动部署 → 部署后检查 → 问题排查 → 更新 → 云端训练**全流程。本技能针对 AI 编程助手编写，每步都给出可直接执行的命令与判断标准，避免"不 | `QuantMind` |
| [quantmind-operations](./quantmind-operations) | QuantMind 平台运营操作技能 — 覆盖模型训练、模型管理、后台数据更新、字段信息查询、RSS 新闻对接与分析。在 QuantBot / Claude Code 中处理模型训练、数据同步、新闻分析等任务时使用。触发 | `QuantMind` |
| [quick-design](./quick-design) | Lightweight design spec for small changes — tuning adjustments, minor mechanics, balance tweaks. Skips full GD | `Claude-Code-Game-Studios-main` |
| [quick-stats](./quick-stats) | > | `Claude-Code-Agent-Monitor` |
| [qutip](./qutip) | Simulate and audit closed and open quantum-system models with QuTiP 5, including deterministic, trajectory, st | `scientific-agent-skills` |
| [race](./race) | Race condition / TOCTOU playbook — limit overrun (one-time codes used twice, gift cards spent twice), single-p | `agent` |
| [ralphinho-rfc-pipeline](./ralphinho-rfc-pipeline) | RFC-driven multi-agent DAG execution pattern with quality gates, merge queues, and work unit orchestration. Us | `ECC` |
| [ralphinho-rfc-pipeline-everything-claude-code-main](./ralphinho-rfc-pipeline-everything-claude-code-main) | RFC-driven multi-agent DAG execution pattern with quality gates, merge queues, and work unit orchestration. | `everything-claude-code-main` |
| [rayden-use](./rayden-use) | Build and maintain Rayden UI components and screens in Figma via Figma MCP with full design token enforcement | `agentic-awesome-skills` |
| [rclone-cli](./rclone-cli) | Rclone command-line cloud storage manager reference and usage guide. Use this skill whenever the user mentions | `agentic-awesome-skills` |
| [rd-agent](./rd-agent) | Operate Microsoft's RD-Agent research-agent framework for | `AREX-Skill` |
| [rd-agent-factor-mining](./rd-agent-factor-mining) | RD-Agent A股因子挖掘端到端流水线：环境 preflight → 启动演化 → 轮询完成 → 批量回测评估 → IC/Sharpe 排序 → explain 解读 → export 入库 → Markdown 报 | `QuantMind` |
| [rdkit](./rdkit) | Cheminformatics toolkit for fine-grained molecular control. SMILES/SDF parsing, descriptors (MW, LogP, TPSA),  | `scientific-agent-skills` |
| [react-native-skills](./react-native-skills) | Use when working with react-native-skills tasks or workflows | `agentic-awesome-skills` |
| [react-next-best-practices](./react-next-best-practices) | React/Next.js 项目最佳实践：组件拆分、数据获取、性能、bundle、RSC、hydration 和路由。 | `OpenBMB__PilotDeck` |
| [react-performance](./react-performance) | React and Next.js performance optimization patterns adapted from Vercel Engineering's React Best Practices (ht | `ECC` |
| [reasoningbank-agentdb](./reasoningbank-agentdb) | Implement ReasoningBank adaptive learning with AgentDB's 150x faster vector database. Includes trajectory trac | `ruflo` |
| [reasoningbank-agentdb-ruflo](./reasoningbank-agentdb-ruflo) | Implement ReasoningBank adaptive learning with AgentDB's 150x faster vector database. Includes trajectory trac | `ruflo` |
| [reasoningbank-agentdb-ruflo-main](./reasoningbank-agentdb-ruflo-main) | Implement ReasoningBank adaptive learning with AgentDB's 150x faster vector database. Includes trajectory trac | `ruflo-main` |
| [reasoningbank-agentdb-ruflo-main](./reasoningbank-agentdb-ruflo-main) | Implement ReasoningBank adaptive learning with AgentDB's 150x faster vector database. Includes trajectory trac | `ruflo-main` |
| [reddit-automation](./reddit-automation) | Automate Reddit tasks via Rube MCP (Composio): search subreddits, create posts, manage comments, and browse to | `agentic-awesome-skills` |
| [reddit-automation-antigravity-awesome-skills-main](./reddit-automation-antigravity-awesome-skills-main) | Automate Reddit tasks via Rube MCP (Composio): search subreddits, create posts, manage comments, and browse to | `antigravity-awesome-skills-main` |
| [reddit-fetch](./reddit-fetch) | Fetch content from Reddit using Gemini CLI or curl JSON API fallback. Use when accessing Reddit URLs, research | `claude-code-tips-main` |
| [redis-cli](./redis-cli) | Redis command-line interface (redis-cli) reference and usage guide. Use this skill whenever the user mentions  | `agentic-awesome-skills` |
| [refresh-index](./refresh-index) | Sync every enabled repo in the curated skill index and open a confirmation-gated PR. Use when refreshing alrea | `luongnv89__asm` |
| [regex-vs-llm-structured-text](./regex-vs-llm-structured-text) | Decision framework for choosing between regex and LLM when parsing structured text — start with regex, add LLM | `everything-claude-code-main` |
| [regression-alert](./regression-alert) | > | `Claude-Code-Agent-Monitor` |
| [regression-suite](./regression-suite) | Map test coverage to GDD critical paths, identify fixed bugs without regression tests, flag coverage drift fro | `Claude-Code-Game-Studios-main` |
| [regression-watch](./regression-watch) | > | `Claude-Code-Agent-Monitor` |
| [relaydeck](./relaydeck) | > | `relaydeck` |
| [release-checklist](./release-checklist) | Generates a comprehensive pre-release validation checklist covering build verification, certification requirem | `Claude-Code-Game-Studios-main` |
| [release-guard](./release-guard) | Run release-readiness checks for this repository. Use when validating docs, scripts, verification coverage, an | `Claude-Code-Agent-Monitor` |
| [reliability-report](./reliability-report) | > | `Claude-Code-Agent-Monitor` |
| [relsa-severity-assessment](./relsa-severity-assessment) | Multivariate severity assessment and humane endpoint prediction for laboratory animal studies using the RELSA  | `scientific-agent-skills` |
| [remote-claude-code](./remote-claude-code) | Run Claude Code on a remote host over SSH — a persistent expect-driven login session, headless claude -p with  | `penguin-harness` |
| [remote-collection](./remote-collection) | > | `Claude-Code-Agent-Monitor` |
| [remote-gpu-trainer](./remote-gpu-trainer) | Deploy, monitor, and debug long GPU jobs on RENTED/remote instances (AutoDL, RunPod, vast.ai, Lambda, Slurm, K | `agentic-awesome-skills` |
| [remotion](./remotion) | Generate walkthrough videos from Stitch projects using Remotion with smooth transitions, zooming, and text ove | `agentic-awesome-skills` |
| [remotion-antigravity-awesome-skills-main](./remotion-antigravity-awesome-skills-main) | Generate walkthrough videos from Stitch projects using Remotion with smooth transitions, zooming, and text ove | `antigravity-awesome-skills-main` |
| [remotion-Better-Fullstack](./remotion-Better-Fullstack) | Generate walkthrough videos from Stitch projects using Remotion with smooth transitions, zooming, and text ove | `Better-Fullstack` |
| [remotion-video-creation](./remotion-video-creation) | Best practices for Remotion - Video creation in React. 29 domain-specific rules covering 3D, animations, audio | `everything-claude-code-main` |
| [render-automation](./render-automation) | Automate Render tasks via Rube MCP (Composio): services, deployments, projects. Always search tools first for  | `agentic-awesome-skills` |
| [render-automation-antigravity-awesome-skills-main](./render-automation-antigravity-awesome-skills-main) | Automate Render tasks via Rube MCP (Composio): services, deployments, projects. Always search tools first for  | `antigravity-awesome-skills-main` |
| [repo-onboarding](./repo-onboarding) | Understand this repository quickly before making changes. Use for architecture discovery, ownership mapping, c | `Claude-Code-Agent-Monitor` |
| [repo-scan](./repo-scan) | Cross-stack source code asset audit — classifies every file, detects embedded third-party libraries, and deliv | `everything-claude-code-main` |
| [repo-to-skill](./repo-to-skill) | Generate an agent skill from a GitHub open-source repository. Use when asked to create a skill from a repo URL | `repo-to-skill` |
| [repoprompt](./repoprompt) | Use RepoPrompt CLI for token-efficient codebase exploration | `Continuous-Claude-v3-main` |
| [requesting-code-review](./requesting-code-review) | Use when completing tasks, implementing major features, or before merging to verify work meets requirements | `agentic-awesome-skills` |
| [requesting-code-review-antigravity-awesome-skills-main](./requesting-code-review-antigravity-awesome-skills-main) | Use when completing tasks, implementing major features, or before merging to verify work meets requirements | `antigravity-awesome-skills-main` |
| [requesting-code-review-hermes-agent](./requesting-code-review-hermes-agent) | Pre-commit review: security scan, quality gates, auto-fix. | `hermes-agent` |
| [requesting-code-review-hermes-agent-main](./requesting-code-review-hermes-agent-main) | > | `hermes-agent-main` |
| [requesting-code-review-superpowers-main](./requesting-code-review-superpowers-main) | Use when completing tasks, implementing major features, or before merging to verify work meets requirements | `superpowers-main` |
| [research-lookup](./research-lookup) | Compile current scholarly evidence for a scientific manuscript or research brief. Use when the user explicitly | `scientific-agent-skills` |
| [research-ops](./research-ops) | Evidence-first current-state research workflow for ECC. Use when the user wants fresh facts, comparisons, enri | `everything-claude-code-main` |
| [research-paper-writing](./research-paper-writing) | End-to-end pipeline for writing ML/AI research papers — from experiment design through analysis, drafting, rev | `hermes-agent-main` |
| [retrospective](./retrospective) | Generates a sprint or milestone retrospective by analyzing completed work, velocity, blockers, and patterns. P | `Claude-Code-Game-Studios-main` |
| [returns-reverse-logistics](./returns-reverse-logistics) | Codified expertise for returns authorisation, receipt and inspection, disposition decisions, refund processing | `agentic-awesome-skills` |
| [returns-reverse-logistics-antigravity-awesome-skills-main](./returns-reverse-logistics-antigravity-awesome-skills-main) | Codified expertise for returns authorisation, receipt and inspection, disposition decisions, refund processing | `antigravity-awesome-skills-main` |
| [returns-reverse-logistics-ECC](./returns-reverse-logistics-ECC) | > | `ECC` |
| [returns-reverse-logistics-everything-claude-code-main](./returns-reverse-logistics-everything-claude-code-main) | > | `everything-claude-code-main` |
| [reverse-document](./reverse-document) | Generate design or architecture documents from existing implementation. Works backwards from code/prototypes t | `Claude-Code-Game-Studios-main` |
| [review-all-gdds](./review-all-gdds) | Holistic cross-GDD consistency and game design review. Reads all system GDDs simultaneously and checks for con | `Claude-Code-Game-Studios-main` |
| [review-claudemd](./review-claudemd) | Review recent conversations to find improvements for CLAUDE.md files. | `claude-code-tips-main` |
| [review-multi-agent-orchestration](./review-multi-agent-orchestration) | Use when a supervisor, swarm, graph, planner-worker system, or parallel agent workflow needs review for task b | `agentic-awesome-skills` |
| [review-swarm](./review-swarm) | Parallel read-only multi-agent review of a current git diff or explicit file scope to find behavioral regressi | `agentic-awesome-skills` |
| [review-walkthrough](./review-walkthrough) | Generates an interactive HTML walkthrough for reviewing code changes. Use only when explicitly called. | `trailofbits__skills` |
| [ripwire-mcp](./ripwire-mcp) | > | `redhat-et__ripwire` |
| [ripwire-mcp-ripwire](./ripwire-mcp-ripwire) | > | `ripwire` |
| [roast-me](./roast-me) | Use when someone wants an honest, comedic audit of their own prompting — reads their past agent transcripts, s | `ericrisco__rsc-harness` |
| [robot-framework-skill](./robot-framework-skill) | Generates Robot Framework tests in keyword-driven syntax with Python. Supports SeleniumLibrary, RequestsLibrar | `agentic-awesome-skills` |
| [role-creator](./role-creator) | Create and update Codex custom agents using standalone custom-agent TOML files. | `codex-skills-main` |
| [rosa](./rosa) | Route ROSA LangChain agent construction, ROS 1 and ROS 2 | `AREX-Skill` |
| [routerbase-model-gateway](./routerbase-model-gateway) | Integrate RouterBase as an OpenAI-compatible model gateway for routing GPT, Claude, Gemini, media, audio, and  | `agentic-awesome-skills` |
| [rowan](./rowan) | Rowan is a cloud-native molecular modeling and medicinal-chemistry workflow platform with a Python API. Use fo | `scientific-agent-skills` |
| [ruby](./ruby) | Use when writing or refactoring plain Ruby outside Rails — scripts, CLIs, gems, libraries: Enumerable chains a | `ericrisco__rsc-harness` |
| [ruflo-doctor](./ruflo-doctor) | Run health checks on the Ruflo installation and fix common issues | `ruflo` |
| [ruflo-status](./ruflo-status) | Diagnose Ruflo health, then report system, MCP server, and active-agent status without changing the installati | `ruflo` |
| [rules-distill](./rules-distill) | Scan skills to extract cross-cutting principles and distill them into rules — append, revise, or create new ru | `everything-claude-code-main` |
| [run-agent](./run-agent) | > | `Claude-Code-Agent-Monitor` |
| [run-history](./run-history) | > | `Claude-Code-Agent-Monitor` |
| [rust-patterns](./rust-patterns) | Idiomatic Rust patterns, ownership, error handling, traits, concurrency, and best practices for building safe, | `everything-claude-code-main` |
| [rust-testing](./rust-testing) | Rust testing patterns including unit tests, integration tests, async testing, property-based testing, mocking, | `everything-claude-code-main` |
| [ruview-rvagent](./ruview-rvagent) | Explore and prototype rvAgent + RVF integration for RuView agentic flows. Use when working on cross-cog coordi | `RuView` |
| [rvf-manage](./rvf-manage) | Manage RVF (Ruflo Vector Format) files for portable agent memory and cross-platform transfer | `ruflo` |
| [safe-mode](./safe-mode) | Prevent destructive operations using Claude Code hooks. Three modes — cautious (warn on dangerous commands), l | `pro-workflow` |
| [safety-guard](./safety-guard) | Use this skill to prevent destructive operations when working on production systems or running agents autonomo | `everything-claude-code-main` |
| [salesforce-automation](./salesforce-automation) | Automate Salesforce tasks via Rube MCP (Composio): leads, contacts, accounts, opportunities, SOQL queries. Alw | `agentic-awesome-skills` |
| [salesforce-automation-antigravity-awesome-skills-main](./salesforce-automation-antigravity-awesome-skills-main) | Automate Salesforce tasks via Rube MCP (Composio): leads, contacts, accounts, opportunities, SOQL queries. Alw | `antigravity-awesome-skills-main` |
| [sam-altman](./sam-altman) | Agente que simula Sam Altman — CEO da OpenAI, ex-presidente da Y Combinator, arquiteto da era AGI. | `agentic-awesome-skills` |
| [sam-altman-antigravity-awesome-skills-main](./sam-altman-antigravity-awesome-skills-main) | Agente que simula Sam Altman — CEO da OpenAI, ex-presidente da Y Combinator, arquiteto da era AGI. | `antigravity-awesome-skills-main` |
| [sandbox-claude-inside](./sandbox-claude-inside) | Run Claude Code INSIDE a ca-sandbox box (`--with-claude`). Routed to when the user wants an agent loop running | `arbiterForge__codeArbiter` |
| [santa-method](./santa-method) | Multi-agent adversarial verification with convergence loop. Two independent review agents must both pass befor | `ECC` |
| [santa-method-everything-claude-code-main](./santa-method-everything-claude-code-main) | Multi-agent adversarial verification with convergence loop. Two independent review agents must both pass befor | `everything-claude-code-main` |
| [save-task-list](./save-task-list) | Save current task list for reuse across sessions | `Archon-dev` |
| [scaffold-exercises](./scaffold-exercises) | Create exercise directory structures with sections, problems, solutions, and explainers that pass linting. Use | `mattpocock__skills` |
| [scanpy](./scanpy) | Scanpy is a scalable Python toolkit for analyzing single-cell RNA-seq data, built on AnnData. Apply this skill | `agentic-awesome-skills` |
| [scanpy-antigravity-awesome-skills-main](./scanpy-antigravity-awesome-skills-main) | Scanpy is a scalable Python toolkit for analyzing single-cell RNA-seq data, built on AnnData. Apply this skill | `antigravity-awesome-skills-main` |
| [scanpy-scientific-agent-skills](./scanpy-scientific-agent-skills) | Standard single-cell RNA-seq analysis pipeline. Use for QC, normalization, dimensionality reduction (PCA/UMAP/ | `scientific-agent-skills` |
| [schema-markup-generator](./schema-markup-generator) | Generate and implement JSON-LD structured data for web apps, blogs, FAQs, and SaaS sites. Supports WebSite, So | `agentic-awesome-skills` |
| [scholar-evaluation](./scholar-evaluation) | Provide qualitative-first, evidence-traceable developmental review of scholarly works and audit low-stakes res | `scientific-agent-skills` |
| [scientific-agent-skills](./scientific-agent-skills) | Maintains the Scientific Agent Skills repository: adding or | `AREX-Skill` |
| [scientific-brainstorming](./scientific-brainstorming) | Facilitates evidence-aware scientific ideation with independent generation, structured discussion, explicit as | `scientific-agent-skills` |
| [scientific-critical-thinking](./scientific-critical-thinking) | Evaluate scientific claims and evidence quality. Use for assessing experimental design validity, identifying b | `scientific-agent-skills` |
| [scientific-schematics](./scientific-schematics) | Create publication-quality scientific diagrams using Nano Banana 2 AI with smart iterative refinement. Uses Ge | `scientific-agent-skills` |
| [scientific-visualization](./scientific-visualization) | Create and audit truthful, accessible, publication-ready scientific figures with Matplotlib, Seaborn, or Plotl | `scientific-agent-skills` |
| [scientific-writing](./scientific-writing) | This is the core skill for the deep research and writing tool—combining AI-driven deep research with well-form | `agentic-awesome-skills` |
| [scientific-writing-antigravity-awesome-skills-main](./scientific-writing-antigravity-awesome-skills-main) | This is the core skill for the deep research and writing tool—combining AI-driven deep research with well-form | `antigravity-awesome-skills-main` |
| [scientific-writing-scientific-agent-skills](./scientific-writing-scientific-agent-skills) | Draft, revise, and audit scientific manuscripts or reports with explicit evidence provenance, reporting-guidel | `scientific-agent-skills` |
| [scikit-bio](./scikit-bio) | Biological data toolkit. Sequence analysis, alignments, phylogenetic trees, diversity metrics (alpha/beta, Uni | `scientific-agent-skills` |
| [scikit-learn](./scikit-learn) | Machine learning in Python with scikit-learn. Use for classification, regression, clustering, model evaluation | `agentic-awesome-skills` |
| [scikit-learn-antigravity-awesome-skills-main](./scikit-learn-antigravity-awesome-skills-main) | Machine learning in Python with scikit-learn. Use for classification, regression, clustering, model evaluation | `antigravity-awesome-skills-main` |
| [scikit-learn-scientific-agent-skills](./scikit-learn-scientific-agent-skills) | Machine learning in Python with scikit-learn. Use when working with supervised learning (classification, regre | `scientific-agent-skills` |
| [scikit-survival](./scikit-survival) | Build, evaluate, and audit right-censored or competing-risk survival workflows with scikit-survival, including | `scientific-agent-skills` |
| [scope-check](./scope-check) | Analyze a feature or sprint for scope creep by comparing current scope against the original plan. Flags additi | `Claude-Code-Game-Studios-main` |
| [scvi-tools](./scvi-tools) | Deep generative models for single-cell omics. Use when you need probabilistic batch correction (scVI), transfe | `scientific-agent-skills` |
| [sdlc-review](./sdlc-review) | Review Kanban handoffs and route verified outcomes. | `hermes-agent` |
| [seaborn](./seaborn) | Seaborn is a Python visualization library for creating publication-quality statistical graphics. Use this skil | `agentic-awesome-skills` |
| [seaborn-antigravity-awesome-skills-main](./seaborn-antigravity-awesome-skills-main) | Seaborn is a Python visualization library for creating publication-quality statistical graphics. Use this skil | `antigravity-awesome-skills-main` |
| [seaborn-scientific-agent-skills](./seaborn-scientific-agent-skills) | Statistical visualization with pandas integration. Use for quick exploration of distributions, relationships,  | `scientific-agent-skills` |
| [search-first](./search-first) | Research-before-coding workflow. Search for existing tools, libraries, and patterns before writing custom code | `everything-claude-code-main` |
| [search-tools](./search-tools) | Search Tool Hierarchy | `Continuous-Claude-v3-main` |
| [search-tools-Continuous-Claude-v3-main](./search-tools-Continuous-Claude-v3-main) | Search Tool Hierarchy | `Continuous-Claude-v3-main` |
| [second-opinion](./second-opinion) | Load for a cross-model review, challenge, audit, or quick consult. | `kendex` |
| [second-opinion-kendex](./second-opinion-kendex) | Load for a cross-model review, challenge, audit, or quick consult. | `kendex` |
| [segment-automation](./segment-automation) | Automate Segment tasks via Rube MCP (Composio): track events, identify users, manage groups, page views, alias | `agentic-awesome-skills` |
| [segment-automation-antigravity-awesome-skills-main](./segment-automation-antigravity-awesome-skills-main) | Automate Segment tasks via Rube MCP (Composio): track events, identify users, manage groups, page views, alias | `antigravity-awesome-skills-main` |
| [selenium-skill](./selenium-skill) | Generates production-grade Selenium WebDriver automation scripts and tests in Java, Python, JavaScript, C#, Ru | `agentic-awesome-skills` |
| [semantica](./semantica) | Semantica full-stack knowledge graph skill for context graphs, decision intelligence, explainability, extracti | `semantica` |
| [sendgrid-automation](./sendgrid-automation) | Automate SendGrid email delivery workflows including marketing campaigns (Single Sends), contact and list mana | `agentic-awesome-skills` |
| [sendgrid-automation-antigravity-awesome-skills-main](./sendgrid-automation-antigravity-awesome-skills-main) | Automate SendGrid email delivery workflows including marketing campaigns (Single Sends), contact and list mana | `antigravity-awesome-skills-main` |
| [sentry-automation](./sentry-automation) | Automate Sentry tasks via Rube MCP (Composio): manage issues/events, configure alerts, track releases, monitor | `agentic-awesome-skills` |
| [sentry-automation-antigravity-awesome-skills-main](./sentry-automation-antigravity-awesome-skills-main) | Automate Sentry tasks via Rube MCP (Composio): manage issues/events, configure alerts, track releases, monitor | `antigravity-awesome-skills-main` |
| [sentry-fix-issues](./sentry-fix-issues) | Find and fix issues from Sentry using MCP. Use when asked to fix Sentry errors, debug production issues, inves | `openclaw__clawhub` |
| [seo](./seo) | Run a broad SEO audit across technical SEO, on-page SEO, schema, sitemaps, content quality, AI search readines | `agentic-awesome-skills` |
| [seo-antigravity-awesome-skills-main](./seo-antigravity-awesome-skills-main) | Run a broad SEO audit across technical SEO, on-page SEO, schema, sitemaps, content quality, AI search readines | `antigravity-awesome-skills-main` |
| [seo-audit](./seo-audit) | Full website SEO audit with parallel subagent delegation. Crawls up to 500 pages, detects business type, deleg | `claude-seo` |
| [seo-dataforseo](./seo-dataforseo) | Use DataForSEO for live SERPs, keyword metrics, backlinks, competitor analysis, on-page checks, and AI visibil | `agentic-awesome-skills` |
| [seo-dataforseo-antigravity-awesome-skills-main](./seo-dataforseo-antigravity-awesome-skills-main) | Use DataForSEO for live SERPs, keyword metrics, backlinks, competitor analysis, on-page checks, and AI visibil | `antigravity-awesome-skills-main` |
| [seo-dataforseo-claude-seo](./seo-dataforseo-claude-seo) | > | `claude-seo` |
| [seo-everything-claude-code-main](./seo-everything-claude-code-main) | Audit, plan, and implement SEO improvements across technical SEO, on-page optimization, structured data, Core  | `everything-claude-code-main` |
| [session-cleanup](./session-cleanup) | > | `Claude-Code-Agent-Monitor` |
| [session-compare](./session-compare) | > | `Claude-Code-Agent-Monitor` |
| [session-debug](./session-debug) | > | `Claude-Code-Agent-Monitor` |
| [session-report](./session-report) | Generate an explorable HTML report of Claude Code session usage (tokens, cache, subagents, skills, expensive p | `claude-plugins-official-main` |
| [session-search](./session-search) | > | `Claude-Code-Agent-Monitor` |
| [session-share](./session-share) | Share Claude Code sessions between developers. Use when user mentions "share session", "export session", "impo | `agent-deck-main` |
| [setup-engine](./setup-engine) | Configure the project's game engine and version. Pins the engine in CLAUDE.md, detects knowledge gaps, and pop | `Claude-Code-Game-Studios-main` |
| [setup-matt-pocock-skills](./setup-matt-pocock-skills) | Configure this repo for the engineering skills — set up its issue tracker, triage label vocabulary, and domain | `agentic-awesome-skills` |
| [setup-matt-pocock-skills-mattpocock__skills](./setup-matt-pocock-skills-mattpocock__skills) | Configure this repo for the engineering skills: set up its issue tracker, triage label vocabulary, and domain  | `mattpocock__skills` |
| [shap](./shap) | Explain and audit machine-learning predictions with SHAP. Use for selecting SHAP explainers and maskers, compu | `scientific-agent-skills` |
| [shipping-and-launch](./shipping-and-launch) | Prepares production launches. Use when preparing to deploy to production. Use when you need a pre-launch check | `agentic-awesome-skills` |
| [shopify-automation](./shopify-automation) | Automate Shopify tasks via Rube MCP (Composio): products, orders, customers, inventory, collections. Always se | `agentic-awesome-skills` |
| [shopify-automation-antigravity-awesome-skills-main](./shopify-automation-antigravity-awesome-skills-main) | Automate Shopify tasks via Rube MCP (Composio): products, orders, customers, inventory, collections. Always se | `antigravity-awesome-skills-main` |
| [simplify-code](./simplify-code) | Parallel 4-agent cleanup of recent code changes. | `hermes-agent` |
| [simpy](./simpy) | Build, inspect, test, and analyze bounded process-based discrete-event simulations with SimPy, including event | `scientific-agent-skills` |
| [simulation-trading](./simulation-trading) | 模拟交易 — 下单买卖、持仓管理、成交查询、账户状态、资金快照。在 QuantBot / Claude Code 中执行模拟交易、下单、查询持仓、查看账户、管理交易记录时使用。触发词：模拟交易、模拟下单、买入股票、卖出股 | `QuantMind` |
| [skill-agent-topology](./skill-agent-topology) | Audit whether a multi-agent setup earns its coordination cost — use before adding an agent, or when a workflow | `claude-octopus` |
| [skill-agent-topology-claude-octopus](./skill-agent-topology-claude-octopus) | Audit whether a multi-agent setup earns its coordination cost — use before adding an agent, or when a workflow | `claude-octopus` |
| [skill-audit](./skill-audit) | Audit codebases for quality, consistency, and broken patterns — use for pre-release or tech debt review | `claude-octopus` |
| [skill-author](./skill-author) | The authoring gate for new skills. Routed to when the user invokes /new-skill "<gap>". Five gated phases — gap | `arbiterForge__codeArbiter` |
| [skill-authoring](./skill-authoring) | Principles for writing skills that behave the same way every run — use when adding, editing, or reviewing a sk | `claude-octopus` |
| [skill-authoring-claude-octopus](./skill-authoring-claude-octopus) | Principles for writing skills that behave the same way every run — use when adding, editing, or reviewing a sk | `claude-octopus` |
| [skill-authoring-headcount](./skill-authoring-headcount) | Writes and revises agent skills so they trigger at the right moments and give usable instruction when they do. | `headcount` |
| [skill-auto-improver](./skill-auto-improver) | Improve an external, legacy, or drifted SKILL.md to the skill-creator standard — hard validation gates plus an | `luongnv89__asm` |
| [skill-best-practices-sync](./skill-best-practices-sync) | > | `dotfiles` |
| [skill-builder](./skill-builder) | Create new Claude Code Skills with proper YAML frontmatter, progressive disclosure structure, and complete dir | `ruflo` |
| [skill-builder-ruflo](./skill-builder-ruflo) | Create new Claude Code Skills with proper YAML frontmatter, progressive disclosure structure, and complete dir | `ruflo` |
| [skill-builder-ruflo-main](./skill-builder-ruflo-main) | Create new Claude Code Skills with proper YAML frontmatter, progressive disclosure structure, and complete dir | `ruflo-main` |
| [skill-builder-ruflo-main](./skill-builder-ruflo-main) | Create new Claude Code Skills with proper YAML frontmatter, progressive disclosure structure, and complete dir | `ruflo-main` |
| [skill-check](./skill-check) | Validate Claude Code skills against the agentskills specification. Catches structural, semantic, and naming is | `agentic-awesome-skills` |
| [skill-check-antigravity-awesome-skills-main](./skill-check-antigravity-awesome-skills-main) | Validate Claude Code skills against the agentskills specification. Catches structural, semantic, and naming is | `antigravity-awesome-skills-main` |
| [skill-code-review](./skill-code-review) | Expert multi-AI code review with inline PR comments — use for thorough quality and security analysis | `claude-octopus` |
| [skill-comply](./skill-comply) | Visualize whether skills, rules, and agent definitions are actually followed — auto-generates scenarios at 3 p | `everything-claude-code-main` |
| [skill-context-detection](./skill-context-detection) | Auto-detect work context (Dev vs Knowledge) — use to tailor workflows based on current task type | `claude-octopus` |
| [skill-council](./skill-council) | Run a configurable multi-LLM council with personas, budget caps, synthesis, veto gates, and optional implement | `claude-octopus` |
| [skill-coverage-audit](./skill-coverage-audit) | Trace codepaths in diffs, map against tests, auto-generate missing coverage — use before shipping PRs | `claude-octopus` |
| [skill-creator](./skill-creator) | To create new CLI skills following Anthropic's official best practices with zero manual configuration. This sk | `agentic-awesome-skills` |
| [skill-creator-aiox-core-main](./skill-creator-aiox-core-main) | Guide for creating effective skills. This skill should be used when users want to create a new skill (or updat | `aiox-core-main` |
| [skill-creator-anthropics__skills](./skill-creator-anthropics__skills) | Create new skills, modify and improve existing skills, and measure skill performance. Use when users want to c | `anthropics__skills` |
| [skill-creator-antigravity-awesome-skills-main](./skill-creator-antigravity-awesome-skills-main) | To create new CLI skills following Anthropic's official best practices with zero manual configuration. This sk | `antigravity-awesome-skills-main` |
| [skill-creator-Auto-Company](./skill-creator-Auto-Company) | Guide for creating effective skills. This skill should be used when users want to create a new skill (or updat | `Auto-Company` |
| [skill-creator-autonomous-os](./skill-creator-autonomous-os) | Create new skills, modify and improve existing skills, and measure skill performance. Use when users want to c | `autonomous-os` |
| [skill-creator-bb](./skill-creator-bb) | Create or improve BB skills, including their triggers, instructions, and supporting resources. | `bb` |
| [skill-creator-claude-plugins-official-main](./skill-creator-claude-plugins-official-main) | Create new skills, modify and improve existing skills, and measure skill performance. Use when users want to c | `claude-plugins-official-main` |
| [skill-creator-CowAgent](./skill-creator-CowAgent) | Create, install, or update skills in the workspace. Use when (1) installing a skill from a URL or remote sourc | `CowAgent` |
| [skill-creator-deer-flow](./skill-creator-deer-flow) | Create new skills, modify and improve existing skills, and measure skill performance. Use when users want to c | `deer-flow` |
| [skill-creator-EvoSkill-main](./skill-creator-EvoSkill-main) | Guide for creating effective skills. This skill should be used when users want to create a new skill (or updat | `EvoSkill-main` |
| [skill-creator-fastclaw](./skill-creator-fastclaw) | Create new skills, modify and improve existing skills, and measure skill performance. Use when users want to c | `fastclaw` |
| [skill-creator-gbrain](./skill-creator-gbrain) | \| | `gbrain` |
| [skill-creator-luongnv89__asm](./skill-creator-luongnv89__asm) | Create, improve, evaluate, benchmark skills. Use when authoring a new skill, updating an existing one, running | `luongnv89__asm` |
| [skill-creator-ms](./skill-creator-ms) | Guide for creating effective skills for AI coding agents working with Azure SDKs and Microsoft Foundry service | `agentic-awesome-skills` |
| [skill-creator-ms-antigravity-awesome-skills-main](./skill-creator-ms-antigravity-awesome-skills-main) | Guide for creating effective skills for AI coding agents working with Azure SDKs and Microsoft Foundry service | `antigravity-awesome-skills-main` |
| [skill-creator-OpenBMB__PilotDeck](./skill-creator-OpenBMB__PilotDeck) | Create new skills, modify and improve existing skills, and measure skill performance. Use when users want to c | `OpenBMB__PilotDeck` |
| [skill-creator-openclaw](./skill-creator-openclaw) | Author or review AgentSkills: create, repair, validate, or restructure SKILL.md files and bundled resources. | `openclaw` |
| [skill-creator-understudy-ai__understudy](./skill-creator-understudy-ai__understudy) | Create, edit, improve, review, audit, or clean up AgentSkills. Use when creating a new skill from scratch or w | `understudy-ai__understudy` |
| [skill-debate](./skill-debate) | Structured multi-provider AI debates between Claude and available advisors — use for critical decisions | `claude-octopus` |
| [skill-debug](./skill-debug) | Debug a reproducible symptom with a bounded feedback loop and original-scenario verification | `claude-octopus` |
| [skill-decision-support](./skill-decision-support) | Present options with trade-offs for informed decision-making — use when choosing between approaches | `claude-octopus` |
| [skill-deck](./skill-deck) | Generate slide deck presentations from briefs — use when you need slides, pitch decks, or visual summaries | `claude-octopus` |
| [skill-deck-claude-octopus](./skill-deck-claude-octopus) | Generate slide deck presentations from briefs — use when you need slides, pitch decks, or visual summaries | `claude-octopus` |
| [skill-design-lineage](./skill-design-lineage) | Persist design documents with branch tracking, revision chains, and cross-session discovery | `claude-octopus` |
| [skill-developer](./skill-developer) | Comprehensive guide for creating and managing skills in Claude Code with auto-activation system, following Ant | `agentic-awesome-skills` |
| [skill-developer-antigravity-awesome-skills-main](./skill-developer-antigravity-awesome-skills-main) | Comprehensive guide for creating and managing skills in Claude Code with auto-activation system, following Ant | `antigravity-awesome-skills-main` |
| [skill-developer-Continuous-Claude-v3-main](./skill-developer-Continuous-Claude-v3-main) | Meta-skill for creating and managing Claude Code skills | `Continuous-Claude-v3-main` |
| [skill-development](./skill-development) | This skill should be used when the user wants to "create a skill", "add a skill to plugin", "write a new skill | `claude-plugins-official-main` |
| [skill-development-Continuous-Claude-v3-main](./skill-development-Continuous-Claude-v3-main) | Skill Development Rules | `Continuous-Claude-v3-main` |
| [skill-doc-delivery](./skill-doc-delivery) | Convert markdown to DOCX, PPTX, XLSX, PDF office documents — use when you need exportable deliverables | `claude-octopus` |
| [skill-doc-sync](./skill-doc-sync) | Post-ship doc sync across project markdown. Use when: sync docs, update docs, document changes, release notes. | `claude-octopus` |
| [skill-doctor](./skill-doctor) | Environment diagnostics — check providers, auth, config, hooks, scheduler, and more | `claude-octopus` |
| [skill-factory](./skill-factory) | Run a full build-and-ship pipeline from a spec — use for hands-off project generation | `claude-octopus` |
| [skill-factory-claude-octopus](./skill-factory-claude-octopus) | Run a full build-and-ship pipeline from a spec — use for hands-off project generation | `claude-octopus` |
| [skill-finish-branch](./skill-finish-branch) | Wrap up a branch — run tests, create PR, merge or discard — use when implementation is done | `claude-octopus` |
| [skill-improve](./skill-improve) | Improve a skill using a test-fix-retest loop. Runs static checks, proposes targeted fixes, rewrites the skill, | `Claude-Code-Game-Studios-main` |
| [skill-improver](./skill-improver) | Iteratively improve a Claude Code skill using the skill-reviewer agent until it meets quality standards. Use w | `agentic-awesome-skills` |
| [skill-improver-antigravity-awesome-skills-main](./skill-improver-antigravity-awesome-skills-main) | Iteratively improve a Claude Code skill using the skill-reviewer agent until it meets quality standards. Use w | `antigravity-awesome-skills-main` |
| [skill-improver-trailofbits__skills](./skill-improver-trailofbits__skills) | Runs an autonomous review-and-fix improvement loop over a Claude Code skill until a review comes back clean, w | `trailofbits__skills` |
| [skill-index-updater](./skill-index-updater) | Add GitHub skill repos to the ASM index: clone, audit, eval, regenerate index, rebuild catalog, open PR. Use w | `luongnv89__asm` |
| [skill-inspector](./skill-inspector) | Review AI agent skills before installation using NVIDIA SkillSpector and source-aware semantic review. Use whe | `SkillSpector` |
| [skill-install-improved](./skill-install-improved) | Install an improved variant of one named skill: resolve it by local path, repo, or name, run skill-auto-improv | `luongnv89__asm` |
| [skill-installer](./skill-installer) | Instala, valida, registra e verifica novas skills no ecossistema. 10 checks de seguranca, copia, registro no o | `agentic-awesome-skills` |
| [skill-installer-antigravity-awesome-skills-main](./skill-installer-antigravity-awesome-skills-main) | Instala, valida, registra e verifica novas skills no ecossistema. 10 checks de seguranca, copia, registro no o | `antigravity-awesome-skills-main` |
| [skill-intake](./skill-intake) | Move incoming issues and pull requests through triage states until each is actionable or closed — use when the | `claude-octopus` |
| [skill-intent-contract](./skill-intent-contract) | Use when starting a complex or ambiguous task that risks scope drift | `claude-octopus` |
| [skill-inventory](./skill-inventory) | > | `Claude-Code-Agent-Monitor` |
| [skill-issues](./skill-issues) | Track project blockers, bugs, and gaps across sessions — use when issues pile up or need triage | `claude-octopus` |
| [skill-issues-claude-octopus](./skill-issues-claude-octopus) | Track project blockers, bugs, and gaps across sessions — use when issues pile up or need triage | `claude-octopus` |
| [skill-iterative-loop](./skill-iterative-loop) | Run tasks in a loop until goals are met — use for iterative refinement, polling, or convergence | `claude-octopus` |
| [skill-knowledge-work](./skill-knowledge-work) | Switch to Knowledge Work mode for research and writing — use when task is non-code focused | `claude-octopus` |
| [skill-meta-prompt](./skill-meta-prompt) | Craft better prompts using proven optimization techniques — use when your prompt needs refinement | `claude-octopus` |
| [skill-parallel-agents](./skill-parallel-agents) | Decompose large tasks across parallel agents — use for migrations, multi-file refactors, or batch work | `claude-octopus` |
| [skill-prd](./skill-prd) | Write an AI-optimized PRD using multi-AI orchestration — use when scoping a new feature or product | `claude-octopus` |
| [skill-pressure-test](./skill-pressure-test) | Interrogate a plan, decision, or design one question at a time until it holds — use to stress-test your own th | `claude-octopus` |
| [skill-prototype](./skill-prototype) | Prototype one risky assumption within a fixed budget, then keep or discard the result | `claude-octopus` |
| [skill-resume](./skill-resume) | Pick up where you left off from a previous session — use after context resets, compaction, or new conversation | `claude-octopus` |
| [skill-review-response](./skill-review-response) | Use when a reviewer, CI bot, or another AI leaves feedback to address | `claude-octopus` |
| [skill-reviewer](./skill-reviewer) | Reviews DeerFlow skill packages for readiness, triggers, safety boundaries, resources, and evidence. Invoke wh | `deer-flow` |
| [skill-rollback](./skill-rollback) | Roll back to a previous checkpoint via git — use when a change went wrong and you need to revert | `claude-octopus` |
| [skill-router](./skill-router) | The index of every pro-workflow skill and command, grouped by job, with when to reach for each and whether it  | `pro-workflow` |
| [skill-security-framing](./skill-security-framing) | URL validation and content sanitization for untrusted sources — use when handling external input safely | `claude-octopus` |
| [skill-sentinel](./skill-sentinel) | Auditoria e evolucao do ecossistema de skills. Qualidade de codigo, seguranca, custos, gaps, duplicacoes, depe | `agentic-awesome-skills` |
| [skill-sentinel-antigravity-awesome-skills-main](./skill-sentinel-antigravity-awesome-skills-main) | Auditoria e evolucao do ecossistema de skills. Qualidade de codigo, seguranca, custos, gaps, duplicacoes, depe | `antigravity-awesome-skills-main` |
| [skill-shortener](./skill-shortener) | Refactor a too-long SKILL.md by progressive disclosure: measure token cost, classify every section KEEP/CUT/MO | `luongnv89__asm` |
| [skill-staged-review](./skill-staged-review) | Use when a PR or feature needs both specification and code-quality review | `claude-octopus` |
| [skill-status](./skill-status) | Show where you are in the workflow and what to do next — use for progress checks and orientation | `claude-octopus` |
| [skill-stocktake](./skill-stocktake) | Use when auditing Claude skills and commands for quality. Supports Quick Scan (changed skills only) and Full S | `ECC` |
| [skill-stocktake-everything-claude-code-main](./skill-stocktake-everything-claude-code-main) | Use when auditing Claude skills and commands for quality. Supports Quick Scan (changed skills only) and Full S | `everything-claude-code-main` |
| [skill-suggester](./skill-suggester) | Scan prompt history for recurring patterns and unmet needs, then propose new skills or command templates | `agentic-awesome-skills` |
| [skill-task-management](./skill-task-management) | Manage tasks with Claude Code native tools — use to track TODOs, delegate work, and monitor progress | `claude-octopus` |
| [skill-task-management-v2](./skill-task-management-v2) | Manage tasks with Claude Code native tools — use to track TODOs, delegate work, and monitor progress | `claude-octopus` |
| [skill-tdd](./skill-tdd) | Build a behavior change with observed red, minimal green, and measured test consolidation | `claude-octopus` |
| [skill-test](./skill-test) | Validate skill files for structural compliance and behavioral correctness. Three modes: static (linter), spec  | `Claude-Code-Game-Studios-main` |
| [skill-thought-partner](./skill-thought-partner) | Brainstorm creatively with pattern spotting and paradox hunting — use for ideation and exploration | `claude-octopus` |
| [skill-thought-partner-claude-octopus](./skill-thought-partner-claude-octopus) | Brainstorm creatively with pattern spotting and paradox hunting — use for ideation and exploration | `claude-octopus` |
| [skill-upgrader](./skill-upgrader) | Upgrade any skill to v5 Hybrid format using decision theory + modal logic | `Continuous-Claude-v3-main` |
| [skill-upstream-pr](./skill-upstream-pr) | Improve an open-source GitHub skill and open a friendly suggestion PR upstream: fork, run skill-auto-improver, | `luongnv89__asm` |
| [skill-verification-gate](./skill-verification-gate) | Use when about to declare work complete, fixed, passing, or done | `claude-octopus` |
| [skill-verification-gate-claude-octopus](./skill-verification-gate-claude-octopus) | Use when about to declare work complete, fixed, passing, or done | `claude-octopus` |
| [skill-verify](./skill-verify) | Use when a nontrivial change needs end-to-end verification before committing or shipping | `claude-octopus` |
| [skill-visual-feedback](./skill-visual-feedback) | Process screenshot-based UI/UX feedback to fix visual issues — use when users share screenshots of bugs | `claude-octopus` |
| [skill-work-slicing](./skill-work-slicing) | Break a plan or spec into vertical slices that each declare what blocks them — use when work is agreed but not | `claude-octopus` |
| [skill-writer](./skill-writer) | Create and improve agent skills following the Agent Skills specification. Use when asked to create, write, or  | `agentic-awesome-skills` |
| [skill-writer-antigravity-awesome-skills-main](./skill-writer-antigravity-awesome-skills-main) | Create and improve agent skills following the Agent Skills specification. Use when asked to create, write, or  | `antigravity-awesome-skills-main` |
| [skill-writing-plans](./skill-writing-plans) | Create zero-context implementation plans with bite-sized tasks — use for multi-step feature planning | `claude-octopus` |
| [skill-writing-plans-claude-octopus](./skill-writing-plans-claude-octopus) | Create zero-context implementation plans with bite-sized tasks — use for multi-step feature planning | `claude-octopus` |
| [skillpack-harvest](./skillpack-harvest) | \| | `gbrain` |
| [skills](./skills) | Story Creation and Translation AI Agent with Studio Chat, CLI, and TUI - use for long-form novels, short ficti | `inkos` |
| [skills-audit](./skills-audit) | > | `dotfiles` |
| [skrl](./skrl) | Route public skrl 2.1.0 reinforcement-learning workflows across | `AREX-Skill` |
| [slack-automation](./slack-automation) | Automate Slack workspace operations including messaging, search, channel management, and reaction workflows th | `agentic-awesome-skills` |
| [slack-automation-antigravity-awesome-skills-main](./slack-automation-antigravity-awesome-skills-main) | Automate Slack workspace operations including messaging, search, channel management, and reaction workflows th | `antigravity-awesome-skills-main` |
| [slack-bot-builder](./slack-bot-builder) | Build Slack apps using the Bolt framework across Python, | `agentic-awesome-skills` |
| [slicing-code-context](./slicing-code-context) | Selects bounded, graph-informed source slices with Trailmark and delegates focused code analysis or patch-prop | `trailofbits__skills` |
| [slideops](./slideops) | Turn a repository into a cited HTML slide deck and detect the day it drifts from the code. Citations record fi | `agentic-awesome-skills` |
| [slides](./slides) | Create strategic HTML presentations with Chart.js, design tokens, responsive layouts, copywriting formulas, an | `ui-ux-pro-max-skill` |
| [slo-check](./slo-check) | > | `Claude-Code-Agent-Monitor` |
| [smart-docs](./smart-docs) | AI-powered comprehensive codebase documentation generator. Analyzes project structure, identifies architecture | `sopaco__deepwiki-rs` |
| [smart-git-automation](./smart-git-automation) | Smart change detection, auto branch naming, and streamlined commit/PR workflow | `agentic-awesome-skills` |
| [smart-strategy-stock-picking](./smart-strategy-stock-picking) | 智能策略选股 — 基于 QuantDB 数据的条件选股。在 QuantBot / Claude Code 中按自然语言或条件筛选股票、构建股票池、生成策略时使用。触发词：选股、筛选股票、股票池、条件选股、智能策略、按条件 | `QuantMind` |
| [smarts](./smarts) | Use SMARTS 2.0.1 for multi-agent autonomous-driving simulation, | `AREX-Skill` |
| [smartui-skill](./smartui-skill) | Generates SmartUI visual regression test configurations for screenshot comparison on TestMu AI cloud. Framewor | `agentic-awesome-skills` |
| [smoke-check](./smoke-check) | Run the critical path smoke test gate before QA hand-off. Executes the automated test suite, verifies core fun | `Claude-Code-Game-Studios-main` |
| [soak-test](./soak-test) | Generate a soak test protocol for extended play sessions. Defines what to observe, measure, and log during lon | `Claude-Code-Game-Studios-main` |
| [songsee](./songsee) | Audio spectrograms/features (mel, chroma, MFCC) via CLI. | `hermes-agent` |
| [songwriting-and-ai-music](./songwriting-and-ai-music) | Songwriting craft and Suno AI music prompts. | `hermes-agent` |
| [soul-audit](./soul-audit) | \| | `gbrain` |
| [source-driven-development](./source-driven-development) | Grounds every implementation decision in official documentation. Use when you want authoritative, source-cited | `agentic-awesome-skills` |
| [sparc-methodology](./sparc-methodology) | \| | `ruflo` |
| [sparc-methodology-ruflo-main](./sparc-methodology-ruflo-main) | \| | `ruflo-main` |
| [sparc-methodology-RuView](./sparc-methodology-RuView) | SPARC (Specification, Pseudocode, Architecture, Refinement, Completion) comprehensive development methodology  | `RuView` |
| [sparrow](./sparrow) | Route Sparrow document-intelligence workflows across structured | `AREX-Skill` |
| [spec-driven-development](./spec-driven-development) | Creates specs before coding. Use when starting a new project, feature, or significant change and no specificat | `agentic-awesome-skills` |
| [spec-driven-development-kandev](./spec-driven-development-kandev) | Single-session Kandev feature workflow: clarify intent, create durable requirements, system designs, plans, an | `kandev` |
| [spec-driven-loop](./spec-driven-loop) | Freeze PRD, technical design, and acceptance criteria before medium-to-large Codex work; coordinate agents wit | `agentic-awesome-skills` |
| [spend-forecast](./spend-forecast) | > | `Claude-Code-Agent-Monitor` |
| [spike](./spike) | Throwaway experiments to validate an idea before build. | `hermes-agent` |
| [springboot-patterns](./springboot-patterns) | Spring Boot architecture patterns, REST API design, layered services, data access, caching, async processing,  | `everything-claude-code-main` |
| [springboot-tdd](./springboot-tdd) | Test-driven development for Spring Boot using JUnit 5, Mockito, MockMvc, Testcontainers, and JaCoCo. Use when  | `everything-claude-code-main` |
| [springboot-verification](./springboot-verification) | Verification loop for Spring Boot projects: build, static analysis, tests with coverage, security scans, and d | `everything-claude-code-main` |
| [sprint-plan](./sprint-plan) | Generates a new sprint plan or updates an existing one based on the current milestone, completed work, and ava | `Claude-Code-Game-Studios-main` |
| [sprint-status](./sprint-status) | Fast sprint status check. Reads the current sprint plan, scans story files for status, and produces a concise  | `Claude-Code-Game-Studios-main` |
| [sprint-status-pro-workflow](./sprint-status-pro-workflow) | Track parallel work sessions and prevent confusion across multiple Claude Code instances. Every major step end | `pro-workflow` |
| [sprint-summary](./sprint-summary) | > | `Claude-Code-Agent-Monitor` |
| [square-automation](./square-automation) | Automate Square tasks via Rube MCP (Composio): payments, orders, invoices, locations. Always search tools firs | `agentic-awesome-skills` |
| [square-automation-antigravity-awesome-skills-main](./square-automation-antigravity-awesome-skills-main) | Automate Square tasks via Rube MCP (Composio): payments, orders, invoices, locations. Always search tools firs | `antigravity-awesome-skills-main` |
| [stability-ai](./stability-ai) | Geracao de imagens via Stability AI (SD3.5, Ultra, Core). Text-to-image, img2img, inpainting, upscale, remove- | `agentic-awesome-skills` |
| [stability-ai-antigravity-awesome-skills-main](./stability-ai-antigravity-awesome-skills-main) | Geracao de imagens via Stability AI (SD3.5, Ultra, Core). Text-to-image, img2img, inpainting, upscale, remove- | `antigravity-awesome-skills-main` |
| [stabilize](./stabilize) | USE ONLY WHEN A HUMAN EXPLICITLY INVOKES IT — never auto-select or run this proactively: not after adding a fe | `daintree` |
| [stable-baselines3](./stable-baselines3) | Production-ready reinforcement learning algorithms (PPO, SAC, DQN, TD3, DDPG, A2C) with scikit-learn-like API. | `scientific-agent-skills` |
| [start](./start) | First-time onboarding — asks where you are, then guides you to the right workflow. No assumptions. | `Claude-Code-Game-Studios-main` |
| [statistical-analysis](./statistical-analysis) | Guided statistical analysis for research data - test selection, assumption checking, effect sizes, power analy | `scientific-agent-skills` |
| [statistical-power](./statistical-power) | Sample-size and statistical power calculations for planning studies. Use whenever someone asks "how many subje | `scientific-agent-skills` |
| [statsmodels](./statsmodels) | Statsmodels is Python's premier library for statistical modeling, providing tools for estimation, inference, a | `agentic-awesome-skills` |
| [statsmodels-antigravity-awesome-skills-main](./statsmodels-antigravity-awesome-skills-main) | Statsmodels is Python's premier library for statistical modeling, providing tools for estimation, inference, a | `antigravity-awesome-skills-main` |
| [statsmodels-scientific-agent-skills](./statsmodels-scientific-agent-skills) | Statistical models library for Python. Use when you need specific model classes (OLS, GLM, mixed models, ARIMA | `scientific-agent-skills` |
| [steve-jobs](./steve-jobs) | Agente que simula Steve Jobs — cofundador da Apple, CEO da Pixar, fundador da NeXT, o maior designer de produt | `agentic-awesome-skills` |
| [steve-jobs-antigravity-awesome-skills-main](./steve-jobs-antigravity-awesome-skills-main) | Agente que simula Steve Jobs — cofundador da Apple, CEO da Pixar, fundador da NeXT, o maior designer de produt | `antigravity-awesome-skills-main` |
| [stitch-loop](./stitch-loop) | Teaches agents to iteratively build websites using Stitch with an autonomous baton-passing loop pattern | `agentic-awesome-skills` |
| [stitch-loop-antigravity-awesome-skills-main](./stitch-loop-antigravity-awesome-skills-main) | Teaches agents to iteratively build websites using Stitch with an autonomous baton-passing loop pattern | `antigravity-awesome-skills-main` |
| [stock-market-analysis](./stock-market-analysis) | 股票市场深度数据分析与导出 — 全市场信号扫描、行业轮动、个股研报级深度分析（基本面/估值/技术/资金筹码/情绪/风险六维）、数据挖掘、CSV/Excel 导出。在 QuantBot / Claude Code 中分析股 | `QuantMind` |
| [stock-picks](./stock-picks) | 每日复盘后的股票推荐（多维度选股）— 综合 A股每日复盘（市场方向/板块/资金/新闻情绪）+ 个股深度分析（9层），用多维度筛选条件（L2微观结构为主/模型融合分/仓位信号/板块强度/新闻情绪）从全市场挑出未来几天大概率 | `QuantMind` |
| [stock-research](./stock-research) | 个股深度研究（多 Agent 框架版）— 借鉴 TradingAgents-CN 多角色编排（技术/新闻/资金情绪/基本面/市场 5 分析师并行 → 多空辩论 → 研究经理汇总），数据全部走 QuantMind 本地（Q | `QuantMind` |
| [story-done](./story-done) | End-of-story completion review. Reads the story file, verifies each acceptance criterion against the implement | `Claude-Code-Game-Studios-main` |
| [story-readiness](./story-readiness) | Validate that a story file is implementation-ready. Checks for embedded GDD requirements, ADR references, engi | `Claude-Code-Game-Studios-main` |
| [strands-agents](./strands-agents) | Route Strands Agents monorepo work across the Python SDK, | `AREX-Skill` |
| [strands-review](./strands-review) | Local preview of the strands-agents/devtools `/strands review` agent. Body is the upstream Task Reviewer SOP v | `harness-sdk` |
| [strategic-compact](./strategic-compact) | Suggests manual context compaction at logical intervals to preserve context through task phases rather than ar | `everything-claude-code-main` |
| [strategic-compact-everything-claude-code-main](./strategic-compact-everything-claude-code-main) | Suggests manual context compaction at logical intervals to preserve context through task phases rather than ar | `everything-claude-code-main` |
| [stream-chain](./stream-chain) | Stream-JSON chaining for multi-agent pipelines, data transformation, and sequential workflows | `ruflo` |
| [stream-chain-ruflo](./stream-chain-ruflo) | Stream-JSON chaining for multi-agent pipelines, data transformation, and sequential workflows | `ruflo` |
| [stream-chain-ruflo-main](./stream-chain-ruflo-main) | Stream-JSON chaining for multi-agent pipelines, data transformation, and sequential workflows | `ruflo-main` |
| [stream-chain-ruflo-main](./stream-chain-ruflo-main) | Stream-JSON chaining for multi-agent pipelines, data transformation, and sequential workflows | `ruflo-main` |
| [stream-chain-RuView](./stream-chain-RuView) | Stream-JSON chaining for multi-agent pipelines, data transformation, and sequential workflows | `RuView` |
| [stripe-automation](./stripe-automation) | Automate Stripe tasks via Rube MCP (Composio): customers, charges, subscriptions, invoices, products, refunds. | `agentic-awesome-skills` |
| [stripe-automation-antigravity-awesome-skills-main](./stripe-automation-antigravity-awesome-skills-main) | Automate Stripe tasks via Rube MCP (Composio): customers, charges, subscriptions, invoices, products, refunds. | `antigravity-awesome-skills-main` |
| [subagent-driven-development](./subagent-driven-development) | Use when executing implementation plans with independent tasks in the current session | `agentic-awesome-skills` |
| [subagent-driven-development-antigravity-awesome-skills-main](./subagent-driven-development-antigravity-awesome-skills-main) | Use when executing implementation plans with independent tasks in the current session | `antigravity-awesome-skills-main` |
| [subagent-driven-development-arbiterForge__codeArbiter](./subagent-driven-development-arbiterForge__codeArbiter) | The implementation engine. Routed to by /sprint (full plan, autonomous) and by executing-plans (scoped batch,  | `arbiterForge__codeArbiter` |
| [subagent-driven-development-hermes-agent-main](./subagent-driven-development-hermes-agent-main) | Use when executing implementation plans with independent tasks. Dispatches fresh delegate_task per task with t | `hermes-agent-main` |
| [subagent-driven-development-superpowers-main](./subagent-driven-development-superpowers-main) | Use when executing implementation plans with independent tasks in the current session | `superpowers-main` |
| [subagent-orchestrator](./subagent-orchestrator) | Coordinate quota-aware parallel subagents for large, multi-file Antigravity tasks. | `agentic-awesome-skills` |
| [subagents](./subagents) | Invoke this skill when the user asks to use subagents or when a substantial, self-contained task can run indep | `openpi` |
| [supabase](./supabase) | Supabase / PostgREST Row-Level-Security playbook — pull the anon (or leaked service_role) key out of the front | `agent` |
| [supabase-automation](./supabase-automation) | Automate Supabase database queries, table management, project administration, storage, edge functions, and SQL | `agentic-awesome-skills` |
| [supabase-automation-antigravity-awesome-skills-main](./supabase-automation-antigravity-awesome-skills-main) | Automate Supabase database queries, table management, project administration, storage, edge functions, and SQL | `antigravity-awesome-skills-main` |
| [supabase-postgres-best-practices](./supabase-postgres-best-practices) | Postgres performance optimization and best practices from Supabase. Use this skill when writing, reviewing, or | `agentic-awesome-skills` |
| [super-agi](./super-agi) | Routes SuperAGI autonomous-agent framework tasks across | `AREX-Skill` |
| [super-swarm-spark](./super-swarm-spark) | > | `codex-skills-main` |
| [survey-generator](./survey-generator) | Generate source-backed AI/ML survey paper artifacts with curated bibliographies and Fireworks/Kimi HTML render | `agentic-awesome-skills` |
| [swarm-advanced](./swarm-advanced) | \| | `ruflo` |
| [swarm-advanced-ruflo](./swarm-advanced-ruflo) | Advanced swarm orchestration patterns for research, development, testing, and complex distributed workflows | `ruflo` |
| [swarm-advanced-ruflo-main](./swarm-advanced-ruflo-main) | \| | `ruflo-main` |
| [swarm-advanced-ruflo-main](./swarm-advanced-ruflo-main) | Advanced swarm orchestration patterns for research, development, testing, and complex distributed workflows | `ruflo-main` |
| [swarm-advanced-RuView](./swarm-advanced-RuView) | Advanced swarm orchestration patterns for research, development, testing, and complex distributed workflows | `RuView` |
| [swarm-orchestration](./swarm-orchestration) | Orchestrate multi-agent swarms with agentic-flow for parallel task execution, dynamic topology, and intelligen | `ruflo` |
| [swarm-orchestration-ruflo](./swarm-orchestration-ruflo) | > | `ruflo` |
| [swarm-orchestration-ruflo-main](./swarm-orchestration-ruflo-main) | Orchestrate multi-agent swarms with agentic-flow for parallel task execution, dynamic topology, and intelligen | `ruflo-main` |
| [swarm-orchestration-ruflo-main](./swarm-orchestration-ruflo-main) | > | `ruflo-main` |
| [swarm-orchestration-RuView](./swarm-orchestration-RuView) | Orchestrate multi-agent swarms with agentic-flow for parallel task execution, dynamic topology, and intelligen | `RuView` |
| [swarm-planner](./swarm-planner) | > | `codex-skills-main` |
| [swarms](./swarms) | Route Swarms users to the right single-agent, CLI, workflow, or | `AREX-Skill` |
| [swarms-swarms](./swarms-swarms) | Build agents and multi-agent systems with the Swarms framework — the Agent class, tools, autonomous loops, mem | `swarms` |
| [sweep](./sweep) | Sweep this conversation into agtx tasks and push them to the kanban board. Use when the user wants to capture, | `agtx` |
| [swift-actor-persistence](./swift-actor-persistence) | Thread-safe data persistence in Swift using actors — in-memory cache with file-backed storage, eliminating dat | `everything-claude-code-main` |
| [swift-concurrency-6-2](./swift-concurrency-6-2) | Swift 6.2 Approachable Concurrency — single-threaded by default, @concurrent for explicit background offloadin | `everything-claude-code-main` |
| [swiftui-design](./swiftui-design) | \| | `abingyyds__open-design` |
| [swiftui-design-open-design](./swiftui-design-open-design) | \| | `open-design` |
| [swiftui-expert-skill](./swiftui-expert-skill) | Write, review, and refactor SwiftUI for iOS or macOS, covering data flow, view composition, performance, ident | `agentic-awesome-skills` |
| [swiftui-expert-skill-antigravity-awesome-skills-main](./swiftui-expert-skill-antigravity-awesome-skills-main) | Write, review, or improve SwiftUI code following best practices for state management, view composition, perfor | `antigravity-awesome-skills-main` |
| [swin-transformer](./swin-transformer) | Use this repo skill for Microsoft Swin-Transformer | `AREX-Skill` |
| [sympy](./sympy) | SymPy is a Python library for symbolic mathematics that enables exact computation using mathematical symbols r | `agentic-awesome-skills` |
| [sympy-antigravity-awesome-skills-main](./sympy-antigravity-awesome-skills-main) | SymPy is a Python library for symbolic mathematics that enables exact computation using mathematical symbols r | `antigravity-awesome-skills-main` |
| [sympy-scientific-agent-skills](./sympy-scientific-agent-skills) | Use when you need exact symbolic math in Python — algebra, calculus, equation solving, symbolic linear algebra | `scientific-agent-skills` |
| [sys-configure](./sys-configure) | Configure Claude Octopus — redirects to /octo:setup interactive wizard | `claude-octopus` |
| [systematic-debugging](./systematic-debugging) | 4-phase root cause debugging: understand bugs before fixing. | `hermes-agent` |
| [systematic-debugging-hermes-agent-main](./systematic-debugging-hermes-agent-main) | Use when encountering any bug, test failure, or unexpected behavior. 4-phase root cause investigation — NO fix | `hermes-agent-main` |
| [takeover](./takeover) | Subdomain takeover playbook — sweep subdomains for dangling CNAMEs / NS records pointing at unclaimed third-pa | `agent` |
| [tamarind](./tamarind) | Access a collection of open-source molecular design and structural biology tools on the Tamarind Bio platform, | `scientific-agent-skills` |
| [task-intelligence](./task-intelligence) | Protocolo de Inteligência Pré-Tarefa — ativa TODOS os agentes relevantes do ecossistema ANTES de executar qual | `agentic-awesome-skills` |
| [task-intelligence-antigravity-awesome-skills-main](./task-intelligence-antigravity-awesome-skills-main) | Protocolo de Inteligência Pré-Tarefa — ativa TODOS os agentes relevantes do ecossistema ANTES de executar qual | `antigravity-awesome-skills-main` |
| [taskflow](./taskflow) | Coordinate multi-step detached tasks as one durable TaskFlow job with owner context, state, waits, and child t | `openclaw` |
| [tasks](./tasks) | Use when an approved plan needs slicing into an ordered task list before any code — the SDD `tasks` phase. Eac | `ericrisco__rsc-harness` |
| [tavily-web](./tavily-web) | Web search, content extraction, crawling, and research capabilities using Tavily API. Use when you need to sea | `agentic-awesome-skills` |
| [tavily-web-antigravity-awesome-skills-main](./tavily-web-antigravity-awesome-skills-main) | Web search, content extraction, crawling, and research capabilities using Tavily API. Use when you need to sea | `antigravity-awesome-skills-main` |
| [tdd](./tdd) | Test-driven development. Use when the user wants to build features or fix bugs test-first, mentions "red-green | `agentic-awesome-skills` |
| [tdd-orchestrator](./tdd-orchestrator) | Master TDD orchestrator specializing in red-green-refactor discipline, multi-agent workflow coordination, and  | `agentic-awesome-skills` |
| [tdd-orchestrator-antigravity-awesome-skills-main](./tdd-orchestrator-antigravity-awesome-skills-main) | Master TDD orchestrator specializing in red-green-refactor discipline, multi-agent workflow coordination, and  | `antigravity-awesome-skills-main` |
| [tdd-workflow](./tdd-workflow) | Use this skill when writing new features, fixing bugs, or refactoring code. Enforces test-driven development w | `everything-claude-code-main` |
| [tdd-workflow-everything-claude-code-main](./tdd-workflow-everything-claude-code-main) | Use this skill when writing new features, fixing bugs, or refactoring code. Enforces test-driven development w | `everything-claude-code-main` |
| [tdd-workflows-tdd-cycle](./tdd-workflows-tdd-cycle) | Use when working with tdd workflows tdd cycle | `agentic-awesome-skills` |
| [tdd-workflows-tdd-cycle-antigravity-awesome-skills-main](./tdd-workflows-tdd-cycle-antigravity-awesome-skills-main) | Use when working with tdd workflows tdd cycle | `antigravity-awesome-skills-main` |
| [tdd-workflows-tdd-red](./tdd-workflows-tdd-red) | Generate failing tests for the TDD red phase to define expected behavior and edge cases. | `agentic-awesome-skills` |
| [tdd-workflows-tdd-red-antigravity-awesome-skills-main](./tdd-workflows-tdd-red-antigravity-awesome-skills-main) | Generate failing tests for the TDD red phase to define expected behavior and edge cases. | `antigravity-awesome-skills-main` |
| [tdd-workflows-tdd-refactor](./tdd-workflows-tdd-refactor) | Use when working with tdd workflows tdd refactor | `agentic-awesome-skills` |
| [tdd-workflows-tdd-refactor-antigravity-awesome-skills-main](./tdd-workflows-tdd-refactor-antigravity-awesome-skills-main) | Use when working with tdd workflows tdd refactor | `antigravity-awesome-skills-main` |
| [tdx-live-trading](./tdx-live-trading) | TDX 通达信实盘交易 + 模拟/实盘全链路实时监控。覆盖：实时推理(L2)、策略选择、自动买卖(下单/挂单/卖出/撤单)、交易记录、持仓查询、桥健康/链路状态监控。用户说「实时推理」「自动买卖」「实盘下单」「挂单」「撤 | `QuantMind` |
| [teach](./teach) | Teach the user a new skill or concept, within this workspace. | `agentic-awesome-skills` |
| [team-agent-orchestration](./team-agent-orchestration) | Run team-based orchestration for agent squads using work items, ownership, agent Kanban, merge gates, and cont | `ECC` |
| [team-audio](./team-audio) | Orchestrate audio team: audio-director + sound-designer + technical-artist + gameplay-programmer for full audi | `Claude-Code-Game-Studios-main` |
| [team-builder](./team-builder) | Interactive agent picker for composing and dispatching parallel teams | `everything-claude-code-main` |
| [team-combat](./team-combat) | Orchestrate the combat team: coordinates game-designer, gameplay-programmer, ai-programmer, technical-artist,  | `Claude-Code-Game-Studios-main` |
| [team-level](./team-level) | Orchestrate level design team: level-designer + narrative-director + world-builder + art-director + systems-de | `Claude-Code-Game-Studios-main` |
| [team-live-ops](./team-live-ops) | Orchestrate the live-ops team for post-launch content planning: coordinates live-ops-designer, economy-designe | `Claude-Code-Game-Studios-main` |
| [team-narrative](./team-narrative) | Orchestrate the narrative team: coordinates narrative-director, writer, world-builder, and level-designer to c | `Claude-Code-Game-Studios-main` |
| [team-polish](./team-polish) | Orchestrate the polish team: coordinates performance-analyst, technical-artist, sound-designer, and qa-tester  | `Claude-Code-Game-Studios-main` |
| [team-qa](./team-qa) | Orchestrate the QA team through a full testing cycle. Coordinates qa-lead (strategy + test plan) and qa-tester | `Claude-Code-Game-Studios-main` |
| [team-release](./team-release) | Orchestrate the release team: coordinates release-manager, qa-lead, devops-engineer, and producer to execute a | `Claude-Code-Game-Studios-main` |
| [team-ui](./team-ui) | Orchestrate the UI team through the full UX pipeline: from UX spec authoring through visual design, implementa | `Claude-Code-Game-Studios-main` |
| [teams-meeting-pipeline](./teams-meeting-pipeline) | Teams meeting summaries, job replay, Graph subscriptions. | `hermes-agent` |
| [tech-debt](./tech-debt) | Track, categorize, and prioritize technical debt across the codebase. Scans for debt indicators, maintains a d | `Claude-Code-Game-Studios-main` |
| [tech-search](./tech-search) | \| | `aiox-core-main` |
| [technical-change-tracker](./technical-change-tracker) | Track code changes with structured JSON records, state machine enforcement, and AI session handoff for bot con | `antigravity-awesome-skills-main` |
| [telegram](./telegram) | Integracao completa com Telegram Bot API. Setup com BotFather, mensagens, webhooks, inline keyboards, grupos,  | `agentic-awesome-skills` |
| [telegram-antigravity-awesome-skills-main](./telegram-antigravity-awesome-skills-main) | Integracao completa com Telegram Bot API. Setup com BotFather, mensagens, webhooks, inline keyboards, grupos,  | `antigravity-awesome-skills-main` |
| [telegram-automation](./telegram-automation) | Automate Telegram tasks via Rube MCP (Composio): send messages, manage chats, share photos/documents, and hand | `agentic-awesome-skills` |
| [telegram-automation-antigravity-awesome-skills-main](./telegram-automation-antigravity-awesome-skills-main) | Automate Telegram tasks via Rube MCP (Composio): send messages, manage chats, share photos/documents, and hand | `antigravity-awesome-skills-main` |
| [test-driven-development](./test-driven-development) | TDD: enforce RED-GREEN-REFACTOR, tests before code. | `hermes-agent` |
| [test-driven-development-hermes-agent-main](./test-driven-development-hermes-agent-main) | Use when implementing any feature or bugfix, before writing implementation code. Enforces RED-GREEN-REFACTOR c | `hermes-agent-main` |
| [test-evidence-review](./test-evidence-review) | Quality review of test files and manual evidence documents. Goes beyond existence checks — evaluates assertion | `Claude-Code-Game-Studios-main` |
| [test-flakiness](./test-flakiness) | Detect non-deterministic (flaky) tests by reading CI run logs or test result history. Aggregates pass rates pe | `Claude-Code-Game-Studios-main` |
| [test-framework-migration-skill](./test-framework-migration-skill) | Migrates and converts test automation scripts between Selenium, Playwright, Puppeteer, and Cypress. | `agentic-awesome-skills` |
| [test-helpers](./test-helpers) | Generate engine-specific test helper libraries for the project's test suite. Reads existing test patterns and  | `Claude-Code-Game-Studios-main` |
| [test-setup](./test-setup) | Scaffold the test framework and CI/CD pipeline for the project's engine. Creates the tests/ directory structur | `Claude-Code-Game-Studios-main` |
| [testing-handbook-generator](./testing-handbook-generator) | Generates Claude Code skills from the Trail of Bits Testing Handbook (appsec.guide), analyzing handbook pages  | `trailofbits__skills` |
| [testng-skill](./testng-skill) | Generates TestNG tests in Java with groups, data providers, parallel execution, XML suite configuration, and l | `agentic-awesome-skills` |
| [tianshou](./tianshou) | Use Tianshou 2.0.1 for PyTorch/Gymnasium deep reinforcement | `AREX-Skill` |
| [tigeropen](./tigeropen) | \| | `QuantMind` |
| [tiktok-automation](./tiktok-automation) | Automate TikTok tasks via Rube MCP (Composio): upload/publish videos, post photos, manage content, and view us | `agentic-awesome-skills` |
| [tiktok-automation-antigravity-awesome-skills-main](./tiktok-automation-antigravity-awesome-skills-main) | Automate TikTok tasks via Rube MCP (Composio): upload/publish videos, post photos, manage content, and view us | `antigravity-awesome-skills-main` |
| [tiledbvcf](./tiledbvcf) | Efficient storage and retrieval of genomic variant data using TileDB. Scalable VCF/BCF ingestion, incremental  | `scientific-agent-skills` |
| [time-of-day](./time-of-day) | > | `Claude-Code-Agent-Monitor` |
| [time-skill](./time-skill) | Display the current time in Pakistan Standard Time (PKT, UTC+5). Use when the user asks for the current time,  | `claude-code-best-practice` |
| [time-skill-claude-code-best-practice-main](./time-skill-claude-code-best-practice-main) | Display the current time in Pakistan Standard Time (PKT, UTC+5). Use when the user asks for the current time,  | `claude-code-best-practice-main` |
| [timesfm-forecasting](./timesfm-forecasting) | Zero-shot time series forecasting with Google's TimesFM foundation model. Use for any univariate time series ( | `scientific-agent-skills` |
| [tmux](./tmux) | Remote-control tmux sessions for interactive CLIs by sending keystrokes and scraping pane output. | `understudy-ai__understudy` |
| [to-issues](./to-issues) | Break a plan, spec, or PRD into independently-grabbable issues on the project issue tracker using tracer-bulle | `agentic-awesome-skills` |
| [to-prd](./to-prd) | Turn the current conversation into a PRD and publish it to the project issue tracker — no interview, just synt | `agentic-awesome-skills` |
| [todoist-automation](./todoist-automation) | Automate Todoist task management, projects, sections, filtering, and bulk operations via Rube MCP (Composio).  | `agentic-awesome-skills` |
| [todoist-automation-antigravity-awesome-skills-main](./todoist-automation-antigravity-awesome-skills-main) | Automate Todoist task management, projects, sections, filtering, and bulk operations via Rube MCP (Composio).  | `antigravity-awesome-skills-main` |
| [token-budget-advisor](./token-budget-advisor) | >- | `everything-claude-code-main` |
| [token-coach](./token-coach) | Plan a token-efficient Claude Code or Codex setup, or get a quick health check. Coaching, not the full audit ( | `token-optimizer` |
| [token-dashboard](./token-dashboard) | Open the Token Optimizer dashboard in your browser (context usage, quality, savings). Use to view the dashboar | `token-optimizer` |
| [token-optimizer](./token-optimizer) | Audit a Claude Code or Codex setup for context-window waste, then fix it and measure the savings. Use when con | `token-optimizer` |
| [tool-design](./tool-design) | This skill should be used when the user asks to "design agent tools", "create tool descriptions", "reduce tool | `Agent-Skills-for-Context-Engineering-main` |
| [tool-design-agentic-awesome-skills](./tool-design-agentic-awesome-skills) | Build tools that agents can use effectively, including architectural reduction patterns. Use when creating new | `agentic-awesome-skills` |
| [tool-design-antigravity-awesome-skills-main](./tool-design-antigravity-awesome-skills-main) | Build tools that agents can use effectively, including architectural reduction patterns. Use when creating new | `antigravity-awesome-skills-main` |
| [torch-geometric](./torch-geometric) | PyTorch Geometric (PyG) for graph neural networks — node/link/graph classification, message passing (GCN, GAT, | `scientific-agent-skills` |
| [torchdrug](./torchdrug) | Build and troubleshoot TorchDrug 0.2.1 workflows for molecular graphs, property prediction, self-supervised pr | `scientific-agent-skills` |
| [touchdesigner-mcp](./touchdesigner-mcp) | Control a running TouchDesigner instance via twozero MCP — create operators, set parameters, wire connections, | `NousResearch__hermes-plugin-touchdesigner` |
| [trace-claude-code](./trace-claude-code) | \| | `Continuous-Claude-v3-main` |
| [trace-mcp](./trace-mcp) | Use trace-mcp tools for code navigation, impact analysis, and framework-aware queries instead of Read/Grep/Glo | `trace-mcp` |
| [trading-agents](./trading-agents) | 个股深度投研分析（智能体自主版）— 拉取 QuantMind 本地数据（371维特征/风险评分/模型推理分数/新闻）→ 多空子代理辩论 → 综合研判 → 生成 md 报告 → 导出 PDF 到平台「股票报告」页。任何大模 | `QuantMind` |
| [transcript-grep](./transcript-grep) | > | `Claude-Code-Agent-Monitor` |
| [transformers](./transformers) | Hugging Face Transformers for loading Hub models, running pipeline inference, text generation, and Trainer fin | `scientific-agent-skills` |
| [transfuser](./transfuser) | Guides TransFuser autonomous-driving model training, multimodal | `AREX-Skill` |
| [trello-automation](./trello-automation) | Automate Trello boards, cards, and workflows via Rube MCP (Composio). Create cards, manage lists, assign membe | `agentic-awesome-skills` |
| [trello-automation-antigravity-awesome-skills-main](./trello-automation-antigravity-awesome-skills-main) | Automate Trello boards, cards, and workflows via Rube MCP (Composio). Create cards, manage lists, assign membe | `antigravity-awesome-skills-main` |
| [triage](./triage) | Move issues and external PRs through a state machine of triage roles — categorise, verify, grill if needed, an | `agentic-awesome-skills` |
| [troubleshooting](./troubleshooting) | Uses Chrome DevTools MCP and documentation to troubleshoot connection and target issues. Trigger this skill wh | `chrome-devtools-mcp` |
| [tutorial-engineer](./tutorial-engineer) | Creates step-by-step tutorials and educational content from code. Transforms complex concepts into progressive | `agentic-awesome-skills` |
| [tutorial-engineer-antigravity-awesome-skills-main](./tutorial-engineer-antigravity-awesome-skills-main) | Creates step-by-step tutorials and educational content from code. Transforms complex concepts into progressive | `antigravity-awesome-skills-main` |
| [twitter-automation](./twitter-automation) | Automate Twitter/X tasks via Rube MCP (Composio): posts, search, users, bookmarks, lists, media. Always search | `agentic-awesome-skills` |
| [twitter-automation-antigravity-awesome-skills-main](./twitter-automation-antigravity-awesome-skills-main) | Automate Twitter/X tasks via Rube MCP (Composio): posts, search, users, bookmarks, lists, media. Always search | `antigravity-awesome-skills-main` |
| [typescript-expert](./typescript-expert) | TypeScript and JavaScript expert with deep knowledge of type-level programming, performance optimization, mono | `agentic-awesome-skills` |
| [typescript-expert-antigravity-awesome-skills-main](./typescript-expert-antigravity-awesome-skills-main) | TypeScript and JavaScript expert with deep knowledge of type-level programming, performance optimization, mono | `antigravity-awesome-skills-main` |
| [typst-author](./typst-author) | Generate idiomatic Typst (.typ) code, edit and troubleshoot Typst documents and projects, and answer Typst syn | `MathModelAgent` |
| [ui-demo](./ui-demo) | Record polished UI demo videos using Playwright. Use when the user asks to create a demo, walkthrough, screen  | `everything-claude-code-main` |
| [ui-preview](./ui-preview) | Capture headless-Chrome screenshots of the tingly-box frontend (running locally in mock mode) so frontend chan | `tingly-box` |
| [ui-skills-root](./ui-skills-root) | Use before UI-related work to select the smallest useful UI Skills context through the ui-skills CLI. | `agentic-awesome-skills` |
| [ultra-rag](./ultra-rag) | Routes UltraRAG pipeline orchestration, MCP server workflows, and | `AREX-Skill` |
| [umap-learn](./umap-learn) | Use UMAP-learn for nonlinear dimensionality reduction, 2D/3D embeddings, clustering preprocessing, supervised  | `scientific-agent-skills` |
| [uncertainty-and-units](./uncertainty-and-units) | Track physical units and propagate measurement uncertainty in scientific calculations using pint and uncertain | `scientific-agent-skills` |
| [unified-ai-gateway](./unified-ai-gateway) | Operate and evaluate Unified AI System through nine governed MCP tools, including provider-free prompt enhance | `agentic-awesome-skills` |
| [unified-notifications-ops](./unified-notifications-ops) | Operate notifications as one ECC-native workflow across GitHub, Linear, desktop alerts, hooks, and connected c | `everything-claude-code-main` |
| [uniprot-database](./uniprot-database) | Direct REST API access to UniProt. Protein searches, FASTA retrieval, ID mapping, Swiss-Prot/TrEMBL. For Pytho | `agentic-awesome-skills` |
| [uniprot-database-antigravity-awesome-skills-main](./uniprot-database-antigravity-awesome-skills-main) | Direct REST API access to UniProt. Protein searches, FASTA retrieval, ID mapping, Swiss-Prot/TrEMBL. For Pytho | `antigravity-awesome-skills-main` |
| [unslop](./unslop) | Post-process AI-generated text through the unslop CLI to strip AI writing patterns before publishing | `agentic-awesome-skills` |
| [unstract](./unstract) | Use Unstract to operate its document-extraction APIs, hosted MCP | `AREX-Skill` |
| [update-swiftui-apis](./update-swiftui-apis) | Scan Apple's SwiftUI documentation for deprecated APIs and update the SwiftUI Expert Skill with modern replace | `agentic-awesome-skills` |
| [upsonic](./upsonic) | Guides Upsonic Python agent-framework workflows, including agents, | `AREX-Skill` |
| [usage-trends](./usage-trends) | > | `Claude-Code-Agent-Monitor` |
| [usfiscaldata](./usfiscaldata) | Query the U.S. Treasury Fiscal Data REST API for federal financial data. No API key required. Use for national | `scientific-agent-skills` |
| [using-agent-skills](./using-agent-skills) | Discover and choose the right Kandev agent skill for a task. Use when starting a session, when the user asks w | `kandev` |
| [using-git-worktrees](./using-git-worktrees) | OPTIONAL per-task isolation for autonomous parallel work. Routed to only on explicit opt-in by subagent-driven | `arbiterForge__codeArbiter` |
| [using-neon](./using-neon) | Neon is a serverless Postgres platform that separates compute and storage to offer autoscaling, branching, ins | `agentic-awesome-skills` |
| [using-neon-antigravity-awesome-skills-main](./using-neon-antigravity-awesome-skills-main) | Neon is a serverless Postgres platform that separates compute and storage to offer autoscaling, branching, ins | `antigravity-awesome-skills-main` |
| [using-superpowers](./using-superpowers) | Use when starting any conversation - establishes how to find and use skills, requiring Skill tool invocation b | `agentic-awesome-skills` |
| [using-superpowers-antigravity-awesome-skills-main](./using-superpowers-antigravity-awesome-skills-main) | Use when starting any conversation - establishes how to find and use skills, requiring Skill tool invocation b | `antigravity-awesome-skills-main` |
| [using-superpowers-superpowers-main](./using-superpowers-superpowers-main) | Use when starting any conversation - establishes how to find and use skills, requiring Skill tool invocation b | `superpowers-main` |
| [ux-design](./ux-design) | Guided, section-by-section UX spec authoring for a screen, flow, or HUD. Reads game concept, player journey, a | `Claude-Code-Game-Studios-main` |
| [ux-review](./ux-review) | Validates a UX spec, HUD design, or interaction pattern library for completeness, accessibility compliance, GD | `Claude-Code-Game-Studios-main` |
| [uxui-principles](./uxui-principles) | Evaluate interfaces against 168 research-backed UX/UI principles, detect antipatterns, and inject UX context i | `agentic-awesome-skills` |
| [uxui-principles-antigravity-awesome-skills-main](./uxui-principles-antigravity-awesome-skills-main) | Evaluate interfaces against 168 research-backed UX/UI principles, detect antipatterns, and inject UX context i | `antigravity-awesome-skills-main` |
| [v3-mcp-optimization](./v3-mcp-optimization) | MCP server optimization and transport layer enhancement for claude-flow v3. Implements connection pooling, loa | `ruflo` |
| [v3-mcp-optimization-ruflo](./v3-mcp-optimization-ruflo) | MCP server optimization and transport layer enhancement for claude-flow v3. Implements connection pooling, loa | `ruflo` |
| [v3-mcp-optimization-ruflo-main](./v3-mcp-optimization-ruflo-main) | MCP server optimization and transport layer enhancement for claude-flow v3. Implements connection pooling, loa | `ruflo-main` |
| [v3-mcp-optimization-ruflo-main](./v3-mcp-optimization-ruflo-main) | MCP server optimization and transport layer enhancement for claude-flow v3. Implements connection pooling, loa | `ruflo-main` |
| [vaex](./vaex) | Use this skill for processing and analyzing large tabular datasets (billions of rows) that exceed available RA | `scientific-agent-skills` |
| [validate-plugin](./validate-plugin) | Validate a Claude Code plugin structure, frontmatter, and MCP tool references | `ruflo` |
| [validate-ui](./validate-ui) | \| | `Archon-dev` |
| [vc-agent-strategy-compare](./vc-agent-strategy-compare) | Evaluate 4 execution strategies (sequential, parallel-subagents, workflow, agent-team) for a phase or fan-out  | `vibecode-pro-max-kit` |
| [vc-audit-context](./vc-audit-context) | Audit project context routing, shared-skill discoverability, and Claude/Codex wiring. Use when context docs or | `vibecode-pro-max-kit` |
| [vc-audit-vc](./vc-audit-vc) | >- | `vibecode-pro-max-kit` |
| [vc-intent-clarify](./vc-intent-clarify) | Clarify intent before RIPER-5 phase delegation. Scores ambiguity (4 signals); generates structured multi-choic | `vibecode-pro-max-kit` |
| [vc-scenario](./vc-scenario) | Generate comprehensive edge cases and test scenarios by decomposing features across 12 dimensions. Use before  | `vibecode-pro-max-kit` |
| [vc-sense](./vc-sense) | Use when the user explicitly asks to apply Vibe Code Common Sense, create project rails, scaffold AI-agent pro | `vibe-code-common-sense` |
| [vector-setup](./vector-setup) | First-run setup for ruvector@0.2.25 — installs ONNX/Brain/SONA add-ons, registers the MCP server, and verifies | `ruflo` |
| [venue-templates](./venue-templates) | Prepare journal manuscripts, conference papers, research posters, and grant documents using venue-specific for | `scientific-agent-skills` |
| [vercel-automation](./vercel-automation) | Automate Vercel tasks via Rube MCP (Composio): manage deployments, domains, DNS, env vars, projects, and teams | `agentic-awesome-skills` |
| [vercel-automation-antigravity-awesome-skills-main](./vercel-automation-antigravity-awesome-skills-main) | Automate Vercel tasks via Rube MCP (Composio): manage deployments, domains, DNS, env vars, projects, and teams | `antigravity-awesome-skills-main` |
| [vercel-cli-with-tokens](./vercel-cli-with-tokens) | Deploy and manage projects on Vercel using token-based authentication. Use when working with Vercel CLI using  | `agentic-awesome-skills` |
| [vercel-optimize](./vercel-optimize) | Audit deployed Vercel apps for cost and performance issues using metrics, project config, code scans, and vers | `agentic-awesome-skills` |
| [vercel-react-view-transitions](./vercel-react-view-transitions) | Guide React and Next.js view transitions, shared element animations, route transitions, transition types, and  | `agentic-awesome-skills` |
| [verification-agent](./verification-agent) | Provides a structured verification workflow for code and prompt outputs. Use when asked to validate correctnes | `claude-code-prompts-master` |
| [verification-loop](./verification-loop) | A comprehensive verification system for Claude Code sessions. Use when verifying a Claude Code session's work  | `ECC` |
| [verification-loop-ECC](./verification-loop-ECC) | A comprehensive verification system for Claude Code sessions. Use when verifying a Claude Code session's work  | `ECC` |
| [verification-loop-everything-claude-code-main](./verification-loop-everything-claude-code-main) | A comprehensive verification system for Claude Code sessions. | `everything-claude-code-main` |
| [verify](./verify) | Verify harness changes end-to-end without docker — drive the real pinned CLI against a header-capturing stub s | `anthropics__defending-code-reference-harness` |
| [verify-citations](./verify-citations) | Verify citations and references in a document, report, or article against real sources. Use when the user asks | `agentic-awesome-skills` |
| [verify-claims](./verify-claims) | Audit a report, plan, or handoff by re-deriving every load-bearing claim from primary sources. Use before trus | `happier` |
| [version-release](./version-release) | Choose and apply the correct semantic version bump for this repository. Use for every user-visible release, be | `Claude-Code-Agent-Monitor` |
| [version-release-Claude-Code-Agent-Monitor](./version-release-Claude-Code-Agent-Monitor) | Choose and apply the correct semantic version bump for this repository. Use for every user-visible release, be | `Claude-Code-Agent-Monitor` |
| [vexor](./vexor) | Vector-powered CLI for semantic file search with a Claude/Codex skill | `agentic-awesome-skills` |
| [vexor-antigravity-awesome-skills-main](./vexor-antigravity-awesome-skills-main) | Vector-powered CLI for semantic file search with a Claude/Codex skill | `antigravity-awesome-skills-main` |
| [vibe-delegate](./vibe-delegate) | Delegate coding tasks to the Mistral Vibe CLI (`vibe`) only when the | `agentic-awesome-skills` |
| [vibe-to-agentic-framework](./vibe-to-agentic-framework) | The conceptual framework behind the presentation — what "Vibe Coding to Agentic Engineering" means, why the jo | `claude-code-best-practice` |
| [vibe-to-agentic-framework-claude-code-best-practice-main](./vibe-to-agentic-framework-claude-code-best-practice-main) | The conceptual framework behind the presentation — what "Vibe Coding to Agentic Engineering" means, why the jo | `claude-code-best-practice-main` |
| [video-content-extractor](./video-content-extractor) | Extract key frames from MP4 videos at configurable intervals, run Tesseract OCR, and generate structured Markd | `agentic-awesome-skills` |
| [video-editing](./video-editing) | AI-assisted video editing workflows for cutting, structuring, and augmenting real footage. Covers the full pip | `everything-claude-code-main` |
| [video-editing-everything-claude-code-main](./video-editing-everything-claude-code-main) | AI-assisted video editing workflows for cutting, structuring, and augmenting real footage. Covers the full pip | `everything-claude-code-main` |
| [video-perception](./video-perception) | Use when the user mentions a video file (.mp4, .mov, .avi, .mkv, .webm), a YouTube URL, asks to watch/analyze/ | `claude-video-vision` |
| [video-talkcraft](./video-talkcraft) | 终极口播视频 skill：中文口播稿 + 成品配音 → CPU 字级时间戳 → SHOTBOOK 层矩阵分镜 → Remotion 电影感成片（横屏默认/竖屏）。当用户要"做口播视频"、"解说/科普视频"、"把文案变成视 | `video-talkcraft` |
| [videodb](./videodb) | See, Understand, Act on video and audio. See- ingest from local files, URLs, RTSP/live feeds, or live record d | `everything-claude-code-main` |
| [visa-doc-translate](./visa-doc-translate) | Translate visa application documents (images) to English and create a bilingual PDF with original and translat | `everything-claude-code-main` |
| [vitest-skill](./vitest-skill) | Generates Vitest tests in JavaScript/TypeScript with Vite-native speed. Jest-compatible API with ESM support a | `agentic-awesome-skills` |
| [voice-ai-development](./voice-ai-development) | Expert in building voice AI applications - from real-time voice | `agentic-awesome-skills` |
| [voice-ai-development-antigravity-awesome-skills-main](./voice-ai-development-antigravity-awesome-skills-main) | Expert in building voice AI applications - from real-time voice | `antigravity-awesome-skills-main` |
| [vscode-extension-guide-en](./vscode-extension-guide-en) | Guide for VS Code extension development from scaffolding to Marketplace publication | `agentic-awesome-skills` |
| [vss-query-analytics](./vss-query-analytics) | Use this skill when reading video-analytics metrics, incidents, alerts, and sensor data via VA-MCP (Docker :99 | `video-search-and-summarization` |
| [warp-delegate](./warp-delegate) | Delegate coding tasks to the Warp Agent CLI (`oz`) only when the user | `agentic-awesome-skills` |
| [wasm-agent](./wasm-agent) | Create and manage sandboxed WASM agents for isolated code execution | `ruflo` |
| [wasm-gallery](./wasm-gallery) | Browse, publish, and install WASM agents from the community gallery | `ruflo` |
| [waypoint-bio](./waypoint-bio) | Use when working with Outpost Bio's open microbiome foundation models - the Waypoint checkpoints (Waypoint-6m, | `scientific-agent-skills` |
| [weather-fetcher](./weather-fetcher) | Instructions for fetching current weather temperature data for Dubai, UAE from Open-Meteo API | `claude-code-best-practice` |
| [weather-fetcher-claude-code-best-practice-main](./weather-fetcher-claude-code-best-practice-main) | Instructions for fetching current weather temperature data for Dubai, UAE from Open-Meteo API | `claude-code-best-practice-main` |
| [weather-svg-creator](./weather-svg-creator) | Creates an SVG weather card showing the current temperature for Dubai. Writes the SVG to orchestration-workflo | `claude-code-best-practice` |
| [weather-svg-creator-claude-code-best-practice-main](./weather-svg-creator-claude-code-best-practice-main) | Creates an SVG weather card showing the current temperature for Dubai. Writes the SVG to orchestration-workflo | `claude-code-best-practice-main` |
| [weaviate](./weaviate) | Search, query, inspect, create, and import data into Weaviate vector database collections using official scrip | `agentic-awesome-skills` |
| [weaviate-cookbooks](./weaviate-cookbooks) | Build Weaviate AI apps from official cookbook blueprints for RAG, agentic RAG, data exploration, multimodal PD | `agentic-awesome-skills` |
| [web-design-guidelines](./web-design-guidelines) | 前端 UI 审查清单：可读性、视觉密度、间距、响应式、空状态、加载态和可访问性。 | `OpenBMB__PilotDeck` |
| [web-media-getter](./web-media-getter) | One query across free image / video / GIF APIs (stock + historical/archival + GIF engines), returning normaliz | `agentic-awesome-skills` |
| [web-scraper](./web-scraper) | Web scraping inteligente multi-estrategia. Extrai dados estruturados de paginas web (tabelas, listas, precos). | `agentic-awesome-skills` |
| [webdriverio-skill](./webdriverio-skill) | Generates WebdriverIO (WDIO) automation tests in JavaScript or TypeScript. Supports local and TestMu AI cloud. | `agentic-awesome-skills` |
| [webflow-automation](./webflow-automation) | Automate Webflow CMS collections, site publishing, page management, asset uploads, and ecommerce orders via Ru | `agentic-awesome-skills` |
| [webflow-automation-antigravity-awesome-skills-main](./webflow-automation-antigravity-awesome-skills-main) | Automate Webflow CMS collections, site publishing, page management, asset uploads, and ecommerce orders via Ru | `antigravity-awesome-skills-main` |
| [webhook-management](./webhook-management) | > | `Claude-Code-Agent-Monitor` |
| [weekly-report](./weekly-report) | > | `Claude-Code-Agent-Monitor` |
| [weekly-review-planning](./weekly-review-planning) | Weekly reset: commitments, stalled work, next-week plan. | `hermes-agent` |
| [wgm](./wgm) | Turns a rough request into working software via a governed build loop: align first, plan, then iterate one tas | `agentic-awesome-skills` |
| [what-if-oracle](./what-if-oracle) | Run structured What-If scenario analysis with 4–6 branch possibility exploration (best, likely, worst, wild ca | `scientific-agent-skills` |
| [whatsapp-automation](./whatsapp-automation) | Automate WhatsApp Business tasks via Rube MCP (Composio): send messages, manage templates, upload media, and h | `agentic-awesome-skills` |
| [whatsapp-automation-antigravity-awesome-skills-main](./whatsapp-automation-antigravity-awesome-skills-main) | Automate WhatsApp Business tasks via Rube MCP (Composio): send messages, manage templates, upload media, and h | `antigravity-awesome-skills-main` |
| [whatsapp-cloud-api](./whatsapp-cloud-api) | Integracao com WhatsApp Business Cloud API (Meta). Mensagens, templates, webhooks HMAC-SHA256, automacao de at | `agentic-awesome-skills` |
| [whatsapp-cloud-api-antigravity-awesome-skills-main](./whatsapp-cloud-api-antigravity-awesome-skills-main) | Integracao com WhatsApp Business Cloud API (Meta). Mensagens, templates, webhooks HMAC-SHA256, automacao de at | `antigravity-awesome-skills-main` |
| [wigolo](./wigolo) | Local-first web intelligence MCP server for AI coding agents. Ten tools for search, fetch, crawl, cache, extra | `wigolo` |
| [wiki-builder](./wiki-builder) | Create and maintain reusable research wikis with source provenance, configurable structure, and local markdown | `agentic-awesome-skills` |
| [wiki-viewer](./wiki-viewer) | Render a self-contained HTML viewer for a pro-workflow wiki. Pages, sources, claims, seed queue, page-link gra | `pro-workflow` |
| [winnow](./winnow) | How to read winnow stubs in tool results. A block marked "[winnow] ... hidden" was judged unlikely to matter f | `GhalebDweikat__winnow` |
| [workflow-create](./workflow-create) | Author a workflow — either an MCP workflow template (persisted, lifecycle) or a native .claude/workflows/*.js  | `ruflo` |
| [workflow-optimizer](./workflow-optimizer) | > | `Claude-Code-Agent-Monitor` |
| [workflow-report](./workflow-report) | > | `Claude-Code-Agent-Monitor` |
| [workflows](./workflows) | Author or run durable BB workflows when the user requests workflow execution or multi-agent orchestration. | `bb` |
| [workflows-openpi](./workflows-openpi) | Orchestrates multi-agent work with OpenPI's inline JavaScript Workflow DSL. Use when a task needs multi-phase  | `openpi` |
| [workspace-surface-audit](./workspace-surface-audit) | Audit the active repo, MCP servers, plugins, connectors, env surfaces, and harness setup, then recommend the h | `ECC` |
| [workspace-surface-audit-everything-claude-code-main](./workspace-surface-audit-everything-claude-code-main) | Audit the active repo, MCP servers, plugins, connectors, env surfaces, and harness setup, then recommend the h | `everything-claude-code-main` |
| [worktrunk](./worktrunk) | Guidance for Worktrunk (the `wt` CLI) — git worktree management, hooks, and config. Load when working out whic | `max-sixty__worktrunk` |
| [worktrunk-worktrunk](./worktrunk-worktrunk) | Guidance for Worktrunk (the `wt` CLI) — git worktree management, hooks, and config. Load when working out whic | `worktrunk` |
| [wrike-automation](./wrike-automation) | Automate Wrike project management via Rube MCP (Composio): create tasks/folders, manage projects, assign work, | `agentic-awesome-skills` |
| [wrike-automation-antigravity-awesome-skills-main](./wrike-automation-antigravity-awesome-skills-main) | Automate Wrike project management via Rube MCP (Composio): create tasks/folders, manage projects, assign work, | `antigravity-awesome-skills-main` |
| [write-connector](./write-connector) | Add a new built-in OpenWiki source connector. Use when a user asks to create or implement an OpenWiki connecto | `openwiki` |
| [write-skills](./write-skills) | Create or revise agent skills. Use when adding a new skill file, renaming a skill, simplifying an existing ski | `skills` |
| [writing-great-skills](./writing-great-skills) | Reference for writing and editing skills well — the vocabulary and principles that make a skill predictable. | `agentic-awesome-skills` |
| [writing-plans](./writing-plans) | Use when you have a spec or requirements for a multi-step task. Creates comprehensive implementation plans wit | `hermes-agent-main` |
| [writing-skills](./writing-skills) | Use when creating, updating, or improving agent skills. | `agentic-awesome-skills` |
| [writing-skills-antigravity-awesome-skills-main](./writing-skills-antigravity-awesome-skills-main) | Use when creating, updating, or improving agent skills. | `antigravity-awesome-skills-main` |
| [writing-skills-superpowers-main](./writing-skills-superpowers-main) | Use when creating new skills, editing existing skills, or verifying skills work before deployment | `superpowers-main` |
| [x-api](./x-api) | X/Twitter API integration for posting tweets, threads, reading timelines, search, and analytics. Covers OAuth  | `ECC` |
| [x-api-ECC](./x-api-ECC) | X/Twitter API integration for posting tweets, threads, reading timelines, search, and analytics. Covers OAuth  | `ECC` |
| [x-api-everything-claude-code-main](./x-api-everything-claude-code-main) | X/Twitter API integration for posting tweets, threads, reading timelines, search, and analytics. Covers OAuth  | `everything-claude-code-main` |
| [x402](./x402) | Set up Browser Use Cloud payments with x402 — pay per request from a crypto wallet (USDC on Base mainnet), no  | `browser-use` |
| [xlsx](./xlsx) | Create, read, edit Excel .xlsx workbooks and CSVs. | `hermes-agent` |
| [xurl](./xurl) | A CLI tool for making authenticated requests to the X (Twitter) API. Use this skill when you need to post twee | `understudy-ai__understudy` |
| [xvary-stock-research](./xvary-stock-research) | Thesis-driven equity analysis from public SEC EDGAR and market data; /analyze, /score, /compare workflows with | `agentic-awesome-skills` |
| [xvary-stock-research-antigravity-awesome-skills-main](./xvary-stock-research-antigravity-awesome-skills-main) | Thesis-driven equity analysis from public SEC EDGAR and market data; /analyze, /score, /compare workflows with | `antigravity-awesome-skills-main` |
| [yann-lecun](./yann-lecun) | Agente que simula Yann LeCun — inventor das Convolutional Neural Networks, Chief AI Scientist da Meta, Prêmio  | `agentic-awesome-skills` |
| [yann-lecun-antigravity-awesome-skills-main](./yann-lecun-antigravity-awesome-skills-main) | Agente que simula Yann LeCun — inventor das Convolutional Neural Networks, Chief AI Scientist da Meta, Prêmio  | `antigravity-awesome-skills-main` |
| [yann-lecun-debate](./yann-lecun-debate) | Sub-skill de debates e posições de Yann LeCun. Cobre críticas técnicas detalhadas aos LLMs, rivalidades intele | `agentic-awesome-skills` |
| [yann-lecun-debate-antigravity-awesome-skills-main](./yann-lecun-debate-antigravity-awesome-skills-main) | Sub-skill de debates e posições de Yann LeCun. Cobre críticas técnicas detalhadas aos LLMs, rivalidades intele | `antigravity-awesome-skills-main` |
| [yann-lecun-tecnico](./yann-lecun-tecnico) | Sub-skill técnica de Yann LeCun. Cobre CNNs, LeNet, backpropagation, JEPA (I-JEPA, V-JEPA, MC-JEPA), AMI (Adva | `agentic-awesome-skills` |
| [yann-lecun-tecnico-antigravity-awesome-skills-main](./yann-lecun-tecnico-antigravity-awesome-skills-main) | Sub-skill técnica de Yann LeCun. Cobre CNNs, LeNet, backpropagation, JEPA (I-JEPA, V-JEPA, MC-JEPA), AMI (Adva | `antigravity-awesome-skills-main` |
| [yao-meta-skill](./yao-meta-skill) | Create, refactor, evaluate, and package agent skills from workflows, prompts, transcripts, docs, or notes. Use | `agentic-awesome-skills` |
| [yield-intelligence](./yield-intelligence) | Passive income portfolio analysis — activate when user asks about dividend yields, Treasury rates, REIT income | `agentic-awesome-skills` |
| [youtube-automation](./youtube-automation) | Automate YouTube tasks via Rube MCP (Composio): upload videos, manage playlists, search content, get analytics | `agentic-awesome-skills` |
| [youtube-automation-antigravity-awesome-skills-main](./youtube-automation-antigravity-awesome-skills-main) | Automate YouTube tasks via Rube MCP (Composio): upload videos, manage playlists, search content, get analytics | `antigravity-awesome-skills-main` |
| [youtube-content](./youtube-content) | YouTube transcripts to summaries, threads, blogs. | `hermes-agent` |
| [youtube-notetaker](./youtube-notetaker) | Turn YouTube talks into local study notes with slides, transcripts, editable annotations, and a markdown-backe | `agentic-awesome-skills` |
| [zarr-python](./zarr-python) | Chunked N-D arrays for cloud storage (Zarr-Python 3). Compressed arrays, parallel I/O, S3/GCS via fsspec, NumP | `scientific-agent-skills` |
| [zcode-delegate](./zcode-delegate) | Delegate coding tasks to the Z.AI ZCode CLI only when the user explicitly | `agentic-awesome-skills` |
| [zendesk-automation](./zendesk-automation) | Automate Zendesk tasks via Rube MCP (Composio): tickets, users, organizations, replies. Always search tools fi | `agentic-awesome-skills` |
| [zendesk-automation-antigravity-awesome-skills-main](./zendesk-automation-antigravity-awesome-skills-main) | Automate Zendesk tasks via Rube MCP (Composio): tickets, users, organizations, replies. Always search tools fi | `antigravity-awesome-skills-main` |
| [zoho-cliq](./zoho-cliq) | \| | `application-skills` |
| [zoho-crm-automation](./zoho-crm-automation) | Automate Zoho CRM tasks via Rube MCP (Composio): create/update records, search contacts, manage leads, and con | `agentic-awesome-skills` |
| [zoho-crm-automation-antigravity-awesome-skills-main](./zoho-crm-automation-antigravity-awesome-skills-main) | Automate Zoho CRM tasks via Rube MCP (Composio): create/update records, search contacts, manage leads, and con | `antigravity-awesome-skills-main` |
| [zoom-automation](./zoom-automation) | Automate Zoom meeting creation, management, recordings, webinars, and participant tracking via Rube MCP (Compo | `agentic-awesome-skills` |
| [zoom-automation-antigravity-awesome-skills-main](./zoom-automation-antigravity-awesome-skills-main) | Automate Zoom meeting creation, management, recordings, webinars, and participant tracking via Rube MCP (Compo | `antigravity-awesome-skills-main` |
