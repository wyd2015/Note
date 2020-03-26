## 报错内容
```yml
Could not open/create prefs root node Software\JavaSoft\Prefs atroot 0x80000002. Windows RegCreateKeyEx(...) returned error code 5
```
## 解决办法
- 命令行窗口输入 regedit，打开注册表编辑;
- 如果 `HKEY_LOCAL_MACHINE\Software\JavaSoft\Prefs` 不存在，新建并赋予权限（完全控制），如果 `HKEY_LOCAL_MACHINE\Software\JavaSoft\Prefs` 存在，直接赋予最高权限（完全控制）