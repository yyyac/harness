# 领域文档

规定各工程 skill 在探索代码库时应如何消费本仓库的领域文档。

## 探索之前先读这些

- 仓库根目录的 **`GLOSSARY.md`**，或
- 仓库根目录的 **`GLOSSARY-MAP.md`**（若存在）：它指向每个上下文的 `GLOSSARY.md`，读取与当前主题相关的每一个。
- **`docs/adr/`**：读取涉及你即将动手区域的 ADR。多上下文仓库还应检查 `src/<context>/docs/adr/` 中上下文级的决策。

若这些文件不存在，**静默继续**。不要提示缺失，也不要主动建议先创建它们。`/domain-modeling` skill（经由 `/grill-with-docs` 与 `/improve-codebase-architecture` 触达）会在术语或决策真正落地时惰性创建这些文件。

## 文件结构

单上下文仓库（绝大多数仓库）：

```
/
├── GLOSSARY.md
├── docs/adr/
│   ├── 0001-event-sourced-orders.md
│   └── 0002-postgres-for-write-model.md
└── src/
```

多上下文仓库（根目录存在 `GLOSSARY-MAP.md`）：

```
/
├── GLOSSARY-MAP.md
├── docs/adr/                          ← 系统级决策
└── src/
    ├── ordering/
    │   ├── GLOSSARY.md
    │   └── docs/adr/                  ← 上下文本级决策
    └── billing/
        ├── GLOSSARY.md
        └── docs/adr/
```

## 使用 glossary 的词汇

当你的输出需要命名某个领域概念（issue 标题、重构提案、假设、测试名），使用 `GLOSSARY.md` 中定义的术语。不要漂移到 glossary 明确规避的同义词。

若所需概念尚未进入 glossary，这本身是一个信号：要么你在发明项目并未使用的语言（重新考虑），要么存在真实缺口（记录给 `/domain-modeling`）。

## 标出 ADR 冲突

若你的输出与现有 ADR 相矛盾，显式指出，而不是静默覆盖：

> _与 ADR-0007（event-sourced orders）冲突，但值得重开讨论，因为……_
