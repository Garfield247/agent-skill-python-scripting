# 幂等批处理设计与排障诊断武器库 (Batch Processing & Troubleshooting)

## 1. 批处理幂等性设计准则

1. **重复执行安全 (Idempotent)**：多次运行同一迁移或清理脚本产生完全相同的结果，严禁产生重复数据；
2. **断点跳过机制**：支持 `--resume` 参数，跳过已处理的唯一业务键；
3. **Dry-Run 模式支持**：破坏性操作脚本必须支持 `--dry-run` 仅输出模拟影响面。

---

## 2. 经典脚本排障速查矩阵

| 现象状态 | 根本原因剖析 (RCA) | 修复武器与处置策略 |
| :--- | :--- | :--- |
| **子进程执行卡死无响应** | 未设置 `timeout` 且外部命令等待用户输入（如 `ssh` 询问指纹、`sudo` 等待密码） | 必须显式传入 `timeout=N`，并设置非交互参数（如 `ssh -o BatchMode=yes`） |
| **管道缓冲区死锁 (Deadlock)** | 使用 `Popen` 设置了 `stdout=PIPE` 但未及时消费，管道 64KB 缓冲区打满导致子进程阻塞 | 统一采用 `subprocess.run(capture_output=True)` 自动消费管道数据 |
| **处理大文件被系统 OOM 强杀 (137)** | 一次性 `readlines()` 将数 GB 文件全量灌入内存 | 改用分块生成器或按行流式消费 |
| **本地运行正常但在 Cron 中报 command not found** | Cron 运行环境的 `PATH` 与当前用户终端环境变量不同步 | 在脚本中写死外部二进制工具的绝对路径（如 `/usr/local/bin/git`），或显式指定环境字典 |
