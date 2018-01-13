# 禁用Win10 Cortana
## 环境
- 系统：Windows 10 Enterprise
- 版本：1709
- 处理器：Intel(R) Core(TM) i7-7700K CPU @ 4.20GHz
- 内存：16.0GB
- 类型：64位操作系统 64位处理器
## 说明
通过修改注册表项来达到禁用Cortana的目的，修改注册表前强烈建议备份注册表项。
## 步骤
1. 使用`win+R`组合键打开"Run"对话框；
2. 键入`regedit`回车打开"Registry Editor"；
3. 依次导航进入"Computer\HKEY_LOCAL_MACHINE\SOFTWARE\Policies\Microsoft\Windows\Windows Search"；
4. 在右侧窗格中点击右键，选择"New - DWORD (32-bit) Value"，然后把新建的值命名为AllowCortana，数值数据按默认的0即可;
5. 修改完成后，关闭"Registry Editor"，注销后重新登录生效；
6. 若要重新启用Cortana，将"AllowCortana"的值设置为1或者删除AllowCortana项，注销后重新登录即可。
