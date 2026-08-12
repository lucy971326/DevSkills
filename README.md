# DevSkills

个人开发 Agent Skills，沉淀经过真实开发验证的工作流与约束，帮助 Agent 基于证据工作，减少脱离代码和单一上下文带来的错误判断。

## 安装

### 交互式安装

运行命令，然后按提示选择 Skill、目标 Agent、安装范围和安装方式：

```powershell
npx skills add lucy971326/DevSkills
```

在 Agent 选择界面使用：

- `↑` / `↓`：移动
- `Space`：选择或取消
- `Enter`：确认

### 指定安装

例如，只安装 `dual-verify` 到 Codex 全局目录：

```powershell
npx skills add lucy971326/DevSkills --skill dual-verify --agent codex --global
```

查看仓库中的 Skills：

```powershell
npx skills add lucy971326/DevSkills --list
```

## 当前 Skills

### `dual-verify`

使用两个相互隔离的 Agent，从不同证据路径独立验证源码、架构设计和重要技术结论，再由主 Agent 交叉审核。

使用示例：

```text
请使用 $dual-verify，独立验证这个框架源码调用链的真实行为。
```

## 目录结构

```text
DevSkills/
├── README.md
└── skills/
    └── dual-verify/
        ├── SKILL.md
        └── agents/
            └── openai.yaml
```

每个 Skill 至少包含一个带 YAML frontmatter 的 `SKILL.md`。

## License

MIT
