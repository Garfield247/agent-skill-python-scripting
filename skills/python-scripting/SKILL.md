---
name: python-scripting
description: >-
  Python 3.10+ 现代自动化运维脚本、CLI 命令行工具与批处理工程规范技能。
  涵盖 Typer/Click 现代化命令行交互规范、安全 Subprocess 进程调用 (严禁 shell=True, 强制显式 timeout)、
  pathlib.Path 现代路径与流式文件处理、Linux 标准退出码语义、Loguru/Rich 结构化控制台及轻量并发执行池。
---

# Python 3.10+ 现代脚本与 CLI 工具开发规范 (Python Scripting Skill)

本技能定义了构建现代自动化运维脚本、CLI 工具与数据批处理程序时的安全门禁、工程质量基线与渐进式手册索引。

---

## 1. 核心铁律与绝对红线 (Core Redlines)

1. 🚨 **安全子进程执行门禁**：
   - 外部系统命令必须采用参数列表传参，**绝对禁止 `shell=True`**，严防注入漏洞；
   - 所有子进程调用**必须显式声明 `timeout` 超时参数**，杜绝因外部交互等待导致进程永久挂死；
   - 统一采用 `subprocess.run(..., check=True, capture_output=True)`。
2. 🚨 **现代化 CLI 与丰富控制台**：
   - 弃用陈旧的 `argparse`，统一采用基于类型注解的 `Typer`（或 `Click`）；
   - 关键操作必须具备 `--dry-run` 模拟演练模式；采用 `rich` 输出进度与格式化高熵信息。
3. 🚨 **现代路径与流式 I/O**：
   - 全面使用 `pathlib.Path`，彻底淘汰 `os.path` 字符串拼接；
   - 大文件处理必须采用按块生成器（Chunk Streaming）或逐行消费，严禁全量载入内存导致 OOM。
4. 🚨 **POSIX 退出码与信号清理闭环**：
   - 严格遵循标准退出码（`0` 成功，`1` 业务错误，`2` 参数错误）；
   - 必须注册 `SIGINT`/`SIGTERM` 信号钩子，确保被外部中断时安全清理临时工作区。

---

## 2. 标准开发工作流 (Standard Workflow)

1. **CLI 骨架**：基于 `Typer` 定义命令行子命令、参数选项与默认值；
2. **安全执行**：外部系统调用统一经过安全封装函数 `run_cmd` 并限定超时；
3. **流式处理**：涉及海量数据与文件读写，使用 `pathlib.Path` + 分块迭代器处理；
4. **中断与退出**：挂载信号监听并在执行结束返回明确 POSIX 退出码。

---

## 3. 渐进式专业参考手册 (Progressive Disclosure References)

根据具体涉及的业务场景，**按需查阅 `references/` 目录下的专门规约**，严禁将全量子文档一次性无脑全部展开：

| 场景模块 | 参考子文档 | 核心涵盖规范 |
| :--- | :--- | :--- |
| **子进程安全调用** | [01_subprocess_and_safety.md](references/01_subprocess_and_safety.md) | • 严禁 `shell=True` 与防 Shell 注入军规<br>• 强制 `timeout` 超时与标准 `run_cmd` 封装模版 |
| **现代 CLI 与交互** | [02_typer_cli_and_rich.md](references/02_typer_cli_and_rich.md) | • Typer 强类型命令行参数校验与自解释 `--help`<br>• Rich 交互式进度条 (ETA 与速率监控) |
| **现代路径与流式 I/O**| [03_pathlib_and_streaming_io.md](references/03_pathlib_and_streaming_io.md) | • `pathlib.Path` 现代文件系统抽象范式<br>• 超大文件固定分块流式生成器 (防 OOM) |
| **退出码与信号善后** | [04_exit_codes_and_signals.md](references/04_exit_codes_and_signals.md) | • POSIX 标准退出码语义字典 (`0`, `1`, `2`, `130`)<br>• `SIGINT`/`SIGTERM` 中断信号安全回收模版 |
| **批处理与排障武器库** | [05_batch_processing_and_troubleshooting.md](references/05_batch_processing_and_troubleshooting.md) | • 批处理幂等性设计与 `--resume` 断点跳过<br>• 管道死锁、OOM 137、Cron 环境缺失排障表 |
