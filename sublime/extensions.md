# 扩展

### Package Control

Package Control 是一个方便 Sublime text 管理扩展的扩展。

**安装**

1. 快捷键 "ctrl+`" 或者菜单 "View > Show Console" 打开控制台；

2. 复制粘贴执行以下代码：

   ```
   import urllib.request,os,hashlib; h = '6f4c264a24d933ce70df5dedcf1dcaee' + 'ebe013ee18cced0ef93d5f746d80ef60'; pf = 'Package Control.sublime-package'; ipp = sublime.installed_packages_path(); urllib.request.install_opener( urllib.request.build_opener( urllib.request.ProxyHandler()) ); by = urllib.request.urlopen( 'http://packagecontrol.io/' + pf.replace(' ', '%20')).read(); dh = hashlib.sha256(by).hexdigest(); print('Error validating download (got %s instead of %s), please try manual install' % (dh, h)) if dh != h else open(os.path.join( ipp, pf), 'wb' ).write(by)
   ```

3. 待执行完成， "Package Control" 就安装完成了。


### A File Icon

为侧边栏文件添加特定的图标以提升视觉体验。设计来源参照于 Atom 文件图标

**安装**

1. 快捷键 "ctrl+shift+p" 或者菜单 "Tools > Command Palette..." 打开 Command Palette ；
2. 选择 "Package Control: Install Package" ；
3. 找到 "A File Icon" 然后敲击回车/ Enter 键；
4. 待执行完成， "A File Icon" 就安装完成了。


### Anaconda

Anaconda 是一个可以将 Sublime Text3 变成 Python IDE 的插件，使用它，可以提高你的工作效率和质量。

**安装**

1. 快捷键 "ctrl+shift+p" 或者菜单 "Tools > Command Palette..." 打开 Command Palette ；
2. 选择 "Package Control: Install Package" ；
3. 找到 "Anaconda" 然后敲击回车/ Enter 键；
4. 待执行完成， "Anaconda" 就安装完成了。

### Terminal

Terminal 是一个允许你从当前目录或当前项目根目录启动终端的插件。

**安装**

1. 快捷键 "ctrl+shift+p" 或者菜单 "Tools > Command Palette..." 打开 Command Palette ；
2. 选择 "Package Control: Install Package" ；
3. 找到 "Terminal" 然后敲击回车/ Enter 键；
4. 待执行完成， "Terminal" 就安装完成了。