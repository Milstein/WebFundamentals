---
layout: article
title: "图片优化"
description: "图片通常占据网页上下载字节的绝大部分，通常，也占据了大量的视觉空间。因此，优化图片通常可以最大程度地减少网站下载的字节数以及提高性能：浏览器下载的字节数越少，占用客户端的带宽就越少，浏览器下载并在屏幕上呈现有用内容的速度就越快。"
introduction: "图片通常占据网页上下载字节的绝大部分，通常，也占据了大量的视觉空间。因此，优化图片通常可以最大程度地减少网站下载的字节数以及提高性能：浏览器下载的字节数越少，占用客户端的带宽就越少，浏览器下载并在屏幕上呈现有用内容的速度就越快。"
article:
  written_on: 2014-05-07
  updated_on: 2014-05-10
  order: 3
collection: optimizing-content-efficiency
authors:
  - ilyagrigorik
key-takeaways:
  replace:
    - 删除不必要的图片资源
    - 尽可能利用 CSS3 效果
    - 使用网络字体取代在图片中进行文字编码
  vector-raster:
    - 矢量图最适用于由几何图形组成的图片
    - 矢量图与缩放和分辨率无关
    - 光栅图应该用于包含许多不规则形状和细节的复杂场景
  hidpi:
    - 高分辨率屏幕的每个 CSS 像素包含多个设备像素
    - 高分辨率图片所需的像素数和字节数要多得多
    - 任何分辨率都采用相同的图片优化方法
  optimizing-vector:
    - SVG 是基于 XML 的图片格式
    - SVG 文件应进行缩减，以减小文件大小
    - SVG 文件应使用 GZIP 进行压缩
  optimizing-raster:
    - 光栅图是像素栅格
    - 每个像素编码了颜色和透明度信息
    - 图片压缩程序使用不同的方法减少每个像素所需的位数，以减小图片的文件大小
  lossless-lossy:
    - 由于眼睛的工作方式，图片非常适用于有损压缩
    - 图片优化是进行有损压缩和无损压缩的一项功能
    - 图片格式的差异是由于优化图片时所使用的有损算法和无损算法的差异以及使用方式的差异
    - "不存在任何适用于所有图片的最佳格式或'质量设置'：每个特定压缩程序和图片内容的组合都产生独特的输出"
  formats:
    - "首先选择正确的通用格式：GIF、PNG、JPEG"
    - "体验并选择每种格式的最佳设置：质量、调色板大小等"
    - 考虑为较新的客户端添加 WebP 和 JPEG XR 资源 缩放的图片：
    - 提供缩放的资源是最简单、最有效的优化方法之一
    - 密切关注较大的资源，因为这些资源会产生大量额外开销
    - 通过将图片缩放到显示尺寸，减少不必要的像素数


notes:
  decompressed:
    - "简单提一句，无论将数据从服务器传输到客户端时使用哪种图片格式，在浏览器对图片进行解码时，每个像素始终占用 4 个字节的内存。对于较大的图片以及可用内存不足的设备（例如低端移动设备）来说，这可能是一个重要的约束条件。"
  artifacts:
    - "从左到右 (PNG)：32 位（16M 色）、7 位（128 色）、5 位（32 色）。包含渐变色过渡的复杂场景（渐变、天空等） 需要较大的调色板，以避免在 5 位资源中产生马赛克天空之类的视觉伪影。另一方面，如果图片只使用少量颜色，较大的调色板只会浪费珍贵的位数！"
  quality:
    - "注意，因为对图片进行编码所使用的算法不同，所以，不同图片格式的质量级别无法直接进行比较：质量为 90 的 JPEG 与质量为 90 的 WebP 效果截然不同。实际上，即使相同图片格式的质量级别，根据压缩程序的使用方式不同，视觉效果可能也会不同！"
  resized:
    - '在 Chrome DevTools 中将光标置于图片元素上，会显示该图片资源的"自然"尺寸和"显示"尺寸。在上面的例子中，下载的是 300 x 260 像素的图片，在客户端上显示时，降级为 245 x 212。'
---

{% wrap content%}

<style>
  img, video, object {
    max-width: 100%;
  }

  img.center {
    display: block;
    margin-left: auto;
    margin-right: auto;
  }
</style>

{% include modules/toc.liquid %}

图片优化既是一门艺术，也是一门科学：图片优化是一门艺术，是因为单个图片的压缩不存在最好的特定性方案，而图片优化之所以是一门科学，是因为许多开发得很出色的方法和算法可以明显减小图片的大小。要找到图片的最优设置，需要按照许多维度进行认真分析：格式能力、编码数据内容、像素尺寸等。

## 删除和替换图片

{% include modules/takeaway.liquid list=page.key-takeaways.replace %}

第一个要问您自己的问题应该是，要实现所需的效果，是否确实需要图片。好的设计应是简单的设计，而且始终可以提供最佳的性能。如果可以删除图片资源（与网页上的 HTML、CSS、 JavaScript 和其他资源相比，图片资源通常需要大量的字节），这始终是最佳的优化策略。俗话说，好的图片所传达的信息胜过千言万语，所以，您需要找到平衡点。

接下来，应考虑是否存在备选技术，可以通过更高效的方式实现所需的效果：

* **CSS 效果**（渐变、阴影等） 和 CSS 动画可以用于提供与分辨率无关的资源，任何分辨率和缩放级别都可以显示得非常清晰，而只需要图片文件所需字节数的一小部分。
* **网络字体**可以在保留选择文字、搜索文字以及调整文字大小的能力的同时，使用漂亮的字体 - 适用性大大提高。

如果您发现自己正在图片资源中对文字进行编码，不妨停下来重新考虑一下。出色的排版对好的设计、品牌塑造和可读性至关重要，但是，图片中的文字提供的用户体验较差：无法选择、搜索、缩放、访问文字，并且不适用于高 DPI 的设备。使用网络字体要求拥有 [自己的一组优化](https://www.igvita.com/2014/01/31/optimizing-web-font-rendering-performance/)，但是，可以解决所有上述问题，对于显示文字，始终是一个较好的选择。


## 矢量图与光栅图

{% include modules/takeaway.liquid list=page.key-takeaways.vector-raster %}

如果确定图片确实是实现所需效果的最优格式，接下来的关键选择就是选择合适的格式：

&nbsp;

<div class="clear">
  <div class="g--half">
    <b>矢量</b>
    <img class="center" src="images/vector-zoom.png" alt="放大的矢量图">
  </div>

  <div class="g--half g--last">
    <b>光栅</b>
    <img src="images/raster-zoom.png" alt="放大的光栅图">
  </div>
</div>

* [矢量图](http://en.wikipedia.org/wiki/Vector_graphics) 使用线、点和多边形来展示图片。
* [光栅图](http://en.wikipedia.org/wiki/Raster_graphics) 通过对矩形栅格内每个像素的值进行编码来展示图片。

每种格式都有自己的优缺点。矢量格式最适用于由简单的几何形状（例如徽标、文字、图标等）组成的图片，任何分辨率和缩放设置都可以提供清晰的效果，该格式最适用于需要以不同尺寸显示的高分辨率场景和资源。

但是，如果场景复杂（例如照片），矢量格式就达不到要求了：描述所有形状的 SVG 标记量可能异乎寻常的高，但是输出看起来仍可能没有`照片的真实感`。如果是这种情况，就应使用光栅图格式（例如 GIF、PNG、JPE 或 JPEG-XR 和 WebP 等某种较新的格式）。

光栅图没有与分辨率或缩放无关这么好的属性 - 在放大光栅图时，会看到粗糙和模糊的图片。因此，可能需要以不同分辨率保存多个版本的光栅图，以便为用户提供最优的体验。


## 高分辨率屏幕的含义

{% include modules/takeaway.liquid list=page.key-takeaways.hidpi %}

在谈到图片像素时，我们需要分清不同类型的像素：CSS 像素和设备像素。一个 CSS 像素可能包含多个设备像素 - 例如，一个 CSS 像素可能与一个设备像素直接对应，也可能包含多个设备像素。关键点是什么？ 那就是，设备像素越多，屏幕上所显示内容的细节就越精细。

<img src="images/css-vs-device-pixels.png" class="center" alt="CSS 像素与设备像素">

高 DPI (HiDPI) 屏幕可以产生绚丽的效果，但是也存在一个明显的权衡：图片资源需要具有更多细节，才能利用更高的设备像素数。好消息是，矢量图最适用于此任务，因为矢量图以任何分辨率都能呈现出清晰的效果 - 我们可能需要更高的处理成本才能呈现更精细的细节，但是，基础资源是相同的，并且与分辨率无关。

另一方面，因为光栅图是为每个像素编码图片数据，所以，面临的挑战会大得多。因此，像素数越多，光栅图的文件就越大。例如，让我们看一下以 100 x 100 (CSS) 像素显示的照片资源之间的差异：

<table class="table-3">
<colgroup><col span="1"><col span="1"><col span="1"></colgroup>
<thead>
  <tr>
    <th>屏幕分辨率</th>
    <th>总像素数</th>
    <th>未压缩的文件大小（每个像素 4 个字节）</th>
  </tr>
</thead>
<tbody>
<tr>
  <td data-th="分辨率"> 1x</td>
  <td data-th="总像素数"> 100 x 100 = 10,000</td>
  <td data-th="文件大小"> 40,000 个字节</td>
</tr>
<tr>
  <td data-th="分辨率"> 2x</td>
  <td data-th="总像素数"> 100 x 100 x 4 = 40,000</td>
  <td data-th="文件大小"> 160,000 个字节</td>
</tr>
<tr>
  <td data-th="分辨率"> 3x</td>
  <td data-th="总像素数"> 100 x 100 x 9 = 90,000</td>
  <td data-th="文件大小"> 360,000 个字节</td>
</tr>
</tbody>
</table>

如果物理屏幕的分辨率加倍，总像素数将增加四倍：水平像素数加倍，乘以垂直像素数加倍。因此，`2x`屏幕不仅是加倍，而是所需像素数的四倍！

那么，这实际上意味着什么？ 高分辨率屏幕可以显示绚丽的图片，这是一项出色的产品特征。但是，高分辨率屏幕也需要高分辨率图片：尽可能选择矢量图，因为矢量图与分辨率无关，始终可以实现清晰的效果，如果需要光栅图，应提供并优化每个图片的多个版本 - 后面将介绍更加详细的信息。


## 优化矢量图

{% include modules/takeaway.liquid list=page.key-takeaways.optimizing-vector %}

所有较新的浏览器都支持可缩放矢量图 (SVG)，SVG 是基于 XML 的图片格式，适用于二维图片：可以将 SVG 标记直接嵌入网页，也可以作为外部资源嵌入。然后，可以通过大多数基于矢量的绘图软件创建 SVG 文件，或直接在喜欢的文本编辑器中手动创建。

{% highlight xml %}
<?xml version="1.0" encoding="utf-8"?>
<!-- Generator: Adobe Illustrator 17.1.0, SVG Export Plug-In . SVG Version: 6.00 Build 0)  -->
<svg version="1.2" baseProfile="tiny" id="Layer_1" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink"
   x="0px" y="0px" viewBox="0 0 612 792" xml:space="preserve">
<g id="XMLID_1_">
  <g>
    <circle fill="red" stroke="black" stroke-width="2" stroke-miterlimit="10" cx="50" cy="50" r="40"/>
  </g>
</g>
</svg>
{% endhighlight %}

上面的例子呈现一个简单的圆形，轮廓为黑色，背景为红色，从 Adobe Illustrator 导出。正如您所知道的那样，其中包含大量元数据，例如图层信息、注释和 XML 名称空间，在浏览器中呈现资源时，通常不需要这些数据。因此，通过运行 [svgo](https://github.com/svg/svgo) 之类的工具缩减 SVG 文件，始终是个好主意。

当前的例子中，svgo 将 Illustrator 生成的上述 SVG 文件大小减小了 58%，从 470 字节缩减到 199 字节。另外，因为 SVG 是基于 XML 的格式，所以，还可以采用 GZIP 压缩来减小传输大小 - 确保将服务器配置为压缩 SVG 资源！


## 优化光栅图

{% include modules/takeaway.liquid list=page.key-takeaways.optimizing-raster %}

光栅图就是一个 2 维`像素`栅格 - 例如，100 x 100 像素的图片是 10,000 个像素的序列。然后，每个像素存储`[RGBA](http://en.wikipedia.org/wiki/RGBA_color_space)`值：(R) 红色通道、(G) 绿色通道、(B) 蓝色通道和 (A) alpha（透明度）通道。

在内部，浏览器为每个通道分配 256 个值（色阶），转换为每个通道 8 位 (2 ^ 8 = 256)，每个像素 4 个字节（4 个通道 x 8 位 = 32 位 = 4 个字节）。因此，如果我们知道栅格尺寸，可以很容易计算文件大小：

* 100 x 100 像素的图片由 10,000 个像素组成
* 10,000 个像素 x 4 个字节 = 40,000 个字节
* 40,000 个字节 / 1024 = 39 KB

^

{% include modules/remember.liquid title="Note" list=page.notes.decompressed %}

<table class="table-3">
<colgroup><col span="1"><col span="1"><col span="1"></colgroup>
<thead>
  <tr>
    <th>Dimensions</th>
    <th>像素</th>
    <th>文件大小</th>
  </tr>
</thead>
<tbody>
<tr>
  <td data-th="尺寸"> 100 x 100</td>
  <td data-th="像素"> 10,000</td>
  <td data-th="文件大小"> 39 KB</td>
</tr>
<tr>
  <td data-th="尺寸"> 200 x 200</td>
  <td data-th="像素"> 40,000</td>
  <td data-th="文件大小"> 156 KB</td>
</tr>
<tr>
  <td data-th="尺寸"> 300 x 300</td>
  <td data-th="像素"> 90,000</td>
  <td data-th="文件大小"> 351 KB</td>
</tr>
<tr>
  <td data-th="尺寸"> 500 x 500</td>
  <td data-th="像素"> 250,000</td>
  <td data-th="文件大小"> 977 KB</td>
</tr>
<tr>
  <td data-th="尺寸"> 800 x 800</td>
  <td data-th="像素"> 640,000</td>
  <td data-th="文件大小"> 2,500 KB</td>
</tr>
</tbody>
</table>

100 x 100 像素的图片为 39 KB，看起来好像不是个大问题，但是如果图片较大，文件会迅速增大，使图片资源的下载既缓慢成本又高。幸运的是，目前我们所描述的是`未压缩的`图片格式。我们该如何减小图片文件大小？

一个简单的策略就是将图片的`位深`从每个通道 8 位减小为更小的调色板：每个通道 8 位为每个通道提供 256 个值，共提供 16,777,216 (2563) 个颜色。如果我们将调色板减小为 256 色，会出现什么情况？ RGB 通道总共将只需要 8 位，每个像素立即节省两个字节 -- 将原来每个像素 4 个字节的格式压缩了 50%！

<img src="images/artifacts.png" class="center" alt="压缩伪影">

{% include modules/remember.liquid title="Note" list=page.notes.artifacts %}

接下来，在优化了各个像素中存储的数据之后，我们会变得更聪明，再观察一下相邻的像素：原来，许多图片（尤其是照片）的许多相邻像素具有类似的颜色 - 例如天空、重复的纹理等。利用这一信息，压缩程序可以采用`[增量编码](http://en.wikipedia.org/wiki/Delta_encoding)`，不必为每个像素分别存储值，可以存储相邻像素之间的差异：如果相邻像素相同，则增量为`零`，我们只需要存储一位！ 但是这样就够了吗？

人眼对不同颜色的敏感度也不同：为此，我们可以通过减小或增大这些颜色的调色板来优化颜色编码。
相邻像素构成二维栅格，这意味着每个像素有多个相邻像素：我们可以利用这一点来进一步改进增量编码。
不要只是关注每个像素直接相邻的像素，我们可以观察更大范围的相邻像素，使用不同的设置为不同的像素区进行编码。然后呢？

正如您所知道的那样，图片优化迅速变得更加复杂（根据角度的不同，您也可以认为是变得更加有趣），是学术研究和商业研究非常活跃的一个领域。图片占据了大量字节，开发更好的图片压缩方法具有很大的价值！ 如果您希望了解更多信息，请访问 [Wikipedia 网页](http://en.wikipedia.org/wiki/Image_compression)，或查看 [WebP 压缩方法白皮书](https://developers.google.com/speed/webp/docs/compression) 了解简单的示例。

所以，重申一下，这一领域非常有价值，但是也非常学术化：如何才能帮助我们优化网页上的图片？ 当然，我们不需要去发明新的压缩方法，但是，一定要了解该问题的基本概念：RGBA 像素、位深和各种优化方法。定要了解并记住所有这些概念，这样，我们才能进一步讨论不同的光栅图格式。


## 无损图片压缩与有损图片压缩

{% include modules/takeaway.liquid list=page.key-takeaways.lossless-lossy %}

对于某些类型的数据（例如网页的源代码或可执行文件），非常关键的一点是，压缩程序不更改或丢失任何原始信息：即使只有一位数据丢失或出错，也会完全改变文件内容的含义，甚至更糟的是，完全破坏这个文件。对于某些其他类型的数据（例如图片、音频和视频），只要提供的数据与原始数据`类似`，就可以很好的接受。

实际上，由于眼睛的工作方式，我们通常可以丢弃每个像素的某些信息，以便减小图片的文件大小 - 例如，眼睛对不同颜色的敏感度不同，这意味着我们可以使用较少的位数对某些颜色进行编码。因此，典型的图片优化过程由两个高级的步骤组成：

1. 使用`[有损](http://en.wikipedia.org/wiki/Lossy_compression)`过滤器处理图片，删除一些像素数据
1. 使用`[无损](http://en.wikipedia.org/wiki/Lossy_compression)`过滤器处理图片，压缩像素数据

** 第一步是可选步骤，准确的算法将取决于特定的图片格式，但是，一定要了解，任何图片都可以通过有损压缩步骤来减小文件大小。** 实际上，不同图片格式（例如 GIF、PNG、JPEG 以及其他格式）之间的差异在于在执行有损和无损步骤时，使用（或省略）的特定算法的组合。

那么，有损数据和无损优化的`最优`配置是什么？ 答案取决于图片内容以及您自己的标准（例如文件大小与有损压缩产生的伪影之间的权衡）：某些情况下，也许需要跳过有损优化，完全真实地传递复杂的细节，其他情况下，也许可以采用激进的有损优化来减小图片资源的文件大小。这取决于您自己的判断以及所处的环境 - 不存在任何通用的设置。

<img src="images/save-for-web.png" class="center" alt="存储为 Web 所用格式">

举个简单的例子，在使用 JPEG 等有损格式时，压缩程序通常会提供一个可定制的`质量`设置（例如 Adobe Photoshop 中的`存储为 Web 所用格式`功能提供的质量滑块），该设置通常是介于 1 和 100 之间的一个数字，用于控制特定的有损数据和无损算法组合的内部工作。为了获得最佳效果，体验图片的不同质量设置，不必担心质量下降 - 视觉效果通常非常好，而文件大小可能会减小很多。

{% include modules/remember.liquid title="Note" list=page.notes.quality %}


## 选择正确的图片格式

{% include modules/takeaway.liquid list=page.key-takeaways.formats %}

除了不同的有损压缩算法和无损压缩算法之外，不同的图片格式支持不同的特征（例如动画和透明度 (alpha) 通道）。因此，为特定图片选择`正确的格式`是所需的视觉效果要求和功能要求的组合。


<table class="table-4">
<colgroup><col span="1"><col span="1"><col span="1"><col span="1"></colgroup>
<thead>
  <tr>
    <th>格式</th>
    <th>透明度</th>
    <th>动画</th>
    <th>浏览器</th>
  </tr>
</thead>
<tbody>
<tr>
  <td data-th="格式"><a href="http://en.wikipedia.org/wiki/Graphics_Interchange_Format">GIF</a></td>
  <td data-th="透明度">支持</td>
  <td data-th="动画">支持</td>
  <td data-th="浏览器">所有</td>
</tr>
<tr>
  <td data-th="格式"><a href="http://en.wikipedia.org/wiki/Portable_Network_Graphics">PNG</a></td>
  <td data-th="透明度">支持</td>
  <td data-th="动画">不支持</td>
  <td data-th="浏览器">所有</td>
</tr>
<tr>
  <td data-th="格式"><a href="http://en.wikipedia.org/wiki/JPEG">JPEG</a></td>
  <td data-th="透明度">不支持</td>
  <td data-th="动画">不支持</td>
  <td data-th="浏览器">所有</td>
</tr>
<tr>
  <td data-th="格式"><a href="http://en.wikipedia.org/wiki/JPEG_XR">JPEG XR</a></td>
  <td data-th="透明度">支持</td>
  <td data-th="动画">支持</td>
  <td data-th="浏览器">IE</td>
</tr>
<tr>
  <td data-th="格式"><a href="http://en.wikipedia.org/wiki/WebP">WebP</a></td>
  <td data-th="透明度">支持</td>
  <td data-th="动画">支持</td>
  <td data-th="浏览器"> Chrome、Opera、Android</td>
</tr>
</tbody>
</table>

普遍支持的三种格式为：GIF、PNG 和 JPEG。除了这些格式之外，某些浏览器还支持 WebP 和 JPEG XR 等较新的格式，可以提供更好的整体压缩以及更多的功能。那么，应使用哪种格式？

<img src="images/format-tree.png" class="center" alt="存储为网络所用格式">

1. **是否需要动画？ 如果需要，GIF 是唯一的通用选择。**
  * GIF 限制调色板最多为 256 色，这对于大多数图片都不是一个好的选择。另外，PNG-8 通过小调色板为图片提供更好的压缩。因此，只有在需要动画时，GIF 才是正确的选择。
1. ** 是否需要使用最高的分辨率保留精细的细节？ 使用 PNG。**
  * 除了选择调色板尺寸，PNG 不采用任何有损压缩算法。因此，生成的图片质量最高，但代价是文件大小明显高于其他格式。应谨慎使用。
  * 如果图片资源包含由几何图形组成的图片，应考虑将其转化为矢量 (SVG) 格式！
  * 如果图片资源包含文字，应停下来重新考虑一下。图片中的文字无法选择、搜索或`缩放`。如果需要显示自定义的外观（因为品牌塑造或其他原因），则应使用网络字体。
1. **是否要优化照片、屏幕快照或类似的图片资源？ 使用 JPEG。**
  * JPEG 组合使用有损优化和无损优化来减小图片资源的文件大小。尝试几种 JPEG 质量级别，为图片资源找到质量和文件大小的最佳平衡点。

最后，为每个资源确定了最优图片格式及其设置之后，请考虑增加一个在 WebP 和 JPEG XR 中编码的版本。这两种格式都是新格式，不幸的是，这两种格式尚未获得所有浏览器的普遍支持，但是，可以为较新的客户端明显地节省资源 - 例如，平均来说，与可比的 JPEG 图片相比，WebP 可以 [使文件大小减少 30%](https://developers.google.com/speed/webp/docs/webp_study)。

因为 WebP 和 JPEG XR 均未获得普遍支持，所以，需要为应用或服务器增加一个逻辑，以提供相应的资源：

* 某些 CDN 的图片优化作为服务提供，包括提供 JPEG XR 和 WebP。
* 某些开放源代码的工具（例如 PageSpeed for Apache 或 PageSpeed for Nginx）自动优化、转换和提供相应的资源。
* 您可以增加应用逻辑来检测客户端，检查客户端支持的格式，并提供最适合的图片格式。

最后，请注意，如果使用 Webview 在原生应用中呈现内容，将可以完全控制客户端，可以独占使用 WebP！ Facebook、Google+ 以及许多其他应用都使用 WebP 提供应用内的所有图片 - 节省的资源当然物有所值。要了解 WebP 的详细信息，请观看 Google I/O 2013 的 [WebP：部署更快、更小、更绚丽的图片](https://www.youtube.com/watch?v=pS8udLMOOaE) 演讲。


## 工具和参数调优

不存在任何适用于所有图片的完美的图片格式、工具或优化参数集。为了获得最佳效果，必须根据图片内容及其视觉要求和其他技术要求，选择格式及其设置。

<table class="table-2">
<colgroup><col span="1"><col span="1"></colgroup>
<thead>
  <tr>
    <th>工具</th>
    <th>说明</th>
  </tr>
</thead>
<tbody>
<tr>
  <td data-th="工具"><a href="http://www.lcdf.org/gifsicle/">gifsicle</a></td>
  <td data-th="说明">创建和优化 GIF 图片</td>
</tr>
<tr>
  <td data-th="工具"><a href="http://jpegclub.org/jpegtran/">jpegtran</a></td>
  <td data-th="说明">优化 JPEG 图片</td>
</tr>
<tr>
  <td data-th="工具"><a href="http://optipng.sourceforge.net/">optipng</a></td>
  <td data-th="说明">无损 PNG 优化</td>
</tr>
<tr>
  <td data-th="工具"><a href="http://pngquant.org/">pngquant</a></td>
  <td data-th="说明">有损 PNG 优化</td>
</tr>
</tbody>
</table>


随意体验每个压缩程序的参数。降低质量，看看效果如何，然后取消重来。在发现了一组好的设置之后，可以对网站上的其他类似图片应用这些设置，但是，不要认为必须使用相同设置来压缩所有图片。


## 提供缩放的图片资源

{% include modules/takeaway.liquid list=page.key-takeaways.scaled-images %}

图片优化归结于两个标准：优化用于为每个图片像素进行编码的字节数以及优化像素总数：图片的文件大小就是像素总数乘以用于为每个像素进行编码的字节数。不多不少。

因此，最简单、最有效的一个图片优化方法就是，确保提供的像素恰好是在浏览器中以所需尺寸显示资源所需的像素。听起来很简单，是吗？ 不幸的是，大多数网页的许多图片资源都达不到这个要求：通常，会提供较大的资源，依靠浏览器进行重新缩放 - 这还会占用额外的 CPU 资源 - 并以较低的分辨率显示。

<img src="images/resized-image.png" class="center" alt="已调整尺寸的图片">

{% include modules/remember.liquid title="Note" list=page.notes.resized %}

提供不必要的像素会产生额外开销，而只是让浏览器代替我们重新缩放图片，这就错过了减少和优化呈现网页所需总字节数的绝好机会。另外，请注意，尺寸调整不仅是图片减少的像素数的函数，也是自然尺寸减少的像素数的函数。

<table class="table-3">
<colgroup><col span="1"><col span="1"><col span="1"></colgroup>
<thead>
  <tr>
    <th>自然尺寸</th>
    <th>显示尺寸</th>
    <th>不必要的像素数</th>
  </tr>
</thead>
<tbody>
<tr>
  <td data-th="自然"> 110 x 110</td>
  <td data-th="显示"> 100 x 100</td>
  <td data-th="额外开销"> 110 x 110 - 100 x 100 = 2100</td>
</tr>
<tr>
  <td data-th="自然"> 410 x 410</td>
  <td data-th="显示"> 400 x 400</td>
  <td data-th="额外开销"> 410 x 410 - 400 x 400 = 8100</td>
</tr>
<tr>
  <td data-th="自然"> 810 x 810</td>
  <td data-th="显示"> 800 x 800</td>
  <td data-th="额外开销"> 810 x 810 - 800 x 800 = 16100</td>
</tr>
</tbody>
</table>

请注意，在上述这三种情况下，显示尺寸只比图片的自然尺寸`小 10 个像素`。但是，自然尺寸越大，我们必须编码和提供的额外像素数就会明显增加！ 因此，尽管您也许无法保证每个资源都以精确的显示尺寸提供，但是，**应确保不必要的像素数最少，并确保特别是较大资产以尽可能接近显示尺寸的尺寸提供。**

## 图片优化检查表

图片优化既是一门艺术，也是一门科学：图片优化是一门艺术，是因为单个图片的压缩不存在最好的特定性方案，而图片优化之所以是一门科学，是因为开发得很出色的方法和算法可以明显减小图片的大小。

在优化图片时，要记住下列技巧和方法：

* **首选矢量格式：**矢量图与分辨率和缩放无关，最适用于多设备或高分辨率的情况。
* **缩减和压缩 SVG 资源：**大多数绘图应用程序生成的 XML 标记通常包含不必要的元数据，可以删除；确保服务器配置为对 SVG 资源采用 GZIP 压缩。
* **选择最佳光栅图格式：**确定功能要求，然后选择适合每个特定资源的格式。
* **体验光栅格式的最优质量设置：**随意降低`质量`设置，效果通常非常好，而字节可能会节省很多。
* **删除不必要的图片元数据：**许多光栅图包含不必要的资源元数据：地理信息、相机信息等。使用适合的工具删除这些数据。
* **提供缩放的图片：**调整服务器上的图片尺寸，确保`显示`尺寸尽可能接近图片的`自然`尺寸。尤其要密切关注较大的图片，因为这些图片在调整尺寸时，占用了最大的额外开销！
* **自动、自动、自动：**投资自动化的工具和基础设施，这样，可以确保所有图片资源始终会进行优化。


{% include modules/nextarticle.liquid %}

{% endwrap %}

