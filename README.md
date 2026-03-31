# Callit Skill

`callit-skill` 是一个独立的 skill 项目，用于让 AI agent 通过 `callit-mcp` 管理 Callit Worker。

## Skill 列表
### callit-worker
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
6. When finished, tell me the final status and how to use it later via `$callit-worker` or `/callit-worker`.

### Install MCP (if need)
You need to first ask the user for the callit service URL and MCP Token.
If the user only provides a host (for example, `example.com`), add the scheme and `/mcp` path as needed.
1. Name the MCP `callit-mcp`, and use HTTP transport.
2. Authenticate with the HTTP header `Authorization: Bearer <callit-mcp-token>`.

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

### Claude Code
1. **Clone the Callit-skill repository:**  
```bash
git clone https://github.com/YangZxi/Callit-skill /tmp
```
2. **Copy the skill to Claude**  
Replace `<skill-name>` with the specific Skill folder name. The skill name is provided in the [Skill 列表](#skill-列表).
```bash
cp /tmp/Callit-skill/<skill-name> ~/.codex/skills/
```

### Other Agent Applications
1. **Clone the Callit-skill repository to yourself desired location (or tmp directory):**  
   ```bash
   git clone https://github.com/YangZxi/Callit-skill
   ```
2. **Install the skill according to your agent application's skill installation method**  
3. **Delete cloned items as needed:**  

安装完成后，提醒用户可能需要重启 agent 应用来加载新技能。

