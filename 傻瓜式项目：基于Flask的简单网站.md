## 傻瓜式项目：基于Flask的简单网站
### 前言
本人菜鸟一枚，由于舍友一直在写PHP，感觉建网站好像挺好玩的，所以自己也想弄一个来玩。既然自己最熟悉的语言是Python，那就用Python来试试。
很早就听说过django的威名，但这次不打算用它。因为感觉上建立小白的简单网站不会用到很复杂的功能，听说Flask比较适合小型个人网站，所以就用Flask了。

>菜鸟本人：你喜欢Django还是喜欢Flask？ <br>
>大神舍友：我永远喜欢PHP，PHP是世界上最好的语言（兴奋）！<br>
>菜鸟本人:……没救了……<br>

### 写一个“Hello World！”
咱有个习惯，不管第一次学什么语言用什么框架，都想尽办法弄个`"Hello World！"`出来，也是老传统了。那好，这次也弄出来，而且字要大，气势要足。
根据新手教程，要先安装它，这简单，打开`cmd`先来一发：`pip install flask`，成了。
下面正式开始，在Python里导入它，并创建一个应用实例：
```python
from flask import Flask
app = Flask(__name__)
```
看到代码就很疑惑，这个`__name__`是啥，为啥要把它传进去？当然，大家都知道`if __name__ == "__main__":`……诶！然后就反应过来了，这很明显就是定位应用位置的东西嘛！然后继续按照新手教程敲：
```python
@app.route("/")
def index():
  return "<h1>Hello World!</h1>"
```
嗯？装饰器？啥玩意儿？这个函数就返回一个字符串？等等，这个字符串格式有点眼熟……等下，这就是传说中的html文本吗？！
是的，这就结束了，`flask`自带开发服务器，现在运行它，就可以看到大且气势足的`"Hello World！"`了……吗？
不行！如果是在`cmd`里调用的话，要这样：
```text
$ set FLASK_APP = 刚写的东西.py
$ flask run
* Serving Flask app "刚写的东西"
Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)
```
但如果不想写在`cmd`里，也可以在写好的文件末尾加上熟悉的片段：
```python
if __name__ == "__main__":
  app.run()
```
然后打开浏览器，网址输入：`http://localhost:5000/`，就可以看到非常巨大的`Hello World!`了。结束，第一个网站建好了，鼓掌（

### 开始起步
现在就看兴趣想做什么网站，当时在玩[明日方舟](https://ak.hypergryph.com/index)，看到公开招募网站还挺实用的，就自己打算搞一个类似的。
各位骡（子）的刀（客塔）都知道，公开招募的时候选择特定的标签组合必定会出四星/五星/六星，公开招募工具就是为了这些标签的精准定位，而且现在各位骡的刀最常用的大概是百度搜索关键字`公开招募`结果第一位的那个，那就稍微借鉴一下[大佬的作品](https://ak.graueneko.xyz/)，这样就不用去费心设计了，专心实现功能就好。

### 获取数据
要精确查找筛选标签，那首先要获取数据，要获取游戏数据最好还是直接去官网……本来想这么说的，但是官网并没有这方面的数据。但是可爱的玩家们总结了[干员数据表](http://wiki.joyme.com/arknights/%E5%B9%B2%E5%91%98%E6%95%B0%E6%8D%AE%E8%A1%A8)，这样我们就可以用爬虫搞到我们想要的数据啦！那就开始写爬虫，大家都懂的：
```python
from bs4 import BeautifulSoup
from pandas import DataFrame
from threading import Thread
import urllib
import requests
```
话说这个`threading`多线程库就很蛋疼，因为`GIL锁`的存在，所以这玩意儿对于CPU密集型计算基本没啥作用（当然可以用多进程避开它），但是可以用它来加速I/O密集型的计算，比如读取和保存文件。可以写一个类来启动多线程：
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
后面就省略了，这不是主要内容，爬取后可以存到数据库。对于数据清洗和过滤不太熟悉，或者字符串处理不太熟练的话，也先可以存到Excel里再手动调整。
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
      <td>……</td><td>……</td><td>……</td><td>……</td><td>……</td><td>……</td>
    <tr>
  </tbody>
</table>
顺便还把阵营信息也爬下来了，因为有个小想法。抽卡的时候，抽到的角色画面是阵营图+星级+角色立绘，那么我想实现个小功能，筛选出干员后点击干员头像，就可以看到阵营图+星级+角色立绘，而且角色立绘可以在初始/精一/精二/皮肤之间手动切换。

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
这里提一句，为了不让网页过于单调（毕竟大部分理工男没有艺术细菌），所以还想把yj之前发的视频在网站上循环播放，就直接用硕鼠去B站上下载了，大家不要学哦~

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
