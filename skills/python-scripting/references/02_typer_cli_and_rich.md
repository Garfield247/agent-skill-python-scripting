# 现代 CLI 命令行构建与 Rich 交互式终端 (Typer & Rich)

## 1. 基于 Typer 的现代强类型 CLI 脚本

淘汰冗长样板的 `argparse`，统一采用基于类型注解的 `Typer`：

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

    console.print("[bold green]✔ 处理完成![/bold green]")

if __name__ == "__main__":
    app()
```

---

## 2. Rich 交互式进度条 (ETA 与速率监控)

长耗时批处理任务必须输出具有进度估算的 Rich 进度条：

```python
from rich.progress import Progress, SpinnerColumn, BarColumn, TextColumn, TimeRemainingColumn

with Progress(
    SpinnerColumn(),
    TextColumn("[progress.description]{task.description}"),
    BarColumn(),
    TextColumn("[progress.percentage]{task.percentage:>3.0f}%"),
    TimeRemainingColumn()
) as progress:
    task = progress.add_task("[cyan]正在批量迁移用户数据...", total=10000)
    for batch in fetch_batches():
        process_batch(batch)
        progress.update(task, advance=len(batch))
```
