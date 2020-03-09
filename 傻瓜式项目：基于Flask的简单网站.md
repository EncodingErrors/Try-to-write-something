## 傻瓜式项目：基于Flask的简单网站

### 前言

在学习的过程中，有了尝试建一个网站的想法，当时在玩[明日方舟](https://ak.hypergryph.com/index)，招募干员的时候也需要用到[公开招募计算器](https://ak.graueneko.xyz/)，所以打算自己写一个类似的网站。

由于没有艺术细菌，写这个网站所用的素材都是未经授权的官方数据资源，为了避免不必要的麻烦，这个网站没有对外发布，纯粹的个人练手作品，这里仅记录开发过程。

感觉自己对Python还比较熟练，所以使用flask框架写后端。然而前端几乎没学过学，只在写爬虫的时候了解过html文档，所以写前端的过程很繁琐（大部分时间耗费在css上，页面设计真的熬死理工狗，搞艺术的在我眼里都是神）。

为了让部分工具能正常使用所以是在Windows上开发的。

### 开始接触flask

安装flask框架，<kbd>win</kbd>+<kbd>R</kbd>打开运行窗口，输入`cmd`启动命令提示符，使用语句`pip install flask`安装flask框架。

安装完成后可以开始写程序了，先写一个`Hello World!`程序（文件名暂定为“测试用例.py”）。

```python
from flask import Flask
app = Flask(__name__)
@app.route("/")
def index():
  return "<h1>Hello World!</h1>"
```

然后在命令提示符中输入语句，并打印出如下所示：

```text
$ set FLASK_APP = 测试用例.py
$ flask run
* Serving Flask app "测试用例"
Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)
```

当然也可以在文件最后写上：

```python
if __name__ == "__main__":
  app.run()
```

这样可以直接调试或运行，其结果如下所示：

![VSC](/素材/傻瓜式项目：基于Flask的简单网站/VSC.png)

然后打开`http://127.0.0.1:5000/`，就可以看到被`h1`标签包裹的`Hello World!`了。

### 编写思路

首先确定这个网站需要实现的功能和需求。

在功能方面，根据用户所选择的标签来筛选并查找出相应的干员，并且用户点击搜索结果中的某一干员时，网页全屏显示该干员的高清立绘。

在需求方面，尽可能仿照游戏内的UI设计和风格进行页面布局设计，包括但不限于配色、结构、元素样式等。

其次，研究其实现的方法和步骤。

要实现上述功能，就需要相应的干员数据，可以从[干员数据表](http://wiki.joyme.com/arknights/%E5%B9%B2%E5%91%98%E6%95%B0%E6%8D%AE%E8%A1%A8)爬取。至于高清立绘，由于网上的干员立绘大多较为模糊或者有水印，所以这里选择直接拆解游戏的安装包。

> 然而，由于未知原因（可能太久没关注这些消息了），原来的干员数据表消失了，现在可以使用[这个网址](http://ak.mooncell.wiki/w/%E5%B9%B2%E5%91%98%E4%B8%80%E8%A7%88)代替。

同时为了模仿游戏风格，需要对游戏内的UI进行分析，比如使用某些工具获取常用配色的rgb颜色值，对于我这种完全没有美术知识和天赋的纯种理工生来说，只能进行大量的试验来一步步向预想的样式靠近。

### 获取干员数据

开始写网络爬虫，这里可以直接存到数据库，也可以存到Excel。

```python
from bs4 import BeautifulSoup
from pandas import DataFrame
from threading import Thread
import urllib
import requests
```

> 由于之前的提到的网址变更导致html文档结构改变，所以这一段标签查找的代码暂时删除。

爬取的内容包括干员的代号、星级、性别、阵营、特性和职业，其中代号、阵营和星级这三个内容在查找过程中是不需要的，它们用于结果的显示和高清立绘的显示。

> 由于游戏更新，`男性干员`和`女性干员`标签被删除，所以性别项可以不用爬取。

因为抽卡的时候，抽到的角色画面是阵营+星级+立绘+代号，那么想实现这个样式，筛选出结果后点击干员头像，就可以看到阵营图+星级+角色立绘，而且角色立绘可以在初始/精一/精二/皮肤之间手动切换。

稍微提一下`threading`这个库，因为`GIL锁`的存在，平时不是很经常用，但是对于I/O密集型运算，比如读取保存文件还是很好用的。可以写一个像下面的这样类来获取返回值用于测试：

```python
class ProgramThread(Thread):
    def __init__(self, target, args):
        Thread.__init__(self)
        self.target = target
        self.args = args
        self.result = self.target(*self.args)
    def ReturnResult(self):
        try:
            return self.result
        except:
            return None
```

爬取后的内容和下面的表格类似：

<table>
  <thead>
    <tr>
      <th>代号</th><th>职业</th><th>星级</th><th>性别</th><th>阵营</th><th>特性</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>红</td><td>特种干员</td><td>5</td><td>女性干员</td><td>罗德岛</td><td>近战位, 控场, 快速复活</td>
    </tr>
    <tr>
      <td>梓兰</td><td>辅助干员</td><td>5</td><td>女性干员</td><td>罗德岛</td><td>远程位, 减速</td>
    </tr>
    <tr>
      <td>史都华德</td><td>狙击干员</td><td>5</td><td>男性干员</td><td>罗德岛</td><td>远程位, 输出</td>
    </tr>
    <tr>
      <td>……</td><td>……</td><td>……</td><td>……</td><td>……</td><td>……</td>
    <tr>
  </tbody>
</table>

### 获取图片
高清图片啥的，当然是要最初始的原画图最好啦，那么……就……就……拆包吧……哎呀，不会对外发布拆包内容的，这也是网站没有上线没有开源没有发布的主要原因，只是个娱乐向的东西，不想引来不必要的麻烦。
那么，作为一个对java零基础的菜鸟，当然不会自己造轮子啦，直接去找最新版的[`apktool`](https://ibotpeaches.github.io/Apktool/)，然后在apk下载时配置好路径，下好了在`cmd`里输入指令，开始反编译（应该是反编译吧）：
```text
$ java -jar 【工具名：如apktool_2.4.0.jar】 d 【目标名：如arknights-hg-0821.apk】
I: Using Apktool 2.4.0 on arknights-hg-0821.apk
I: Loading resource table...
I: Decoding AndroidManifest.xml with resources...
……
I: Regular manifest package...
I: Decoding file-resources...
I: Decoding values */* XMLs...
I: Baksmaling classes.dex...
I: Copying assets and libs...
……
```
然后就可以用`AssetStudioGUI`把图片文件读出来了，当然里面文件还是很多的，所以找文件还是要点时间，不过一回生二回熟，习惯就好……<br>
![拆包过程](/素材/傻瓜式项目：基于Flask的简单网站/拆包过程.png)<br>
但是导出文件后发现，所需的大部分的图片都是分成两个部分的。<br>
![](/素材/傻瓜式项目：基于Flask的简单网站/拆包立绘（全局）.png)<br>
比如陈sir的初始立绘就分成`char_010_chen_1.png`和`char_010_chen_1[alpha].png`两个图片文件，第一个文件是背景杂乱的彩色图片，第二个文件是灰度图。<br>
![拆包立绘（彩色）](/素材/傻瓜式项目：基于Flask的简单网站/拆包立绘（彩色）.png)<br>
![拆包立绘（黑白）](/素材/傻瓜式项目：基于Flask的简单网站/拆包立绘（黑白）.png)<br>
这明显是不能直接使用的，后来想到，是不是将这两个图片组合使用就行了，刚好文件名的`alpha`感觉很熟悉，所以当把第二个图片作为透明度通道加入第一张图，就可以获得游戏中的高清立绘了。
不过不知道什么工具能做到这种效果，也没去搜过，那就自己写一个。比如想把两个图片拼合到一起，可以使用`PIL`库实现：
```python
from PIL import Image
def ReadImageToImg(file_name, file_name_alpha):
    #将三通道文件增加透明度通道，并将单通道透明度文件加入该通道。
    img = Image.open(file_name).convert('RGBA')
    img_alpha = Image.open(file_name_alpha).convert('L')
    img.putalpha(img_alpha)
    return img
```
![处理立绘（特写）](/素材/傻瓜式项目：基于Flask的简单网站/处理立绘（特写）.png)<br>
这样就可以了，但图片还是很多的，每两个图片手动执行一次就会很累，所以要获取文件列表让它自己遍历运行就行了，还是很简单的：
```python
import os
def FindFileNameList(file_path):
    #将图片文件分类，三通道图像文件(R,G,B)和单通道透明度文件(alpha)。
    img_list, alpha_list = [], []
    for file_name in os.listdir(file_path):
        if 'png' not in file_name:
            continue
        if 'alpha' in file_name:
            alpha_list.append(file_name)
        else:
            img_list.append(file_name)
    return img_list, alpha_list
file_path = os.path.dirname(os.path.dirname(__file__)) + '/'
img_list, alpha_list = FindFileNameList(file_path)
```
之后循环执行之前的内容就好，不再赘述了，结果如下：<br>
![处理立绘（全局）](/素材/傻瓜式项目：基于Flask的简单网站/处理立绘（全局）.png)<br>
这里提一句，为了不让网页过于单调（毕竟大部分理工男没有艺术细菌），所以还想把yj之前发的视频在网站上循环播放，毕竟yj的pv逼格很高。所以就用[某些工具](https://www.vidpaw.com/online-video-downloader/)从YouTube上下载下来了。

### flask的jinja2引擎
flask会在应用目录中的templates子目录寻找模板，所以要写html的话文件要存在templates文件夹里，把之前写flask的文件改一下：
```python
#src/index.py
from flask import Flask, render_template
app = Flask(__name__)
@app.route("/")
def index():
  tags_dict = {
    "资质":[
        "新手", "资深干员", "高级资深干员"
        ],
    "位置":[
        "近战位", "远程位"
        ],
    "性别":[
        "男性干员", "女性干员"
        ],
    "种类":[
        "先锋干员", "术师干员", "近卫干员", "医疗干员", 
        "特种干员", "辅助干员", "重装干员", "狙击干员"
        ],
    "词缀":[
        "治疗", "支援", "输出", "群攻", "减速", "生存", "防护",
        "削弱", "位移", "控场", "爆发", "召唤", "快速复活", "费用回复"
        ]
  }
  return render_template("index.html", tags_dict = tags_dict)
```
由于游戏更新，标签已经不是上面那些了，但这不影响网页编写，这里只是我比较懒不想改了。首先除了那些标签外，返回值是一个`render_template()`，里面的参数是扔到templates里的index.html和一个标签字典，这个index.html就是一个模板，将这个`tags_dict`字典传进去填充模板内容。
那么怎么填充呢？flask使用一个叫做`Jinjia2`的模板引擎（具体是啥也不清楚），具体使用大概就是下面这样：
```html
<!-- src/templates/index.html -->
<!DOCTYPE html>
<html lang="zh-Hans">
……
{% if tags_dict %}
  <table>
    <thead>
      <tr>
        <th>类别</th><th>标签</th>
      </tr>
    </thead>
    <tbody>
      {% for key in tags_dict %}
        <tr>
          <th>{{ key }}</th>
          <td>
            {% for tags in tags_dict[key] %}
              <button>{{ tags }}</button>
            {% endfor %}
          </td>
        </tr>
      {% endfor %}
    </tbody>
  </table>
{% endif %}
……
</html>
```
看起来很简单，就是遍历字典的`key`再遍历`value`，自动生成一个html表格。上面的代码在开启flask服务器后，就可以看见下面的内容啦。
<table>
  <thead>
    <tr>
      <th>类别</th><th>标签</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>资质</th>
      <td>
        <kbd>新手</kbd><kbd>资深干员</kbd><kbd>高级资深干员</kbd>
      </td>
    </tr>
    <tr>
      <th>位置</th>
      <td>
        <kbd>近战位</kbd><kbd>远程位</kbd>
      </td>
    </tr>
    <tr>
      <th>性别</th>
      <td>
        <kbd>男性干员</kbd><kbd>女性干员</kbd>
      </td>
    </tr>
    <tr>
      <th>种类</th>
      <td>
        <kbd>先锋干员</kbd><kbd>术师干员</kbd><kbd>近卫干员</kbd><kbd>医疗干员</kbd> 
        <kbd>特种干员</kbd><kbd>辅助干员</kbd><kbd>重装干员</kbd><kbd>狙击干员</kbd>
      </td>
    </tr>
    <tr>
      <th>词缀</th>
      <td>
        <kbd>治疗</kbd><kbd>支援</kbd><kbd>输出</kbd><kbd>群攻</kbd><kbd>减速</kbd><kbd>生存</kbd><kbd>防护</kbd>
        <kbd>削弱</kbd><kbd>位移</kbd><kbd>控场</kbd><kbd>爆发</kbd><kbd>召唤</kbd><kbd>快速复活</kbd><kbd>费用回复</kbd>
      </td>
    </tr>
  </tbody>
</table>

### 样式变更
说实话，明日方舟的UI设计是真滴好看，如果把网站做成同样的风格，即没有违和感又美观。这里推荐个网站[Palette Generator](https://palettegenerator.com/)，这就很方便的可以获得游戏中的配色了。由于在手机上截屏还是有点色差的，所以每个人识别的颜色样式可能都不太一样，但是不会差的太远。
<table>
  <tr>
    <td>初始颜色</td><td>rgb(49, 49, 49)</td><td>#313131</td>
  </tr>
  <tr>
    <td>变化后颜色</td><td>rgb(3,151,216)</td><td>#0397D8</td>
  </tr>
</table>

这里的做法是，设定两个不同的class分别应用不同的样式，点击标签按钮后改变class，从而实现样式的改变。这就涉及到前端知识了，以前做数据可视化时发现在浏览器上渲染出来的可交互式图表很好用，所以自学了一些html，css和JavaScript用于修正图表。
把之前的`index.html`里的`<button>`标签改成`<button class='not-selected'>`，这里的意思是把页面里的`<button>`元素“类名”（是这么翻译的吧？）设定为`not-selected`。现在可以把每个`class`“绑定”一个样式，一般由css编写，可以写在`index.html`里用`<style>`标签包裹住。这里就先写在css文件里：

```css
/* src/static/style.css */
……
button {
  margin:某个值;
  pandding:某个值;
  width:某个值;
  height:某个值;
  color: white;
}
.not-selected {
  background-color: rgb(49, 49, 49);
}
.selected {
  background-color: rgb(3,151,216);
}
……
```
![页面截图（无操作）](/素材/傻瓜式项目：基于Flask的简单网站/页面截图（无操作）.png)<br>
现在需要它随着用户点击而改变样式，即点击后，将class的`not-selected`改为`selected`。这时候就需要JavaScript了：

```javascript
// src/static/ChangeButtonClass.js
……
$("button").on("click", function(){
  if($(this).attr("class") == "not-selected"){
    $(this).attr("class", "selected");
  }
  else if($(this).attr("class") == "selected"){
    $(this).attr("class", "not-selected");
  }
});
……
```
![页面截图（有操作未封顶）](/素材/傻瓜式项目：基于Flask的简单网站/页面截图（有操作未封顶）.png)<br>
由于在游戏中最多可选标签为5至6个，所以这里也将可选标签上限设为6个，选择6个标签后则将其他标签修改为不可点击状态，并更改全局样式提醒。<br>
![页面截图（有操作未封顶）](/素材/傻瓜式项目：基于Flask的简单网站/页面截图（封顶）.png)<br>

### 前后交互
按照习惯来说，每次点击标签按钮时都要进行一次查询，所以每次点击按钮都要向服务器发送一次询问信息，向接收一次服务器回应。每个人对自己的要求和标准都不一样，有的人就把所有的数据放在json文件里，不需要前后交流；有些人则是发送查询的tags，接收结果关键字用于填充jinja2模板；有些人会把tags发过去后，根据返回的关键字用js脚本生成新的html文档来个局部刷新；也有的人会根据tags查询结果在后端生成html文档再送到前端……方法因人而异。这里使用的是jQuery的axaj方法：

```javscript
// src/static/SendInfo.js
jQuery.axaj({
  type: type, //看情况写GET还是POST
  url: url, //此处是查询的路由
  data: data, //这里是需要向服务器发送的查询字段，也就是tags 
  dataType: dataType, //从服务器获取字段的预期类型，比如此处填写"script"，那么会将返回字段立即当作脚本执行
  success: function //完成时调用该函数
});
……
```

对应的，需要在`index.py`里添加一个查询方法。

```python
#src/index.py
……
@app.route(url, methods = ["GET", "POST"]) #此处的url是设置的新路由
def findStaff():
  #此处为搜索算法，假设储存返回字段的变量为result
  return result
……
```

### 立绘展示
创建一个新的`<div></div>`标签区域，显示在整个页面最顶层，这个区域用于显示角色立绘，不使用时隐藏，当用户点击搜索结果时显示。<br>
![立绘特写（向右切换）](/素材/傻瓜式项目：基于Flask的简单网站/立绘特写（向右切换）.png)<br>
![立绘特写（精二立绘）](/素材/傻瓜式项目：基于Flask的简单网站/立绘特写（精二立绘）.png)<br>
![立绘特写（皮肤立绘）](/素材/傻瓜式项目：基于Flask的简单网站/立绘特写（皮肤立绘）.png)<br>
按照游戏里的风格稍微调整一下布局，然后添加左右切换立绘的功能，左上角是退出按钮。
