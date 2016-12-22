## OC操作
-----

#### 复制字符串到剪切板

UIPasteboard *pasteboard = [UIPasteboard generalPasteboard];
pasteboard.string = _telLabel.text;


