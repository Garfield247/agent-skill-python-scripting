# POSIX 标准退出码与系统信号清理 (Exit Codes & Signals)

## 1. POSIX 标准退出码规范

确保 CI/CD 流水线与调度系统精准感知执行结果：

| 退出码 | 语义 | 适用场景 |
| :---: | :--- | :--- |
| **`0`** | 成功 (Success) | 脚本全部预期逻辑无异常完成 |
| **`1`** | 运行时/业务错误 (Runtime Error) | 文件读取失败、业务校验不通过、网络请求异常 |
| **`2`** | CLI 参数错误 (Misuse of Command) | 缺少必填参数、参数格式校验不合法 |
| **`130`** | 脚本被用户主动中断 | 捕获到 `SIGINT` (Ctrl+C) |
| **`143`** | 进程被外部优雅终止 | 捕获到 `SIGTERM` (kill 信号) |

---

## 2. 信号优雅善后注册模板

```python
import sys
import signal

def sig_handler(signum, frame):
    """捕获中断信号，安全释放临时文件与网络资源"""
    print("\n[!] 收到系统终止信号，正在清理临时工作区并优雅退出...", file=sys.stderr)
    cleanup_temp_files()
    sys.exit(128 + signum)

# 显式接管中断信号
signal.signal(signal.SIGINT, sig_handler)
signal.signal(signal.SIGTERM, sig_handler)
```
