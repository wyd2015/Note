---
title: 'vscode常用配置'
Date: 2019-04-12 10:22:52
Tag: vscode
---
## jsconfig.json
在项目根目录下添加该文件，实现“智能跳转到定义处”的功能。
文件配置如下，详细配置可参照[官方文档](https://code.visualstudio.com/docs/languages/jsconfig)：
```json
{
  "compilerOptions": {
    "baseUrl": "src/",
    "target": "esnext",
    "module": "commonjs"
  },
  "exclude": [
    "node_modules",
  ]
}
```
## 隐藏 [OPEN EDITORS] 窗格
```json
 "explorer.openEditors.visible": 0
```

## 标题栏显示文件归属
```json
"window.title": "${dirty} ${activeEditorMedium}"
```
- `${dirty}`: 当文件修改后未保存，显示一个圆点;
- `${activeEditorMedium}`: 当前文件相对于工作区文件夹的路径;
- `${separator}`: 条件分隔符（"-"），仅当被带有值或静态文本的变量包围时才显示;
- `${rootName}`: 工作区名称

## 显示所有字符
```json
"editor.renderWhitespace": "all"
```

## 插入符号改为竖线
```json
"editor.cursorBlinking": "phase"
```

## 文件末尾添加空行
```json
"files.insertFinalNewline": true
```

## 删除尾部空格
```json
"files.trimTrailingWhitespace": true
```

## 禁止向微软上传使用数据
```json
"telemetry.enableTelemetry": false,
"telemetry.enableCrashReporter": false
```