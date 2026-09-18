# python-scripting

> Python 3.10+ 现代自动化运维脚本、CLI 命令行工具与批处理工程规范技能库。

## 🌟 核心特性 (Features)

- **安全进程调用黄金法则**：列表参数调用、绝对禁止 `shell=True`、强制显式 `timeout` 防挂死。
- **现代化 CLI 框架**：全面拥抱 `Typer` + `Rich`，开箱即用的类型校验与高可读终端展示。
- **现代路径与流式 I/O**：全面 `pathlib.Path` 化，大文件分块流式读写拒绝 OOM。
- **规范进程生命周期**：严格遵循 POSIX 标准退出码，信号拦截保障临时资源清理。

## 📦 安装与加载 (Installation)

### 方式 1: 安装至 Antigravity / Gemini 全局技能库
```bash
git clone git@github.com:Garfield247/agent-skill-python-scripting.git ~/.gemini/config/skills/python-scripting
```

### 方式 2: 在任意项目中作为本地工作区技能引入
```bash
mkdir -p .agents/skills
git clone git@github.com:Garfield247/agent-skill-python-scripting.git .agents/skills/python-scripting
```

## 📄 开源协议 (License)
本项目采用 [MIT License](LICENSE) 授权。
