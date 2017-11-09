# 修改Windows窗口尺寸
## 环境
- 系统：Windows 10 Enterprise
- 版本：1703
- 处理器：Intel(R) Core(TM) i7-7700K CPU @ 4.20GHz
- 内存：16.0GB
- 类型：64位操作系统 64位处理器
## 说明
通过修改注册表项来达到修改Windows窗口尺寸的目的，修改注册表前强烈建议备份注册表项。
## 步骤
1. 使用`win+R`组合键打开"Run"对话框；
2. 键入`regedit`回车打开"Registry Editor"；
3. 依次导航进入"Computer\HKEY_CURRENT_USER\Control Panel\Desktop\WindowMetrics"；
4. 按照实际需要修改以下表项（每项的值 = 实际像素值 × (-15)）：
    - CaptionHeight：最大化模式下窗口标题栏高度；
    - CaptionWidth：最大化模式下窗口标题栏宽度；
    - MenuHeight：菜单项高度；
    - MenuWidth：菜单项宽度；
    - ScrollHeight：滚动条高度；
    - ScrollWidth：滚动条宽度；
    - SmCaptionHeight：窗口模式下窗口标题栏高度；
    - SmCaptionWidth：窗口模式下窗口标题栏高度；
5. 修改完成后，关闭"Registry Editor"，注销后重新登录生效。
