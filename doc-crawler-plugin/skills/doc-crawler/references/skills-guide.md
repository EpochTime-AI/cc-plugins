# Agent Skills

> 在 Claude Code 中创建、管理和共享 Skills 以扩展 Claude 的功能。

本指南展示了如何在 Claude Code 中创建、使用和管理 Agent Skills。有关 Skills 如何在 Claude 产品中工作的背景信息，请参阅[什么是 Skills？](https://platform.claude.com/docs/zh-CN/agents-and-tools/agent-skills/overview)。

Skill 是一个 markdown 文件，它教 Claude 如何做特定的事情：使用你的团队标准审查 PR、以你喜欢的格式生成提交消息，或查询你公司的数据库架构。当你要求 Claude 做与 Skill 目的相匹配的事情时，Claude 会自动应用它。

## 创建你的第一个 Skill

这个例子创建了一个个人 Skill，教 Claude 使用可视化图表和类比来解释代码。与 Claude 的默认解释不同，这个 Skill 确保每个解释都包含一个 ASCII 图表和一个现实世界的类比。

<Steps>
  <Step title="检查可用的 Skills">
    在创建 Skill 之前，查看 Claude 已经可以访问的 Skills：

    ```
    What Skills are available?
    ```

    Claude 将列出当前加载的任何 Skills。你可能看不到任何，或者你可能看到来自插件或你的组织的 Skills。
  </Step>

  <Step title="创建 Skill 目录">
    在你的个人 Skills 文件夹中为 Skill 创建一个目录。个人 Skills 在你的所有项目中都可用。（你也可以在 `.claude/skills/` 中创建[项目 Skills](#where-skills-live) 以与你的团队共享。）

    ```bash  theme={null}
    mkdir -p ~/.claude/skills/explaining-code
    ```
  </Step>

  <Step title="编写 SKILL.md">
    每个 Skill 都需要一个 `SKILL.md` 文件。该文件以 `---` 标记之间的 YAML 元数据开始，必须包含 `name` 和 `description`，然后是 Claude 在 Skill 活跃时遵循的 Markdown 说明。

    `description` 特别重要，因为 Claude 使用它来决定何时应用 Skill。

    创建 `~/.claude/skills/explaining-code/SKILL.md`：

    ```yaml  theme={null}
    ---
    name: explaining-code
    description: Explains code with visual diagrams and analogies. Use when explaining how code works, teaching about a codebase, or when the user asks "how does this work?"
    ---

    When explaining code, always include:

    1. **Start with an analogy**: Compare the code to something from everyday life
    2. **Draw a diagram**: Use ASCII art to show the flow, structure, or relationships
    3. **Walk through the code**: Explain step-by-step what happens
    4. **Highlight a gotcha**: What's a common mistake or misconception?

    Keep explanations conversational. For complex concepts, use multiple analogies.
    ```
  </Step>

  <Step title="加载并验证 Skill">
    Skills 在创建或修改时会自动加载。验证 Skill 出现在列表中：

    ```
    What Skills are available?
    ```

    你应该在列表中看到 `explaining-code` 及其描述。
  </Step>

  <Step title="测试 Skill">
    打开你项目中的任何文件，并向 Claude 提出与 Skill 描述相匹配的问题：

    ```
    How does this code work?
    ```

    Claude 应该要求使用 `explaining-code` Skill，然后在其解释中包含一个类比和 ASCII 图表。如果 Skill 没有触发，请尝试重新表述以包含描述中的更多关键词，例如"explain how this works"。
  </Step>
</Steps>

本指南的其余部分涵盖了 Skills 的工作原理、配置选项和故障排除。

## Skills 如何工作

Skills 是**模型调用的**：Claude 根据你的请求决定使用哪些 Skills。你不需要显式调用 Skill。当你的请求与其描述相匹配时，Claude 会自动应用相关的 Skills。

当你发送请求时，Claude 遵循以下步骤来查找和使用相关的 Skills：

<Steps>
  <Step title="发现">
    在启动时，Claude 仅加载每个可用 Skill 的名称和描述。这保持启动速度快，同时给 Claude 足够的上下文来知道何时每个 Skill 可能相关。
  </Step>

  <Step title="激活">
    当你的请求与 Skill 的描述相匹配时，Claude 要求使用 Skill。在完整的 `SKILL.md` 加载到上下文之前，你会看到一个确认提示。由于 Claude 读取这些描述来查找相关的 Skills，[编写描述](#skill-not-triggering)时应包含用户会自然说出的关键词。
  </Step>

  <Step title="执行">
    Claude 遵循 Skill 的说明，根据需要加载引用的文件或运行捆绑的脚本。
  </Step>
</Steps>

### Skills 存放在哪里

你存储 Skill 的位置决定了谁可以使用它：

| 位置 | 路径                                    | 适用于          |
| :- | :------------------------------------ | :----------- |
| 企业 | 参见[托管设置](/zh-CN/iam#managed-settings) | 你的组织中的所有用户   |
| 个人 | `~/.claude/skills/`                   | 你，跨所有项目      |
| 项目 | `.claude/skills/`                     | 在此存储库中工作的任何人 |
| 插件 | 与[插件](/zh-CN/plugins)捆绑               | 安装了该插件的任何人   |

如果两个 Skills 有相同的名称，较高的行获胜：托管覆盖个人，个人覆盖项目，项目覆盖插件。

### 何时使用 Skills 与其他选项

Claude Code 提供了多种自定义行为的方式。关键区别：**Skills 由 Claude 根据你的请求自动触发**，而斜杠命令要求你显式输入 `/command`。

| 使用这个                              | 当你想要...                            | 何时运行                 |
| :-------------------------------- | :--------------------------------- | :------------------- |
| **Skills**                        | 给 Claude 专业知识（例如，"使用我们的标准审查 PR"）   | Claude 在相关时选择        |
| **[斜杠命令](/zh-CN/slash-commands)** | 创建可重用的提示（例如，`/deploy staging`）     | 你输入 `/command` 来运行它  |
| **[CLAUDE.md](/zh-CN/memory)**    | 设置项目范围的说明（例如，"使用 TypeScript 严格模式"） | 加载到每个对话中             |
| **[子代理](/zh-CN/sub-agents)**      | 将任务委托给具有自己工具的单独上下文                 | Claude 委托，或你显式调用     |
| **[Hooks](/zh-CN/hooks)**         | 在事件上运行脚本（例如，在文件保存时 lint）           | 在特定工具事件上触发           |
| **[MCP 服务器](/zh-CN/mcp)**         | 将 Claude 连接到外部工具和数据源               | Claude 根据需要调用 MCP 工具 |

**Skills 与子代理**：Skills 向当前对话添加知识。子代理在具有自己工具的单独上下文中运行。使用 Skills 获得指导和标准；当你需要隔离或不同的工具访问时使用子代理。

**Skills 与 MCP**：Skills 告诉 Claude *如何*使用工具；MCP *提供*工具。例如，MCP 服务器将 Claude 连接到你的数据库，而 Skill 教 Claude 你的数据模型和查询模式。

<Note>
  有关 Agent Skills 的架构和现实世界应用的深入探讨，请阅读[为现实世界配备代理的 Agent Skills](https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills)。
</Note>

## 配置 Skills

本部分涵盖 Skill 文件结构、支持文件、工具限制和分发选项。

### 编写 SKILL.md

`SKILL.md` 文件是 Skill 中唯一必需的文件。它有两部分：顶部的 YAML 元数据（`---` 标记之间的部分）和告诉 Claude 如何使用 Skill 的 Markdown 说明：

```yaml  theme={null}
---
name: your-skill-name
description: Brief description of what this Skill does and when to use it
---

# Your Skill Name

## Instructions
Provide clear, step-by-step guidance for Claude.

## Examples
Show concrete examples of using this Skill.
```

#### 可用的元数据字段

你可以在 YAML 前置部分中使用以下字段：

| 字段               | 必需 | 描述                                                                                                                                                                                                   |
| :--------------- | :- | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `name`           | 是  | Skill 名称。必须仅使用小写字母、数字和连字符（最多 64 个字符）。应与目录名称匹配。                                                                                                                                                       |
| `description`    | 是  | Skill 的功能和何时使用它（最多 1024 个字符）。Claude 使用这个来决定何时应用 Skill。                                                                                                                                               |
| `allowed-tools`  | 否  | 当此 Skill 活跃时，Claude 可以使用而无需请求权限的工具。支持逗号分隔的值或 YAML 风格的列表。参见[限制工具访问](#restrict-tool-access-with-allowed-tools)。                                                                                        |
| `model`          | 否  | 当此 Skill 活跃时使用的[模型](https://docs.claude.com/zh-CN/docs/about-claude/models/overview)（例如，`claude-sonnet-4-20250514`）。默认为对话的模型。                                                                        |
| `context`        | 否  | 设置为 `fork` 以在具有自己对话历史的分叉子代理上下文中运行 Skill。                                                                                                                                                             |
| `agent`          | 否  | 指定当设置 `context: fork` 时使用哪个[代理类型](/zh-CN/sub-agents#built-in-subagents)（例如，`Explore`、`Plan`、`general-purpose` 或来自 `.claude/agents/` 的自定义代理名称）。如果未指定，默认为 `general-purpose`。仅在与 `context: fork` 结合时适用。 |
| `hooks`          | 否  | 定义限定于此 Skill 生命周期的 hooks。支持 `PreToolUse`、`PostToolUse` 和 `Stop` 事件。                                                                                                                                  |
| `user-invocable` | 否  | 控制 Skill 是否出现在斜杠命令菜单中。不影响[`Skill` 工具](/zh-CN/slash-commands#skill-tool)或自动发现。默认为 `true`。参见[控制 Skill 可见性](#control-skill-visibility)。                                                                 |

有关完整的编写指导，包括验证规则，请参阅[最佳实践指南](https://docs.claude.com/zh-CN/docs/agents-and-tools/agent-skills/best-practices)。

### 更新或删除 Skill

要更新 Skill，直接编辑其 `SKILL.md` 文件。要删除 Skill，删除其目录。更改立即生效。

### 使用渐进式披露添加支持文件

Skills 与对话历史、其他 Skills 和你的请求共享 Claude 的上下文窗口。为了保持上下文集中，使用**渐进式披露**：将必要信息放在 `SKILL.md` 中，将详细的参考资料放在 Claude 仅在需要时读取的单独文件中。

这种方法让你可以捆绑全面的文档、示例和脚本，而不会提前消耗上下文。Claude 仅在任务需要时加载其他文件。

<Tip>保持 `SKILL.md` 在 500 行以下以获得最佳性能。如果你的内容超过这个，将详细的参考资料分成单独的文件。</Tip>

#### 示例：多文件 Skill 结构

Claude 通过 `SKILL.md` 中的链接发现支持文件。以下示例显示了一个 Skill，其详细文档在单独的文件中，以及 Claude 可以执行而无需读取的实用脚本：

```
my-skill/
├── SKILL.md (required - overview and navigation)
├── reference.md (detailed API docs - loaded when needed)
├── examples.md (usage examples - loaded when needed)
└── scripts/
    └── helper.py (utility script - executed, not loaded)
```

`SKILL.md` 文件引用这些支持文件，以便 Claude 知道它们存在：

````markdown  theme={null}
## Overview

[Essential instructions here]

## Additional resources

- For complete API details, see [reference.md](reference.md)
- For usage examples, see [examples.md](examples.md)

## Utility scripts

To validate input files, run the helper script. It checks for required fields and returns any validation errors:
```bash
python scripts/helper.py input.txt
```
````

<Tip>保持引用一级深。直接从 `SKILL.md` 链接到参考文件。深层嵌套的引用（文件 A 链接到文件 B，文件 B 链接到文件 C）可能导致 Claude 部分读取文件。</Tip>

**为零上下文执行捆绑实用脚本。** 你的 Skill 目录中的脚本可以在不加载其内容到上下文的情况下执行。Claude 运行脚本，只有输出消耗令牌。这对以下情况很有用：

* 复杂的验证逻辑，用散文描述会很冗长
* 作为经过测试的代码比生成的代码更可靠的数据处理
* 受益于跨使用一致性的操作

在 `SKILL.md` 中，告诉 Claude 运行脚本而不是读取它：

```markdown  theme={null}
Run the validation script to check the form:
python scripts/validate_form.py input.pdf
```

有关构建 Skills 的完整指导，请参阅[最佳实践指南](https://docs.claude.com/zh-CN/docs/agents-and-tools/agent-skills/best-practices#progressive-disclosure-patterns)。

### 使用 allowed-tools 限制工具访问

使用 `allowed-tools` 前置部分字段来限制当 Skill 活跃时 Claude 可以使用哪些工具。你可以将工具指定为逗号分隔的字符串或 YAML 列表：

```yaml  theme={null}
---
name: reading-files-safely
description: Read files without making changes. Use when you need read-only file access.
allowed-tools: Read, Grep, Glob
---
```

或使用 YAML 风格的列表以获得更好的可读性：

```yaml  theme={null}
---
name: reading-files-safely
description: Read files without making changes. Use when you need read-only file access.
allowed-tools:
  - Read
  - Grep
  - Glob
---
```

当此 Skill 活跃时，Claude 只能使用指定的工具（Read、Grep、Glob）而无需请求权限。这对以下情况很有用：

* 不应修改文件的只读 Skills
* 范围有限的 Skills：例如，仅数据分析，无文件写入
* 你想限制功能的安全敏感工作流

如果省略 `allowed-tools`，Skill 不会限制工具。Claude 使用其标准权限模型，可能会要求你批准工具使用。

<Note>
  `allowed-tools` 仅在 Claude Code 中的 Skills 支持。
</Note>

### 在分叉上下文中运行 Skills

使用 `context: fork` 在具有自己对话历史的隔离子代理上下文中运行 Skill。这对于执行复杂的多步骤操作而不会使主对话混乱的 Skills 很有用：

```yaml  theme={null}
---
name: code-analysis
description: Analyze code quality and generate detailed reports
context: fork
---
```

### 为 Skills 定义 Hooks

Skills 可以定义在 Skill 生命周期期间运行的 hooks。使用 `hooks` 字段指定 `PreToolUse`、`PostToolUse` 或 `Stop` 处理程序：

```yaml  theme={null}
---
name: secure-operations
description: Perform operations with additional security checks
hooks:
  PreToolUse:
    - matcher: "Bash"
      hooks:
        - type: command
          command: "./scripts/security-check.sh $TOOL_INPUT"
          once: true
---
```

`once: true` 选项每个会话仅运行一次 hook。在第一次成功执行后，hook 被删除。

在 Skill 中定义的 Hooks 限定于该 Skill 的执行，并在 Skill 完成时自动清理。

有关完整的 hook 配置格式，请参阅[Hooks](/zh-CN/hooks)。

### 控制 Skill 可见性

Skills 可以通过三种方式调用：

1. **手动调用**：你在提示中输入 `/skill-name`
2. **程序调用**：Claude 通过[`Skill` 工具](/zh-CN/slash-commands#skill-tool)调用它
3. **自动发现**：Claude 读取 Skill 的描述并在与对话相关时加载它

`user-invocable` 字段仅控制手动调用。当设置为 `false` 时，Skill 从斜杠命令菜单中隐藏，但 Claude 仍然可以通过程序调用或自动发现它。

要通过 `Skill` 工具阻止程序调用，请改用 `disable-model-invocation: true`。

#### 何时使用每个设置

| 设置                               | 斜杠菜单 | `Skill` 工具 | 自动发现 | 用例                               |
| :------------------------------- | :--- | :--------- | :--- | :------------------------------- |
| `user-invocable: true`（默认）       | 可见   | 允许         | 是    | 你想让用户直接调用的 Skills                |
| `user-invocable: false`          | 隐藏   | 允许         | 是    | Claude 可以使用但用户不应手动调用的 Skills     |
| `disable-model-invocation: true` | 可见   | 阻止         | 是    | 你想让用户调用但 Claude 不能通过程序调用的 Skills |

#### 示例：仅模型 Skill

设置 `user-invocable: false` 以从斜杠菜单中隐藏 Skill，同时仍然允许 Claude 通过程序调用它：

```yaml  theme={null}
---
name: internal-review-standards
description: Apply internal code review standards when reviewing pull requests
user-invocable: false
---
```

使用此设置，用户不会在 `/` 菜单中看到 Skill，但 Claude 仍然可以通过 `Skill` 工具调用它或根据上下文自动发现它。

### Skills 和子代理

Skills 和子代理可以通过两种方式协同工作：

#### 给子代理访问 Skills

[子代理](/zh-CN/sub-agents)不会自动从主对话继承 Skills。要给自定义子代理访问特定 Skills，在子代理的 `skills` 字段中列出它们：

```yaml  theme={null}
# .claude/agents/code-reviewer.md
---
name: code-reviewer
description: Review code for quality and best practices
skills: pr-review, security-check
---
```

列出的 Skills 在子代理启动时加载到其上下文中。如果省略 `skills` 字段，则不会为该子代理预加载任何 Skills。

<Note>
  内置代理（Explore、Plan、general-purpose）无法访问你的 Skills。只有你在 `.claude/agents/` 中定义的具有显式 `skills` 字段的自定义子代理可以使用 Skills。
</Note>

#### 在子代理上下文中运行 Skill

使用 `context: fork` 和 `agent` 在具有自己单独上下文的分叉子代理中运行 Skill。有关详细信息，请参阅[在分叉上下文中运行 Skills](#run-skills-in-a-forked-context)。

### 分发 Skills

你可以通过多种方式共享 Skills：

* **项目 Skills**：将 `.claude/skills/` 提交到版本控制。任何克隆存储库的人都会获得 Skills。
* **插件**：要在多个存储库中共享 Skills，在你的[插件](/zh-CN/plugins)中创建一个 `skills/` 目录，其中包含包含 `SKILL.md` 文件的 Skill 文件夹。通过[插件市场](/zh-CN/plugin-marketplaces)分发。
* **托管**：管理员可以通过[托管设置](/zh-CN/iam#managed-settings)在组织范围内部署 Skills。有关托管 Skill 路径，请参阅[Skills 存放在哪里](#where-skills-live)。

## 示例

这些示例展示了常见的 Skill 模式，从最小的单文件 Skills 到具有支持文档和脚本的多文件 Skills。

### 简单 Skill（单文件）

最小的 Skill 仅需要一个带有前置部分和说明的 `SKILL.md` 文件。这个例子通过检查暂存的更改来帮助 Claude 生成提交消息：

```
commit-helper/
└── SKILL.md
```

```yaml  theme={null}
---
name: generating-commit-messages
description: Generates clear commit messages from git diffs. Use when writing commit messages or reviewing staged changes.
---

# Generating Commit Messages

## Instructions

1. Run `git diff --staged` to see changes
2. I'll suggest a commit message with:
   - Summary under 50 characters
   - Detailed description
   - Affected components

## Best practices

- Use present tense
- Explain what and why, not how
```

### 使用多个文件

对于复杂的 Skills，使用渐进式披露来保持主 `SKILL.md` 集中，同时在支持文件中提供详细文档。这个 PDF 处理 Skill 包括参考文档、实用脚本，并使用 `allowed-tools` 将 Claude 限制为特定工具：

```
pdf-processing/
├── SKILL.md              # Overview and quick start
├── FORMS.md              # Form field mappings and filling instructions
├── REFERENCE.md          # API details for pypdf and pdfplumber
└── scripts/
    ├── fill_form.py      # Utility to populate form fields
    └── validate.py       # Checks PDFs for required fields
```

**`SKILL.md`**:

````yaml  theme={null}
---
name: pdf-processing
description: Extract text, fill forms, merge PDFs. Use when working with PDF files, forms, or document extraction. Requires pypdf and pdfplumber packages.
allowed-tools: Read, Bash(python:*)
---

# PDF Processing

## Quick start

Extract text:
```python
import pdfplumber
with pdfplumber.open("doc.pdf") as pdf:
    text = pdf.pages[0].extract_text()
```

For form filling, see [FORMS.md](FORMS.md).
For detailed API reference, see [REFERENCE.md](REFERENCE.md).

## Requirements

Packages must be installed in your environment:
```bash
pip install pypdf pdfplumber
```
````

<Note>
  如果你的 Skill 需要外部包，在描述中列出它们。在 Claude 可以使用它们之前，必须在你的环境中安装包。
</Note>

## 故障排除

### 查看和测试 Skills

要查看 Claude 可以访问哪些 Skills，向 Claude 提出一个问题，例如"What Skills are available?"Claude 在对话开始时将所有可用的 Skill 名称和描述加载到上下文窗口中，以便它可以列出它当前可以访问的 Skills。

要测试特定的 Skill，要求 Claude 执行与 Skill 描述相匹配的任务。例如，如果你的 Skill 的描述是"Reviews pull requests for code quality"，要求 Claude"Review the changes in my current branch."Claude 在请求与其描述相匹配时自动使用 Skill。

### Skill 不触发

description 字段是 Claude 决定是否使用你的 Skill 的方式。模糊的描述，如"Helps with documents"不会给 Claude 足够的信息来将你的 Skill 与相关请求相匹配。

一个好的描述回答两个问题：

1. **这个 Skill 做什么？** 列出具体的功能。
2. **Claude 何时应该使用它？** 包含用户会提到的触发术语。

```yaml  theme={null}
description: Extract text and tables from PDF files, fill forms, merge documents. Use when working with PDF files or when the user mentions PDFs, forms, or document extraction.
```

这个描述有效是因为它命名了具体的操作（extract、fill、merge）并包含用户会说的关键词（PDF、forms、document extraction）。

### Skill 不加载

**检查文件路径。** Skills 必须在正确的目录中，文件名完全是 `SKILL.md`（区分大小写）：

| 类型 | 路径                                             |
| :- | :--------------------------------------------- |
| 个人 | `~/.claude/skills/my-skill/SKILL.md`           |
| 项目 | `.claude/skills/my-skill/SKILL.md`             |
| 企业 | 有关平台特定路径，请参阅[Skills 存放在哪里](#where-skills-live) |
| 插件 | 插件目录内的 `skills/my-skill/SKILL.md`              |

**检查 YAML 语法。** 前置部分中的无效 YAML 会阻止 Skill 加载。前置部分必须以第 1 行的 `---` 开始（前面没有空行），以 Markdown 内容前的 `---` 结束，并使用空格进行缩进（不是制表符）。

**运行调试模式。** 使用 `claude --debug` 查看 Skill 加载错误。

### Skill 有错误

**检查依赖项是否已安装。** 如果你的 Skill 使用外部包，在 Claude 可以使用它们之前，必须在你的环境中安装它们。

**检查脚本权限。** 脚本需要执行权限：`chmod +x scripts/*.py`

**检查文件路径。** 在所有路径中使用正斜杠（Unix 风格）。使用 `scripts/helper.py`，而不是 `scripts\helper.py`。

### 多个 Skills 冲突

如果 Claude 使用了错误的 Skill 或似乎在相似的 Skills 之间混淆，描述可能太相似了。通过使用特定的触发术语使每个描述不同。

例如，与其有两个 Skills 都在描述中有"data analysis"，不如区分它们：一个用于"sales data in Excel files and CRM exports"，另一个用于"log files and system metrics"。你的触发术语越具体，Claude 就越容易将正确的 Skill 与你的请求相匹配。

### 插件 Skills 不出现

**症状**：你从市场安装了一个插件，但当你问 Claude"What Skills are available?"时，其 Skills 不出现。

**解决方案**：清除插件缓存并重新安装：

```bash  theme={null}
rm -rf ~/.claude/plugins/cache
```

然后重启 Claude Code 并重新安装插件：

```shell  theme={null}
/plugin install plugin-name@marketplace-name
```

这会强制 Claude Code 重新下载并重新注册插件的 Skills。

**如果 Skills 仍然不出现**，验证插件的目录结构是否正确。Skills 必须在插件根目录的 `skills/` 目录中：

```
my-plugin/
├── .claude-plugin/
│   └── plugin.json
└── skills/
    └── my-skill/
        └── SKILL.md
```