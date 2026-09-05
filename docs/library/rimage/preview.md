# WinRimage 图像预览模块

## 当前定位

`rimage.preview` 是 GUI 与图像对象之间的独立适配层，不负责窗口布局，也不修改控件属性。

它提供两种预览：

- **快速预览**：由 GDI+ 在内存中编码 JPG/PNG，适合拖动质量或尺寸设置时快速反馈。
- **精确预览**：调用真实 rimage，在独立临时目录中生成结果，适合确认最终编码器结果和文件大小。

快速预览不代表 rimage 的最终结果；最终批处理结果以 rimage 实际输出为准。

## 快速预览

```aardio
import rimage.preview;

var bmp = gdip.bitmap("/res/input.png");
var result,err = rimage.preview.quick(bmp,{format="jpg";quality=80;scale=50});
if(result){
    print(result.width,result.height,result.bytes);
    // 将 result.bitmap 交给预览控件使用
    result.dispose();
}
```

支持的快速编码格式当前为 `jpg/jpeg` 和 `png`。AVIF、JPEG XL 等格式使用精确预览，避免把 GDI+ 的编码结果误认为 rimage 的编码结果。
WebP 也可以使用精确预览；WinRimage 会调用 `gdip.webp` 解码实际 rimage 输出。AVIF/JPEG XL 等格式如果 rimage 编码成功但当前图像组件不能解码，结果仍然是成功的编码结果，只是 `displayable=false`，GUI 应显示文件大小并提供打开文件操作，而不能误报为编码失败。

## 精确预览

```aardio
import rimage.preview;

var result,err = rimage.preview.exact(
    {encoder="mozjpeg";quality=82},
    "C:\images\photo.png",
    {exePath="C:\tools\rimage.exe"}
);
if(result){
    print(result.path,result.bytes,result.width,result.height);
    result.dispose();
}
```

精确预览会创建独立临时目录，调用完成后由 `dispose()` 删除临时目录。调用方如果要长期显示位图，应先创建自己的深拷贝，再释放结果。

## 生命周期

`quick()` 和 `exact()` 的返回值都拥有一个 `dispose()` 方法。GUI 替换预览图、关闭窗口或取消精确预览时必须调用它。

不要把 `gdip.bitmap` 放进长期配置表；配置表应只保存路径和数值参数。

## 剪贴板

```aardio
import rimage.clipboard;

var clip,err = rimage.clipboard.read();
if(clip){
    print(clip.width,clip.height);
    // 使用 clip.bitmap 后释放
    rimage.clipboard.writePng(clip.bitmap);
    clip.dispose();
}
```

`rimage.clipboard.writeHtml()` 等价能力通过：

```aardio
rimage.clipboard.write(bitmap,{html=true});
```

剪贴板属于系统全局状态，模块不会自动监听剪贴板变化；GUI 可根据需要用 `win.clip.viewer` 单独订阅。
