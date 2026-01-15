Using Atlas with AI Agents | Atlas Guides

[Skip to main content](#__docusaurus_skipToContent_fallback)

v1.0 Atlas v1.0 is here! [Read the announcement →](/blog/2025/12/23/atlas-v1)

Copy page

AI agents like GitHub Copilot, Cursor, and Claude Code are great at writing code, but generating database migrations is a different challenge. As schemas grow larger and more complex, ensuring migrations are **deterministic**, **predictable**, and **aligned with company policies** becomes critical.

Atlas solves this problem by letting AI tools focus on editing the schema while Atlas provides the infrastructure for:

1.  [**Migration Generation**](/versioned/diff) - producing correct, safe, and deterministic migrations automatically.
2.  [**Migration Validation**](/versioned/lint) - ensuring migrations are semantically correct and follow best practices.
3.  [**Policy Enforcement**](/lint/analyzers) - applying organizational rules for what changes are allowed and how they should be applied.
4.  [**Unit Testing**](/testing/schema) - using the Atlas testing framework, AI tools can write logic (e.g., functions, views, and queries) along with tests, while Atlas executes them, reports failures, and lets the AI fix issues automatically.
5.  [**Data Migration Testing**](/testing/migrate) - the same framework enables AI tools to generate data migrations, seed data, run tests, detect errors, and guide the AI in resolving them automatically.

And more. Atlas offers a complete toolkit for safe, automated, and policy-driven database change management.

### Instruction files for AI agents[​](#instruction-files-for-ai-agents "Direct link to Instruction files for AI agents")


AI agents can be configured to adapt their behavior on a project or repository level by providing them with specific instructions or rules. This allows them to better understand the context of the code they are working with and provide more relevant suggestions.

To help your AI agent get the most out of Atlas, we have created a set of instructions and rules that can be used to configure them to work with Atlas. These instructions and rules are designed to help the AI agent understand how and when to use optimally use Atlas.

### Prompt Library[​](#prompt-library "Direct link to Prompt Library")


[

##### !GitHub Copilot Instructions


Configure GitHub Copilot with Atlas-specific instructions.

](/guides/ai-tools/github-copilot-instructions)[

##### !Cursor Instructions


Set up Cursor with Atlas-specific rules.

](/guides/ai-tools/cursor-rules)[

##### !Claude Code Instructions


Set up Claude Code with Atlas-specific instructions.

](/guides/ai-tools/claude-code-instructions)

### Example workflow - generating migrations[​](#example-workflow---generating-migrations "Direct link to Example workflow - generating migrations")


The above instructions teach the AI agent to use a specific workflow when generating migrations. When generating migrations, there are several steps that the AI agent should follow:

#### 1\. Analyze the current schema and make necessary changes.[​](#1-analyze-the-current-schema-and-make-necessary-changes "Direct link to 1. Analyze the current schema and make necessary changes.")


![Generating the migrations](/u/ai-tools/cursor-atlas-1.o.gif)

#### 2\. Generate the necessary migration files.[​](#2-generate-the-necessary-migration-files "Direct link to 2. Generate the necessary migration files.")


The AI agent should use `atlas migrate diff` to generate the migration file. After generating the migration, the AI agent should validate the migration file by running `atlas migrate lint` and fix any issues that arise.

![Generating the migrations](/u/ai-tools/cursor-atlas-2.o.gif)

#### 3\. Apply the migration files to the database.[​](#3-apply-the-migration-files-to-the-database "Direct link to 3. Apply the migration files to the database.")


The last step is applying the migration files to the database using `atlas migrate apply`. The assistant should first try applying a dry-run.

![Generating the migrations](/u/ai-tools/cursor-atlas-3.o.gif)

*   [Instruction files for AI agents](#instruction-files-for-ai-agents)
*   [Prompt Library](#prompt-library)
*   [Example workflow - generating migrations](#example-workflow---generating-migrations)