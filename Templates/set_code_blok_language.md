<%* 
// 创建代码块内容
let codeBlock = "```cpp\n\n```";
// 获取编辑器
let editor = app.workspace.activeLeaf.view.editor;
// 获取当前光标位置
let cursor = editor.getCursor();
// 插入代码块
editor.replaceRange(codeBlock, cursor);
// 移动光标到代码块内部
editor.setCursor({
    line: cursor.line + 1,
    ch: 0
});
-%>
