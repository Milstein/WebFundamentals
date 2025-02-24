---
layout: article
title: "优化基于文本的资产的编码和传输大小"
description: "在我们消除了任何不必要的资源之后，下一步就是将浏览器必须下载的剩余资源的总大小减至最小，即通过应用特定于内容类型的和通用的压缩 (GZip) 算法压缩这些资源。"
introduction: "我们的网络应用在范围、目标和功能上都在不断增长。这是件好事！ 但是向着更丰富的网络无情进军的过程也推动了另一种趋势：每个应用所需下载的数据量也在持续稳步增长。为了提供卓越的性能，我们需要优化每一个字节数据的交付！"
article:
  written_on: 2014-04-01
  updated_on: 2014-09-12
  order: 2
collection: optimizing-content-efficiency
authors:
  - ilyagrigorik
key-takeaways:
  compression-101:
    - 压缩就是使用更少的位对信息进行编码的过程
    - 消除不必要的数据总是会产生最好的结果
    - 有许多种不同的压缩技术和算法
    - 您将需要各种各样的技术来实现最佳压缩
  minification:
    - 特定于内容的优化可以显著减小交付的资源的大小。
    - 特定于内容的优化最好作为您的内部版本/发行版本周期的一部分进行应用。
  text-compression:
    - "GZIP 对于基于文本的资产的压缩效果最好：CSS、JavaScript、HTML"
    - 所有现代浏览器都支持 GZIP 压缩并将自动请求该压缩
    - 您的服务器需要配置为支持 GZIP 压缩
    - 某些 CDN 需要特别小心来确保 GZIP 已启用
notes:
  jquery-minify:
    - "典型的例证是，JQuery 库的未压缩开发版本现在即将达到 ~300KB。相同的但经过压缩（删除了注释等）的库 大约为三分之一大：~100KB。"
  gzip:
    - "无论您相信与否，会存在 GZIP 增加资产大小的情形。通常，在以下情况下会出现这种情形，当资产非常小且 GZIP 字典大于压缩节省的字节数时，或者当资源已得到很好的压缩时。某些服务器允许您指定一个`最小文件大小阈值`来避免此问题。"
---

{% wrap content%}

{% include modules/toc.liquid %}

<style>
  img, video, object {
    max-width: 100%;
  }

  img.center {
    display: block;
    margin-left: auto;
    margin-right: auto;
  }

  pre {
    background-color: #f0f0f0;
    padding: 1em 3em;
  }
</style>

## 数据压缩 101

在我们消除了任何不必要的资源之后，下一步就是将浏览器必须下载的剩余资源的总大小减至最小，即，对其进行压缩。根据资源类型（文本、图片、字体等），我们有许多不同的技术供我们使用：可以在服务器上启用的通用工具，适用于特定内容类型的预处理优化，以及需要开发人员输入的特定于资源的优化。

要实现最佳的性能需要结合所有这些技术。

{% include modules/takeaway.liquid list=page.key-takeaways.compression-101 %}

减小数据大小的过程称为`数据压缩`，而这本身就是一个需要深入研究的领域：许多人终其一生致力于算法、技术和优化以提高各种压缩工具的压缩比率、速度和内存要求。勿需多言，对本主题进行全面讨论超出了我们的范围，但是，在一个较高的水平上了解压缩的工作方式以及用于减小网页所需的各种资源的大小所能使用的技术，这仍是很重要的。

为了说明这些要推广的技术的核心原则，让我们看看如何能够优化一个简单的短信格式（此格式仅仅是我们为此示例所编造的）：

    # 下面是一条秘密消息，它由一组标题组成
    # 这组标题采用后面跟着一个新行和加密消息的键-值格式。
    format: secret-cipher
    date: 04/04/14
    AAAZZBBBBEEEMMM EEETTTAAA

1. 消息可能会包含任意注释，这些注释带有`#`前缀。注释不影响消息的含义或消息的任何其他行为。
2. 消息可能会包含`标题`，这些标题是一些键-值对（用`:`分隔），且必须出现在消息的开头。
3. 消息带有文本载荷。

我们能做些什么来减小上述消息的大小（这条消息的长度目前为 200 个字符）？

1. 嗯，注释很有趣，但我们知道它并不实际影响消息的含义，因此我们在传送消息时会清除注释。
2. 可能会有一些可用于以一种有效的方式对标题进行编码的智能技术，例如，我们不知道是否所有消息始终都有`格式`和`日期`，但如果有，我们可以将它们转换为短整 ID 并仅发送这些 ID！ 这就是说，我们不确信是否是这种情况，因此我们现在先不去管它。
3. 载荷是仅文本，而我们不知道其内容真正是什么（很明显，载荷使用一种`秘密消息`），只是看看文本，看起来在其中存在许多冗余。也许，不发送重复的字母，我们可以仅对重复字母的数量进行计数，并更有效地对其进行编码，是这样吗？
    * 例如，`AAA`变为`3A`，或者三个 A 的序列。


结合我们的技术，我们实现了下面的结果：

    format: secret-cipher
    date: 04/04/14
    3A2Z4B3E3M 3E3T3A

新消息的长度为 56 个字符，这意味着我们勉强做到将我们的原始消息压缩达到了可观的 72% - 这个结果不坏，总而言之，我们仅仅开了个头！

当然，您也许会怀疑，这已经非常好了，但这如何能帮助我们优化我们的网页呢？ 无疑，我们不打算尝试编造我们的压缩算法，不是吗？ 答案是不打算，我们不会，但正像您将看到的那样，优化我们网页上的各种资源时，我们将使用几乎相同的技术和思考方式：预处理，特定于内容的优化，以及不同的内容采用不同的算法。


## 源码压缩：预处理和特定于内容的优化

{% include modules/takeaway.liquid list=page.key-takeaways.minification %}

压缩冗余或不必要的数据的最佳方式是全部消除它。当然，我们不能只是删除任意数据，但在某些情况下，我们可能会有数据格式及其属性的特定于内容的知识，这时经常有可能显著减小载荷的大小而不影响其实际含义。

{% include_code _code/minify.html full %}

考虑以上的简单 HTML 网页和它所包含的三种不同的内容类型：HTML 标记、CSS 样式和 JavaScript。其中每种内容类型针对构成有效 HTML 标记、CSS 规则和 JavaScript 内容的元素都有不同的规则，有用于指示注释的不同规则，等等。我们如何可以减小此网页的大小？

* 代码注释是开发人员的最好朋友，但浏览器不需要看到它们! 简单地除去 CSS (`/* ... */`)、HTML (`<!-- ... -->`) 和 JavaScript (`// ...`) 注释可以显著减小网页的总大小。
* `智能`CSS 压缩工具可以察觉到我们正在使用一种无效方式为`.awesome-container`定义规则，并将两个声明折叠为一个而不影响任何其他样式，节省更多字节。
* 在 HTML、CSS 和 JavaScript 中，白空（空格和制表符）仅仅是为了开发人员方便。有一种附加的压缩工具可以去掉所有制表符和空格。

^
{% include_code _code/minified.html full %}

在应用上述步骤之后，我们的网页从 406 个字符变为 150 个字符 - 达到 63% 的压缩节省！ 确实，其可读性不是很好，但也并非必须这样：我们可以保留原始网页作为我们的`开发版本`，然后并在我们准备好在我们的网站上发布该网页时再应用上述步骤。

退一步说，上述示例表明很重要的一点是：一个常规用途压缩工具（意即一个旨在压缩任意文本的压缩工具）也可能会很好地压缩上述网页，但它从来不会知道除去注释，折叠 CSS 规则，或进行许多其他特定于内容的优化。这就是预处理/源码压缩/上下文感知优化可以是如此强大的一个工具的原因所在。

{% include modules/remember.liquid list=page.notes.jquery-minify %}

同样地，可以将上述技术扩展到超越仅基于文本的资产。图片、视频和其他内容类型都包含它们自己的元数据形式和各种载荷。例如，任何时候使用相机拍摄照片，照片通常也会嵌入许多额外的信息：相机设置、位置等。根据您的应用，此数据可能会很重要（例如，一个照片共享网站），或者完全无用，您应该考虑是否应删除它。实际上，对于每张图片，此元数据加起来可以达到数十 KB！

简而言之，作为优化您的资产效率的第一步，请构建不同内容类型的资源，并考虑您可以应用哪些种类的特定于内容的优化以减小其大小，这样做可以产生显著的节省！ 然后，在您确定它们是什么之后，请立即通过将其添加到您的内部版本和发行版本流程来自动执行这些优化 - 这是您可以保证优化将保持原样的唯一方式。

## 使用 GZIP 进行文本压缩

{% include modules/takeaway.liquid list=page.key-takeaways.text-compression %}

[GZIP](http://en.wikipedia.org/Wiki//Gzip) 是一种可以应用于任何字节流的通用压缩工具：启动之后，该工具会记住其中一些先前看到的内容并尝试以一种有效的方式查找并替换重复的数据片段 - 好奇的用户可以查阅 [GZIP 的出色的低层次解释] (https://www.youtube.com/watch?v=whGwm0Lky2s&feature=youtu.be&t=14m11s)。但是，实际上，GZIP 对基于文本的内容压缩效果最好，对于较大的文件，通常会达到高达 70-90% 的压缩率，而通过其他替代算法对已压缩过的资产（例如，大多数图片格式）运行 GZIP 则收效甚微，甚至没有任何提高。

所有现代浏览器都支持并自动商定将 GZIP 压缩用于所有 HTTP 请求：我们的工作是确保服务器得到正确配置，以在客户端请求时提供压缩的资源。


<table class="table-4">
<colgroup><col span="1"><col span="1"><col span="1"><col span="1"></colgroup>
<thead>
  <tr>
    <th>库</th>
    <th>大小</th>
    <th>压缩后的大小</th>
    <th>压缩率</th>
  </tr>
</thead>
<tbody>
<tr>
  <td data-th="库">jquery-1.11.0.js</td>
  <td data-th="大小">276 KB</td>
  <td data-th="压缩后的大小">82 KB</td>
  <td data-th="节省">70%</td>
</tr>
<tr>
  <td data-th="库">jquery-1.11.0.min.js</td>
  <td data-th="大小">94 KB</td>
  <td data-th="压缩后的大小">33 KB</td>
  <td data-th="节省">65%</td>
</tr>
<tr>
  <td data-th="库">angular-1.2.15.js</td>
  <td data-th="大小">729 KB</td>
  <td data-th="压缩后的大小">182 KB</td>
  <td data-th="节省">75%</td>
</tr>
<tr>
  <td data-th="库">angular-1.2.15.min.js</td>
  <td data-th="大小">101 KB</td>
  <td data-th="压缩后的大小">37 KB</td>
  <td data-th="节省">63%</td>
</tr>
<tr>
  <td data-th="库">bootstrap-3.1.1.css</td>
  <td data-th="大小">118 KB</td>
  <td data-th="压缩后的大小">18 KB</td>
  <td data-th="节省">85%</td>
</tr>
<tr>
  <td data-th="库">bootstrap-3.1.1.min.css</td>
  <td data-th="大小">98 KB</td>
  <td data-th="压缩后的大小">17 KB</td>
  <td data-th="节省">83%</td>
</tr>
<tr>
  <td data-th="库">foundation-5.css</td>
  <td data-th="大小">186 KB</td>
  <td data-th="压缩后的大小">22 KB</td>
  <td data-th="节省">88%</td>
</tr>
<tr>
  <td data-th="库">foundation-5.min.css</td>
  <td data-th="大小">146 KB</td>
  <td data-th="压缩后的大小">18 KB</td>
  <td data-th="节省">88%</td>
</tr>
</tbody>
</table>

上述表格阐明了对于许多最流行的 JavaScript 库和 CSS 框架，GZIP 所提供的节省。节省的范围从 60% 到 88%，并注意缩小的文件的组合（在其文件名中用`.min`来标识），加上 GZIP，可提供甚至更大的节省。

1. **首先应用特定于内容的优化：CSS、JS 和 HTML 压缩机。**
2. **应用 GZIP 以压缩缩小的输出。**

最好的是，启用 GZIP 是其中一种实施起来最简单并且回报最高的优化 - 遗憾的是，许多人仍忘记实施它。大多数网络服务器将代表您压缩内容，而您仅需要确认服务器已正确配置，可用于压缩将从 GZIP 压缩受益的所有内容类型。

您的服务器的最佳配置是什么样子的？ HTML5 样板项目包含所有最流行的服务器的 [示例配置文件](https://github.com/h5bp/server-configs)，每个配置标志和设置都有详细的注释：在列表中找到您喜爱的服务器，查找 GZIP 部分，并确认您的服务器已配置建议的设置。

<img src="images/transfer-vs-actual-size.png" class="center" alt="DevTools 实际演示与传输大小">

查看要推广的 GZIP 最快速、简单的方法是打开 Chrome DevTools 并检查`网络`面板中的`大小/ 内容`列 ：`大小`指示资产的传输大小，而`内容`指示资产的未压缩大小。对于上述示例中的 HTML 资产，GZIP 在传输过程中节省了 24.8 KB！

{% include modules/remember.liquid list=page.notes.gzip %}

最后，用一句话作为提醒：尽管大多数服务器将资产提供给用户时会自动为您压缩资产，但是某些 CDN 需要特别小心，并需要手动操作以确保已提供 GZIP 资产。审核您的网站并确保您的资产实际上 [已进行压缩](http://www.whatsmyip.org/http-compression-test/)！



{% include modules/nextarticle.liquid %}

{% endwrap %}

