# 安全子进程执行规范 (Safe Subprocess Execution)

## 1. 外部系统命令调用黄金军规

| 检查维度 | 违规反例 (坚决禁止) | 正规生产标准 (强制执行) |
| :--- | :--- | :--- |
| **Shell 注入防范** | `subprocess.run(f"rm -rf {path}", shell=True)` | `subprocess.run(["rm", "-rf", str(path)], shell=False)` |
| **防进程永久挂起** | `subprocess.run(["tar", "-czf", ...])` (无超时) | 显式声明 `timeout=300` |
| **错误捕获处置** | 忽略返回状态码，导致后续静默失败 | `check=True` 或显式校验 `proc.returncode == 0` |
| **文本解码控制** | 未指定编码导致终端乱码 | 指定 `text=True, encoding="utf-8", errors="replace"` |

---

## 2. 生产级安全命令封装标杆 (`run_cmd`)

```python
import subprocess
import logging
from pathlib import Path

logger = logging.getLogger(__name__)

def run_cmd(args: list[str], cwd: Path | None = None, timeout: int = 60) -> str:
    """安全执行外部系统命令并返回 stdout 输出字符串
    
    Args:
        args: 命令与参数列表（严禁拼接成单个字符串传参）
        cwd: 命令工作目录
        timeout: 超时时间（秒）
    """
    logger.debug(f"Executing: {' '.join(args)} (cwd={cwd})")
    try:
        proc = subprocess.run(
            args,
            cwd=cwd,
            timeout=timeout,
            shell=False, # 严禁开启 shell=True 防止注入攻击
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
