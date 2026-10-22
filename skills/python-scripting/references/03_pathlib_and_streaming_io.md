# 现代路径处理与大文件流式 I/O (Pathlib & Streaming)

## 1. 全面采用 `pathlib.Path`

彻底废弃过时的 `os.path` 字符串拼接：

```python
from pathlib import Path

# 获取当前脚本所在绝对目录
base_dir = Path(__file__).resolve().parent
data_file = base_dir / "data" / "payload.json"

# 优雅快速判断与读写
if data_file.exists():
    content = data_file.read_text(encoding="utf-8")
```

---

## 2. 超大文件分块流式读取（杜绝 OOM）

严禁使用 `f.readlines()` 一次性全量读入 GB 级别日志或文件：

```python
from collections.abc import Iterator
from pathlib import Path

def process_large_file(file_path: Path, chunk_size: int = 64 * 1024) -> Iterator[str]:
    """按固定块大小流式生成数据，保障内存维持在极小常数级别"""
    with open(file_path, mode="r", encoding="utf-8", errors="replace") as f:
        while chunk := f.read(chunk_size):
            yield chunk

def process_lines_streaming(file_path: Path) -> Iterator[str]:
    """按行流式消费文件"""
    with open(file_path, mode="r", encoding="utf-8", errors="replace") as f:
        for line in f:
            yield line.rstrip("\r\n")
```
