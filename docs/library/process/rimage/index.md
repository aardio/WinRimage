# process.rimage

WinRimage 对 rimage 0.13.0 命令行工具的纯 aardio 封装。当前覆盖编码器能力、配置、参数生成、安全计划、同步执行和异步任务。

## 导入

```aardio
import process.rimage;
var config = process.rimage.defaultConfig();
config.outputDirectory = "C:\output";
```

## 设计原则

- GUI 不直接拼接 rimage 命令行。
- `buildArgs()` 返回适用于 `process.popen()` 的参数数组。
- 可选值参数使用 `--name=value` 单 token 形式。
- `plan()` 在启动前检查输入、输出和 metadata 路径冲突。
- 默认配置要求独立输出目录，不允许把原图作为默认输出目标。
- metadata 默认写入临时目录，任务完成后自动清理。

## 编码器能力

`process.rimage.codecs()` 返回 rimage 0.13.0 的编码器能力表。mozjpeg、webp、avif、oxipng 的选项不同，GUI 应按能力表显示控件。jpeg_xl 不提供通用 quality 参数。

## 配置与参数

```aardio
import process.rimage;
var config = process.rimage.defaultConfig();
config.outputDirectory = "C:\output";
config.quality = 78;
config.suffix = "_optimized";
config.resize = ["1600w", "75%"];
var checked,err = process.rimage.validateConfig(config);
if(!checked) return err;
var args = process.rimage.buildArgs(checked,["C:\images\photo.png"]);
```

## 检测 rimage

```aardio
var exe = "C:\tools\rimage.exe";
var info,err = process.rimage.detect(exe);
if(!info) return err;
```

`detect()` 使用 `process.popen` 执行 `--version`，不会经过 `cmd.exe`。

## 执行前计划

```aardio
var plan,err = process.rimage.plan(config,["C:\images\photo.png"]);
if(!plan) return err;
if(!plan.safe) return plan.errors;
```

计划对象包含 `inputs`、`outputs`、`warnings`、`errors`、`commonRoot` 和 `safe` 字段。`plan()` 不创建文件、不启动进程。

## 同步执行

```aardio
var result,err = process.rimage.run(config,["C:\images\photo.png"],{
    exePath = "C:\tools\rimage.exe"
});
if(!result) return err;
```

`run()` 会先调用 `plan()`，再创建 `process.popen`。未指定 `metadataPath` 时使用临时 JSON，读取结果后清理。结果中的 `output` 是 `process.popen.readAll()` 返回的合并输出，`stderr` 是标准错误流。

## 异步任务与取消

GUI 长任务应使用 `runAsync()`，不要在界面线程直接调用同步 `run()`：

```aardio
var task = process.rimage.runAsync(config,["C:\images\photo.png"],{
    exePath = "C:\tools\rimage.exe"
});

var result,err = task.wait(30000);
if(!result) return err;

// 长任务也可以由窗口定时器查询：
var snapshot = task.snapshot();
// 用户点击取消时：
// task.cancel();
// 窗体关闭或任务完成后：
task.close();
```

`runAsync()` 在独立工作线程中创建和读取 `process.popen`，使用 `each()` 持续获取 `stdout`、`stderr` 和合并输出；管道对象不会跨线程传递。任务状态包括：

```text
created / running / cancelling / completed / completedWithErrors / cancelled / failed
```

## 结果模型

最终结果包含：

- `state`
- `exitCode`
- `elapsedMs`
- `outputs`
- `successCount`
- `failedCount`
- `metadata`
- `stdout`
- `stderr`
- `output`

部分文件失败时，可能同时存在成功输出和非零退出码；GUI 必须逐项显示结果，不能只显示一个“失败”。

## 已知安全边界

WinRimage 不把 rimage 0.13.0 的 metadata 冲突检查当作唯一安全保障。`plan()` 会拒绝 metadata 路径等于输入或输出路径，并拒绝已有输出文件（除非明确启用 overwrite）。正式 GUI 还应在执行前展示输出计划；删除原图必须是后续独立的安全动作。
