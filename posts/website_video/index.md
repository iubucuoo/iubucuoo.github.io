# hugo在网页中添加视频


<!--more-->


在  `md`  文件中嵌入  `ShortCodes`  时，其查找顺序如下：

1.  `/layouts/shortcodes/<SHORTCODE>.html`

3.  `/themes/<THEME>/layouts/shortcodes/<SHORTCODE>.html`

即优先查找你的项目的根目录下的  `/layouts/shortcodes`  文件夹下的模板文件，再查找  `theme`  文件夹下的  `/layouts/shortcodes`  文件夹。

## 1. 创建  `html`  模板
我们在 `/layouts/shortcodes/` 目录下创建一个 `bilibili.html` 并粘贴下面的代码：
```html
<!DOCTYPE HTML>
<html>

  <head>
    <!-- style 样式 是为了让网页上的视频框按比例显示而非固定的大小 -->
    <style type="text/css">
      .aspect-ratio {
        position: relative;
        width: 100%;
        height: 0;
        padding-bottom: 75%;
      }

      .aspect-ratio iframe {
        position: absolute;
        width: 100%;
        height: 100%;
        left: 0;
        top: 0;
      }
    </style>
  </head>

  <body>
    <div class="aspect-ratio">
      <iframe
              src="https://player.bilibili.com/player.html?bvid={{.Get 0 }}&page={{ if .Get 1 }}{{.Get 1}}{{ else }}1&high_quality=1&danmaku=0{{end}}"
              scrolling="no" 
              border="0" 
              frameborder="no" 
              framespacing="0" 
              allowfullscreen="true"
              sandbox="allow-top-navigation allow-same-origin allow-forms allow-scripts"
              >
      </iframe>
      <!-- src 中的 &high_quality=1&danmaku=0 设定了高清程度并默认屏蔽弹幕 -->
      <!-- sandbox 阻止了点击视频中的按钮跳转到B站的行为 -->
    </div>
  </body>

</html>
```
### 关于B站链接的参数介绍

B站的链接有点薛定谔，不同的视频显示的链接参数不尽相同，但是总的来说有以下类型：
| key | 说明 |
|--|--|
|  cid| chat id，每个 chat id 对应一组弹幕池如填了 aid ，这个字段不填也没关系 |
| aid| article id ，视频的 av 号就是B站的 avxxxx 后面的数字|
|  bvid| bilibili video id ，视频的 bv 号2020.03 时候，B战把 av 号根据一定的算法转成这个了如果填了 bvid ，那么 aid 不填也可以 |
| page | 第几个视频，起始下标为 1 （默认值也是为1）就是B站视频，选集里的第几个视频 |
|as_wide|是否宽屏1: 宽屏，0: 小屏|
| high_quality | 是否高清1: 高清，0: 最低视频质量（默认）如视频有 360P 720P 1080P 三种，默认或者 high_quality=0 是最低 360P high_quality=1 是最高1080P |
|danmaku|是否开启弹幕1: 开启（默认），0: 关闭|
| t | 打开时，自动跳转到某个时间，并且自动播放（单位秒）比如 t=60 ，那么自动跳转到1分钟，且自动播放 | 





## 2. 嵌入B站视频

在此 html 中，我设定的  `src`  的传入参数是  `bvid`  和  `page`  ，其中 page 是视频的指定视频选集 ( 对于多集视频可以自定义传参，默认为  `1`  ) 。

在 markdown 中需要嵌入视频的位置使用以下形式即可：

``` python
{{</* bilibili BV1sz4y197L8 */>}}
```

对于有多集的视频，我想指定第 10 集，则是：

```python
{{</* bilibili BV1sz4y197L8 10 */>}}
```
## 3. 嵌入 YouTube 视频

Hugo 貌似支持直接使用  `YouTube`  的视频嵌入，因为在我的博客的项目工程中我没有找到相关的 html 模板。YouTube 的网站链接做的相对友好：https://www.youtube.com/watch?v=XXXXXXXXXXX ，只需要把  `v=`  后面的内容复制到  `ShortCodes`  中即可，嵌入方式如下：

```python
{{</* youtube WNeLUngb-Xg */>}}
```

如果你想要自动播放，可以将其参数设置为  `true`  实现。由于我们不能将命名和未命名的参数混在一起使用，因此需要将尚未命名的视频 ID 分配给参数  `id`  ：

```python
{{</* youtube id="w7Ft2ymGmfc" autoplay="true" */>}}
```
---
‍如果你的网站中无法直接嵌入  `YouTube`  的视频，那么如法炮制，在  `/layouts/shortcodes/`  目录下创建一个  `youbube.html`  并粘贴下面的代码：

```html
<div class="embed video-player">
  <iframe 
          class="youtube-player" 
          type="text/html" 
          width="640" 
          height="385"
          src="https://www.youtube.com/embed/{{ index .Params 0 }}?autoplay=1"
          style="
                 position: absolute; 
                 top: 0; 
                 left: 0; 
                 width: 100%; 
                 height: 100%; 
                 border:0;"
          allowfullscreen frameborder="0">
  </iframe>
</div>
<!-- autoplay 也可以设置为 0 禁止自动播放，但是在 Safari 中无论如何设置都无法自动播放 -->

```

然后再按照以上的  `ShortCodes`  进行嵌入即可。

或者你也可以通过直接在  `markdown`  中输入以下内容实现视频插入 ( 如果你不是经常需要嵌入视频的话 ) ：

```html
<div class="embed video-player" 
     style="position: relative; padding-bottom: 56.25%; height: 0; overflow: hidden;">
  <iframe class="youtube-player"
          type="text/html"
          width="640" 
          height="385"
          src="https://www.youtube.com/embed/WNeLUngb-Xg?autoplay=1"
          style="
                 position: absolute; 
                 top: 0; 
                 left: 0; 
                 width: 100%; 
                 height: 100%; 
                 border:0;"
          allowfullscreen frameborder="0">
  </iframe>
</div>

```
## 原文与参考链接

原文链接： [原文链接](https://caymanhk.gitee.io/posts/006_hugo通过shortcodes添加视频)

---

参考链接：
 1. [Create Your Own Shortcodes](https://gohugo.io/templates/shortcode-templates/)

 2. [Shortcodes Of YouTube](https://gohugo.io/content-management/shortcodes/#youtube)

