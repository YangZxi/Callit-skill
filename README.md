# Callit Skill

`callit-skill` 是一个独立的 skill 项目，用于让 AI agent 通过 `callit-mcp` 管理 Callit Worker。

当前项目内提供的 skill 为：

- `callit-worker`

这个 skill 约束了 AI agent 在以下场景中的行为：

- 搜索 Worker
- 创建、修改 Worker
- 读取 Worker 文件列表和文件内容
- 上传、编辑、删除 Worker 中的文件或内容

## How to Install
Tell AI
```text
Fetch and follow instructions from https://raw.githubusercontent.com/YangZxi/Callit-skill/refs/heads/main/README.md to install the skill.
```

## Installation Guide (For AI Agent)
1. Before installing or updating, first check whether `callit-mcp` is already configured in the current environment.
2. If `callit-mcp` is missing, ask me first whether I want to install it.
3. If `callit-mcp` needs to be installed, then ask me to provide the remote URL and authentication token.
4. Do not guess the URL or token, and do not skip the confirmation step.
5. Only continue installing or updating `callit-worker` after `callit-mcp` has been configured successfully.
6. When finished, tell me the final status and how to use it later via `$callit-worker`.

### Codex
1. **Clone the Callit-skill repository:**
   ```bash
   git clone https://github.com/YangZxi/Callit-skill ~/.codex/skills/
   ```
2. **Create the skills symlink:**
   ```bash
   mkdir -p ~/.agents/skills
   ln -s ~/.codex/skills/callit-skill ~/.agents/skills/callit-skill
   ```

### Other Agent Applications
1. **Clone the Callit-skill repository to yourself desired location (or tmp directory):**
   ```bash
   git clone https://github.com/YangZxi/Callit-skill
   ```
2. **Install the skill according to your agent application's skill installation method**
3. **Delete cloned items as needed:**

安装完成后，提醒用户可能需要重启 agent 应用来加载新技能。

