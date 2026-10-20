# 🐘 pink-elephant

**Audit and eliminate the "Pink Elephant Effect" in AI agent instructions, prompts, schemas, and codebases.**  
**审查并清理代码仓库、Prompt、Tool Schema 与 Agent 规则中的“粉红大象效应”（反向声明、负面禁令与墓碑注释）。**

---

## Why `pink-elephant`? / 为什么需要它？

When an instruction says *"Do NOT use `LegacyClient`"* or *"Never pass `user_id` when querying by department"*, the forbidden token enters the LLM's attention window and activates related weights—making the model **more likely** to hallucinate or invoke the exact thing you prohibited.

当你在 `CLAUDE.md`、`AGENTS.md`、Prompt 或代码注释里写下“不要调用 `v1_parse`”、“已移除 XML 解析，请勿使用”时，被禁止的符号与概念会直接进入模型的注意力窗口并激活相关权重，反而诱发错误调用或模仿。

**Core Principle / 核心准则：**
> **Absence is negation; affirmative is the standard.**  
> **缺席即否定，正面即标准。**

1. **Deprecated concepts & legacy ghosts → Physical Deletion (`物理删除`)**  
   Verify zero production consumers via `rg`, then delete the warning paragraph entirely.
2. **Tombstone comments & changelog narratives → Physical Deletion (`物理删除`)**  
   Remove `# removed retry logic` or `# previously used regex here` from code and docs.
3. **Valid behavioral prohibitions → Affirmative Rewrite (`正向改写 / 去象化`)**  
   Rewrite *"Do not use `requests`"* into *"Use the shared `httpx.AsyncClient` for all HTTP calls"*—stripping the forbidden name completely.
4. **Tool / Schema negative traps → Structural Narrowing + Affirmative Description (`结构收敛 + 正向描述`)**  
   Replace *"Do NOT pass `user_id` when..."* with positive trigger conditions, or split/narrow the schema type.

---

## Installation / 安装方式

### Via `skills` CLI (Claude Code / Codex / Cursor)

```bash
npx skills add ox01024/pink-elephant
```

### Via Git Clone

```bash
# Claude Code
git clone https://github.com/ox01024/pink-elephant.git ~/.claude/skills/pink-elephant

# OpenAI Codex
git clone https://github.com/ox01024/pink-elephant.git ~/.codex/skills/pink-elephant
```

---

## Usage / 使用方法

### 1. Audit Mode / 审计模式（只读扫描，输出分级方案）

```text
/pink-elephant 审查当前仓库
```
or
```text
Use $pink-elephant to audit CLAUDE.md and AGENTS.md
```

Outputs a structured report grouped by surface (`Agent Instructions` → `Prompts & Tool Schemas` → `Docs` → `Code Comments`):

```text
[物理删除 | 正向改写 | 结构收敛] 文件路径:行号
原句：<原始负面表述或墓碑注释>
诊断：<激活了哪个粉红大象词汇 / 属于哪项已废弃概念>
方案：<直接物理删除第 X-Y 行 | 改写后的完整正面语句>
```

### 2. Execute Mode / 执行模式（落地清理并验证）

```text
/pink-elephant 清理当前仓库中的粉红大象
```

1. Checks `git status` to preserve unrelated work.
2. Verifies zero production references before deleting deprecated concepts.
3. Rewrites negative rules into clean affirmative standards and deletes tombstone comments.
4. Re-runs `rg` and project tests to confirm zero regressions.

---

## License

[MIT](./LICENSE)
