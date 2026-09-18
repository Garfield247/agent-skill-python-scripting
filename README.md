# python-scripting

> Python 3.10+ 现代自动化运维脚本、CLI 命令行工具与批处理工程规范技能库。

## 🌟 核心特性 (Features)

- **安全进程调用黄金法则**：列表参数调用、绝对禁止 `shell=True`、强制显式 `timeout` 防挂死。
- **现代化 CLI 框架**：全面拥抱 `Typer` + `Rich`，开箱即用的类型校验与高可读终端展示。
- **现代路径与流式 I/O**：全面 `pathlib.Path` 化，大文件分块流式读写拒绝 OOM。
- **规范进程生命周期**：严格遵循 POSIX 标准退出码，信号拦截保障临时资源清理。

## 📦 安装与多 Agent 使用指南 (Installation & Multi-Agent Usage)

本项目遵循开放 Agent 规范，支持在 **Gemini / Antigravity**、**Anthropic Claude**、**Cursor / Codex** 等各类主流 Agent 环境中一键安装与激活：

### 1. Google Antigravity / Gemini Code Assist
- **全局安装（推荐）**：
  ```bash
  git clone git@github.com:Garfield247/agent-skill-python-scripting.git ~/.gemini/config/skills/python-scripting
  ```
- **项目工作区局部引入**：
  ```bash
  mkdir -p .agents/skills
  git clone git@github.com:Garfield247/agent-skill-python-scripting.git .agents/skills/python-scripting
  ```

### 2. Anthropic Claude (Claude Code / Claude Projects)
- **Claude Code (CLI 终端智能体)**：
  克隆至 Claude 全局技能库：
  ```bash
  mkdir -p ~/.claude/skills
  git clone git@github.com:Garfield247/agent-skill-python-scripting.git ~/.claude/skills/python-scripting
  ```
  *或者在项目根目录的 `CLAUDE.md` 中追加引入：*
  ```markdown
  See detailed engineering specifications in: ~/.claude/skills/python-scripting/SKILL.md
  ```
- **Claude Projects (Web / 桌面端)**：
  直接将仓库中的 `SKILL.md` 内容复制并粘贴至 Project 的 **Project Knowledge (项目知识库)** 或 **Custom Instructions (自定义指令)** 中。

### 3. Cursor / GitHub Copilot / OpenAI Codex
- **Cursor (现代 MDC 规则体系)**：
  在项目根目录创建或链接规则：
  ```bash
  mkdir -p .cursor/rules
  # 克隆或软链接为 Cursor 专有规则文件
  git clone git@github.com:Garfield247/agent-skill-python-scripting.git .cursor/rules/python-scripting
  ```
- **GitHub Copilot / Codex**：
  将本技能规范注入 Copilot 指令集：
  ```bash
  mkdir -p .github
  cat << 'EOF' >> .github/copilot-instructions.md
  # 引入本技能核心规则
  EOF
  cat path/to/SKILL.md >> .github/copilot-instructions.md
  ```

## 📄 开源协议 (License)
本项目采用 [MIT License](LICENSE) 授权。
