# WinRimage 图像预览模块

这是 WinRimage 的独立图像预览与剪贴板适配层。详细 API、快速预览、精确 rimage 预览和剪贴板用法请参阅 `preview.md`。

核心模块：

- `rimage.preview`：图像信息、GDI+ 快速预览、rimage 精确预览。
- `rimage.clipboard`：剪贴板图像读写和资源释放。

精确预览可能出现“编码成功但当前组件不可显示”的情况。此时结果仍然有效，使用 `displayable`、`displayError` 和 `path` 判断显示能力与输出文件。
