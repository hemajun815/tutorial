# 修改用户登录界面

## 环境

- 操作系统：Ubuntu
- 系统版本：16.04.3 LTS
- 桌面环境：Unity

## 原理

在 */usr/share/glib-2.0/schemas/* 中创建一个名为 *50_unity-greeter.gschema.override* 的文件来定义各项值，达到覆盖默认值的效果

## 配置

创建文件：`sudo vim /usr/share/glib-2.0/schemas/50_unity-greeter.gschema.override`

写入第一行数据：

```
[com.canonical.unity-greeter]
```

下面的步骤并非都需要，按需执行。

**修改登录界面背景图片**

代码如下：

```js
background = "图片路径"
```

其默认值为

```js
background = "/usr/share/backgrounds/warty-final-ubuntu.png"
```

因此建议，将喜欢的图片拷贝到 */usr/share/backgrounds/* 目录下，然后再对图片路径进行修改。

**禁止用户桌面背景**

需要注意的是，刚刚定义的背景图片优先级默认低于当前正在登录用户的桌面背景图片，实际效果是该图片一闪而过后被当前正在尝试登录的用户的桌面背景图片覆盖。如果将尝试登录的用户切换至Guest，该图片才会出现。这里我们可以通过禁止显示任何用户的自定义桌面背景来实现只显示第一条中定义的登录背景。代码如下：

```js
draw-user-backgrounds = false
```

**修改登录界面底色**

然而即便修改了登录界面背景图片，每次系统加载进入登录界面的一瞬间，还是会出现默认颜色背景。修改背景颜色的代码是：

```js
background-color = "#000000"
```

**禁用登录声音**

代码如下：

```js
play-ready-sound = false
```

**去除背景点状网格**

登录界面的背景上会默认铺一层点状网纹，去除代码如下：

```js
draw-grid = false
```

**修改左下角Ubuntu logo**

修改的代码如下：

```js
logo = 'Logo路径'
```

其默认值为：

```js
logo = "/usr/share/unity-greeter/logo.png"
```

同样，建议将喜欢的图片拷贝到 */usr/share/unity-greeter/* 目录下，再做修改。

**取消显示主机名称**

登录界面左上角会默认显示本主机的主机名，取消代码如下：

```js
show-hostname = false
```

**修改登录界面主题**

代码如下：

```js
theme-name = "Flatabulous"
```

安装 *Flatabulous* 主题可参考[这里](./installing-flat-theme:Flatabulous.md)

上面几项修改按需执行，最终的 *50_unity-greeter.gschema.override* 文件大致内容如下：

```js
[com.canonical.unity-greeter]
background = "/usr/share/backgrounds/coder.jpg"
draw-user-backgrounds = false
background-color = "#000000"
play-ready-sound = false
draw-grid = false
show-hostname = false
theme-name = "Flatabulous"
```

保存文件并退出。

刷新schemas：`sudo glib-compile-schemas /usr/share/glib-2.0/schemas/`。

重启 lightdm 服务：`sudo service lightdm restart`。