# 桌面壁纸配置

## 环境

- 操作系统：Ubuntu
- 系统版本：16.04.3 LTS
- 桌面环境：Unity

## 配置

1. 下载喜欢的壁纸，如 *myBackground.png*；

2. 拷贝壁纸到系统默认路径：`sudo cp myBackground.png /usr/share/backgrounds/`；

3. 此时打开到系统设置里面去修改壁纸，还是看不到 *myBackground.png*，别急，继续下面的步骤；

4. 编辑文件：`sudo vim /usr/share/gnome-background-properties/xenial-wallpapers.xml`；

5. 这是一个 *XML* 文件，我们在 `<wallpapers></wallpapers>` 节点中间增加如下内容：

   ```xml
   <wallpaper>
       <name>myBackground</name>
       <filename>/usr/share/backgrounds/myBackground.png</filename>
       <options>zoom</options>
       <pcolor>#000000</pcolor>
       <scolor>#000000</scolor>
       <shade_type>solid</shade_type>
   </wallpaper>
   ```

6. 保存并退出编辑；

7. 此时打开到系统设置里面去修改壁纸，就能看到 *myBackground.png* 了，选中它，Then enjoy yourself。

