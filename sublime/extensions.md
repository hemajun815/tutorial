# 扩展

### Package Control

Package Control 是一个方便 Sublime text 管理扩展的扩展。

**安装**

1. 快捷键 "ctrl+`" 或者菜单 "View > Show Console" 打开控制台；

2. 复制粘贴执行以下代码：

   ```
   import urllib.request,os,hashlib; h = '6f4c264a24d933ce70df5dedcf1dcaee' + 'ebe013ee18cced0ef93d5f746d80ef60'; pf = 'Package Control.sublime-package'; ipp = sublime.installed_packages_path(); urllib.request.install_opener( urllib.request.build_opener( urllib.request.ProxyHandler()) ); by = urllib.request.urlopen( 'http://packagecontrol.io/' + pf.replace(' ', '%20')).read(); dh = hashlib.sha256(by).hexdigest(); print('Error validating download (got %s instead of %s), please try manual install' % (dh, h)) if dh != h else open(os.path.join( ipp, pf), 'wb' ).write(by)
   ```

3. 待执行完成， Package Control 就安装完成了。

