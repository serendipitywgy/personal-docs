# 【01】clangd 相关配置

## clangd/config.yaml配置
```yaml
CompileFlags:
  Add: [-std=c++23, -I/usr/include]
Index:
  Background: true
```

## .clang-format配置
说明：该配置文件即可放在项目根目录下，也可以放在家目录下。
```YAML
# 基于 LLVM 风格
BasedOnStyle: LLVM

# 缩进宽度
IndentWidth: 4

# 使用空格而不是 Tab
UseTab: Never
TabWidth: 4

# 控制空行
MaxEmptyLinesToKeep: 0

# 控制大括号的位置
# BreakBeforeBraces: Allman

BreakBeforeBraces: Attach

# 控制函数参数换行
AllowAllParametersOfDeclarationOnNextLine: false

# 控制注释对齐
AlignTrailingComments: true

# 控制指针和引号的放置
PointerAlignment: Left


# 控制访问修饰符后的空行
# EmptyLineAfterAccessModifier: true

```