# 🐘 pink-elephant

**Audit, eliminate, and guard against the "Pink Elephant Effect" in AI agent instructions (`CLAUDE.md` / `AGENTS.md`), prompts, tool schemas, and codebases.**  
**审查、清理并预防代码仓库、Prompt、Tool Schema 与 Agent 规则中的“粉红大象效应”（反向声明、对话迭代残留、负面禁令与墓碑注释）。**

---

## 什么是代码仓库里的“粉红大象效应”？

心理学中，当你听到“**不要去想一头粉红大象**”时，脑海里立刻会浮现出一头粉红大象。  
在 LLM 与 Agent 工程中，这种效应会在两个场景下严重污染模型注意力：

### 1. 迭代残留带入交付物（“番茄炒蛋没有东坡肉”）

Agent 在多轮对话迭代中修正了方案（例如先讨论了东坡肉，后决定只做番茄炒蛋），却把**被否决的内容**带进最终标题、注释、Commit、PR 或交付说明：

```diff
- 标题：番茄炒蛋（没有东坡肉）
+ 标题：番茄炒蛋
```

### 2. 负面禁令与墓碑警告激活错误权重

当你在 `CLAUDE.md`、`AGENTS.md`、Prompt 或 Tool Schema 里写下“**不要调用 `v1_parse`**”、“**查询部门时千万不要传 `user_id`**”时，被禁止的符号与参数名会直接进入模型的注意力窗口并激活相关权重——反而诱发模型高频误调你禁止的接口或参数。

**核心准则：**
> **缺席即否定，正面即标准。**  
> **Absence is negation; affirmative is the standard.**

---

## 典型案例对照 / Typical Examples

### 案例 1：消除对话迭代残留（Commit / PR / 文档标题）

```diff
- feat(auth): 改用 JWT 鉴权（移除原讨论的 Session Redis 方案，不含图形验证码）
+ feat(auth): 使用无状态 JWT 完成接口鉴权
```

```diff
- // 解析请求载荷（注意：这里不再做签名校验，签名校验已移交网关层）
+ // 解析已通过网关验签的请求载荷
  func parsePayload(raw []byte) (*Payload, error)
```

### 案例 2：清理 Agent 指令（`CLAUDE.md` / `AGENTS.md`）中的负面禁令

将“不要做 X”改写为唯一可执行的正面标准，并在改写后的句子中**彻底抹去被禁事物的名字**：

```diff
- 不要使用 requests 或 urllib 发请求，避免阻塞事件循环，也不要在每个函数里新建 client。
+ HTTP 调用统一使用共享的 `httpx.AsyncClient` 实例。
```

```diff
- 处理 GitLab 时永远不要使用代理；禁止在日志或回复中原样回显 API Key 和 Token，也不要把密钥写进代码。
+ GitLab（内网服务）使用本地已认证 `glab` 并清空代理环境变量；凭据通过环境变量或 `.env` 注入，在日志与回复中统一脱敏。
```

### 案例 3：物理删除已废弃概念警告与墓碑注释

一旦通过 `rg` 确认生产代码已无承重调用，直接物理删除整段警告或墓碑注释，而非保留“已废弃/已移除”标记：

```diff
- # 注意：v1_parse_email 已经废弃下线，请勿在任何新代码中调用它，一律改用 v2。
- # 2025-12 移除了旧的正则回溯逻辑，千万不要改回正则！
  def parse_email(raw: bytes) -> EmailMessage:
      return v2_parse_email(raw)
```

### 案例 4：收敛 Tool / Schema 参数反向诱导

```diff
  class UserQueryInput(BaseModel):
      department_id: Optional[str] = Field(None, description="按部门 ID 查询成员列表")
      user_id: Optional[str] = Field(
          None,
-         description="用户 ID。注意：当按 department_id 查询时，绝对不要传 user_id，也不要传空字符串！"
+         description="用户 ID。仅在按单个成员精确查询时填写；按部门查询时省略该字段。"
      )
```

---

## 安装方式 / Installation

### 使用 `skills` CLI 安装

```bash
npx skills add ox01024/pink-elephant
```

### 使用 Git Clone 安装

```bash
# Claude Code
git clone https://github.com/ox01024/pink-elephant.git ~/.claude/skills/pink-elephant

# OpenAI Codex
git clone https://github.com/ox01024/pink-elephant.git ~/.codex/skills/pink-elephant
```

---

## 使用方法 / Usage

### 1. 审查模式（Audit Mode）—— 只读扫描并输出重写方案

审查当前项目的 `CLAUDE.md` 和 `AGENTS.md`：

```text
用 pink-elephant 审查一下当前的 CLAUDE.md 和 AGENTS.md
```

或者审查整个代码仓库（覆盖指令文件、Prompt、Tool Schema、文档与代码注释）：

```text
/pink-elephant 审查当前仓库
```

输出格式：

```text
[物理删除 | 正向改写 | 结构收敛] AGENTS.md:34
原句：不要实现用户没有要求的功能，不要为一次性代码创建抽象。
诊断：连续使用“不要”将“未要求功能”、“一次性抽象”放入注意力窗口。
方案：仅实现用户明确要求的功能与配置项，一次性逻辑直接内联编写。
```

### 2. 清理模式（Clean Mode）—— 落地物理删除与正面重写

```text
用 pink-elephant 清理当前的 CLAUDE.md 和 AGENTS.md
```

```text
/pink-elephant 清理当前仓库中的粉红大象
```

1. 先检查 `git status` 保护无关改动。
2. 对拟物理删除的废弃概念，先用 `rg` 核实全仓生产代码零依赖。
3. 执行物理删除与去象化正面改写。
4. 再次运行 `rg` 与相关测试，确认负面词汇与废弃符号清零且无回归。

### 3. 护栏注入模式（Guard Mode）—— 在 `CLAUDE.md` 和 `AGENTS.md` 中增加防粉红大象约束

想从源头防止 Agent 在日常开发中写出“番茄炒蛋（没有东坡肉）”式的注释、Commit、PR 或反向声明？直接让 skill 为你的规则文件注入防象护栏：

```text
用 pink-elephant 给 CLAUDE.md 和 AGENTS.md 增加防粉红大象约束
```

它会在你的 `CLAUDE.md` 和 `AGENTS.md` 中合入一段**自身完全正面表述**的纯净性契约：

```markdown
### 产物纯净性与去粉红大象（Pink Elephant Guard）

- **只表达最终状态**：代码、注释、Commit、MR/PR 标题与描述、文档及 Prompt 只描述当前生效的设计规格与行为，如同最初即如此设计。
- **缺席即否定**：当去掉某功能、排除某行为或在迭代中否决某方案时，直接从代码、文案、注释、Commit/MR 与 Prompt 中移除对应内容，仅保留最终采纳方案（例如直接写 `标题：番茄炒蛋`，而非 `标题：番茄炒蛋（没有东坡肉）`；仅在重大破坏性迁移或用户明确要求时记录决策说明）。
- **正面即标准**：编写规则、Prompt、Tool Schema 与注释时，直接声明期望的唯一正面做法与触发条件；已废弃的工具与旧概念直接物理删除。
```

---

## License

[MIT](./LICENSE)
