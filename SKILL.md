---
name: python-scripting
description: >-
  Python 3.10+ 现代自动化运维脚本、CLI 命令行工具与批处理工程规范技能。
  涵盖 Typer/Click 现代化命令行交互规范、安全 Subprocess 进程调用 (严禁 shell=True, 强制显式 timeout)、
  pathlib.Path 现代路径与流式文件处理、Linux 标准退出码语义、Loguru/Rich 结构化控制台及轻量并发执行池。
---

# Python 3.10+ 现代脚本与 CLI 工具开发规范技能 (Python Scripting Skill)

## 概述 (Overview)

本技能定义了研发工程师与 AI 编码助手在基于 **Python 3.10+** 构建自动化运维脚本、CLI 命令行工具、数据迁移与日常批处理程序时的现代工程标准。

### 核心设计原则

1. **现代化 CLI 交互**：弃用冗长过时的 `argparse`，统一采用基于类型提示的 `Typer`（或 `Click`），实现开箱即用的类型校验与自解释 `--help`。
2. **安全进程调度（核心红线）**：外部命令调用必须使用 `subprocess.run(list)`，**绝对禁止 `shell=True`**，且**必须显式声明 `timeout`** 避免进程永久挂死。
3. **现代路径与流式 I/O**：全面采用 `pathlib.Path`，大文件处理必须按块（Chunk）流式读写，严禁一次性载入内存导致 OOM。
4. **规范退出码与信号兜底**：严格遵循 POSIX 标准退出码（`0` 正常，`1` 业务错误，`2` 参数错误），注册信号钩子（SIGINT/SIGTERM）保障临时资源清理。
5. **结构化控制台与日志**：采用 `loguru` 或 `rich` 提供格式化输出，禁止在生产脚本中随意使用裸 `print()`。

---

# 1. 安全子进程调用核心红线 (Safe Subprocess Execution)

外部系统命令调用是自动化脚本的高危区。必须遵循以下黄金法则：

| 检查项 | 违规反例 (禁止) | 正确标准 (强制) |
| :--- | :--- | :--- |
| **Shell 注入防范** | `subprocess.run(f"rm -rf {path}", shell=True)` | `subprocess.run(["rm", "-rf", str(path)], shell=False)` |
| **防进程永久挂起** | `subprocess.run(["tar", "-czf", ...])` (无超时) | 显式指定 `timeout=300` |
| **错误捕获处置** | 忽略返回状态码，导致后续静默出错 | `check=True` 或显式判断 `result.returncode == 0` |
| **文本解码控制** | 未指定编码导致乱码 | 指定 `text=True, encoding="utf-8", errors="replace"` |

### 生产级外部调用封装模板

```python
import subprocess
import logging
from pathlib import Path

logger = logging.getLogger(__name__)

def run_cmd(args: list[str], cwd: Path | None = None, timeout: int = 60) -> str:
    """安全执行外部系统命令并返回 stdout 结果
    
    Args:
        args: 命令与参数列表（严禁拼接字符串）
        cwd: 命令工作目录
        timeout: 超时时间（秒）
    """
    logger.debug(f"Executing: {' '.join(args)} (cwd={cwd})")
    try:
        proc = subprocess.run(
            args,
            cwd=cwd,
            timeout=timeout,
            shell=False, # 严禁开启 shell=True 防止注入
            check=True,  # 非零退出码自动抛出 CalledProcessError
            capture_output=True,
            text=True,
            encoding="utf-8",
            errors="replace"
        )
        return proc.stdout.strip()
    except subprocess.TimeoutExpired as e:
        logger.error(f"Command timed out after {timeout}s: {' '.join(args)}")
        raise RuntimeError(f"命令执行超时: {' '.join(args)}") from e
    except subprocess.CalledProcessError as e:
        logger.error(f"Command failed with code {e.returncode}: {e.stderr.strip()}")
        raise RuntimeError(f"命令执行失败 [{e.returncode}]: {e.stderr.strip()}") from e
```

---

# 2. 现代 CLI 构建标准 (Typer & Rich)

全面采用现代类型友好的 `Typer` 进行脚本入口编写：

```python
from pathlib import Path
from typing import Annotated
import typer
from rich.console import Console

app = typer.Typer(help="现代化数据处理与运维批处理工具", add_completion=False)
console = Console()

@app.command()
def process(
    input_file: Annotated[
        Path,
        typer.Option(
            "--input", "-i",
            help="待处理的源数据文件路径",
            exists=True,
            file_okay=True,
            dir_okay=False,
            readable=True,
            resolve_path=True,
        )
    ],
    batch_size: Annotated[
        int,
        typer.Option("--batch-size", "-b", help="每批处理的记录数", min=1, max=5000)
    ] = 100,
    dry_run: Annotated[
        bool,
        typer.Option("--dry-run", help="仅模拟运行，不写入实际变更")
    ] = False,
):
    """批量清洗并转换数据文件"""
    console.print(f"[bold green]开始处理:[/bold green] {input_file} (Batch Size: {batch_size})")
    if dry_run:
        console.print("[yellow]⚠ 当前处于 Dry-Run 模式，变更不会落地[/yellow]")

    # 业务处理逻辑...
    console.print("[bold green]✔ 处理完成![/bold green]")

if __name__ == "__main__":
    app()
```

---

# 3. 现代路径处理与流式 I/O (Pathlib & Streaming)

1. **全面使用 `pathlib.Path`**：
   ```python
   from pathlib import Path

   # 严禁使用 os.path.join(os.path.dirname(...), "data")
   base_dir = Path(__file__).resolve().parent
   data_file = base_dir / "assets" / "report.json"

   # 快速安全检查与读写
   if data_file.exists():
       content = data_file.read_text(encoding="utf-8")
   ```

2. **超大文件分块流式读取（防止 OOM）**：
   ```python
   def process_large_file(file_path: Path, chunk_size: int = 64 * 1024):
       """按块流式读取大文件"""
       with open(file_path, mode="r", encoding="utf-8", errors="replace") as f:
           while chunk := f.read(chunk_size):
               yield chunk
   ```

---

# 4. 退出码与信号清理机制 (Exit Codes & Signal Handling)

严格遵循标准退出码定义，确保 CI/CD 流水线与外部调度系统正确识别状态：

```python
import sys
import signal

def sigint_handler(signum, frame):
    """捕获中断信号，执行优雅善后"""
    print("\n[!] 收到用户中断信号 (SIGINT)，正在清理临时文件并安全退出...", file=sys.stderr)
    cleanup_temp_resources()
    sys.exit(130) # 128 + 2 (SIGINT)

signal.signal(signal.SIGINT, sigint_handler)
signal.signal(signal.SIGTERM, sigint_handler)

# 退出码规范:
# 0: 成功 (Success)
# 1: 业务/运行时错误 (General Runtime Error)
# 2: 命令行参数不合法 (Misuse of Shell Builtins / Argument Error)
# 130: 脚本被用户终止 (Ctrl+C)
```

---

# 5. 脚本研发 Checklist

- [ ] 外部系统命令是否使用列表形式且未开启 `shell=True`？
- [ ] 所有子进程调用是否设置了明确的 `timeout` 超时防挂死机制？
- [ ] 路径操作是否已统一采用 `pathlib.Path` 替代过时的 `os.path`？
- [ ] 是否正确设置了标准退出码（成功 `0`，失败非 `0`）？
- [ ] 涉及大文件操作时，是否采用了分块流式读取？
- [ ] 关键控制台输出是否具备时间戳与结构化日志分级？

---

# 6. Bug 分析、排查与子进程异常诊断 (Troubleshooting & Process Diagnostics)

脚本在执行批处理或系统运维操作时发生异常，遵循以下指引定位：

### 6.1 子进程挂起与永久卡死排查 (Subprocess Hang)
- **根因分析**：
  1. **未设置超时**：未指定 `timeout` 导致外部命令等待用户输入（如 `ssh` 提示确认主机指纹、`sudo` 等待密码交互）；
  2. **管道缓冲区死锁 (Pipe Buffer Deadlock)**：使用 `subprocess.Popen(stdout=PIPE, stderr=PIPE)` 但未及时读取，导致操作系统管道缓冲区（64KB）满，子进程永远阻塞。
- **武器与修复**：
  统一采用 `subprocess.run(args, timeout=N, capture_output=True, text=True)`，严禁手动管理裸 Popen 管道。

### 6.2 内存溢出 (OOM) 与文件句柄泄漏
- **现象**：脚本在处理几个 GB 的大型日志或 CSV 文件时进程被操作系统 OOM Killer 强杀（Exit 137）；
- **排查与修复**：
  检查是否存在 `file.readlines()` 或 `f.read()` 一次性全量加载；必须改为 `for line in file:` 或固定分块（Chunk-based Streaming）流式消费。

### 6.3 非零退出码与管道中断排查
- 捕获 `subprocess.CalledProcessError`，务必打印 `e.stderr.strip()` 提供现场证据；
- 检查命令在目标环境的依赖路径（`PATH` 差异），避免在本地可执行但在生产 cron 环境中因环境变量缺失报错 `command not found`。
