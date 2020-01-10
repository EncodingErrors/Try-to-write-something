## 傻瓜式项目：基于Flask的简单网站
#### 前言
本人菜鸟一枚，由于舍友一直在写PHP，感觉建网站好像挺好玩的，所以自己也想弄一个来玩。既然自己最熟悉的语言是Python，那就用Python来试试。
我很早就听说过django的威名，但这次不打算用它。因为感觉上建立小白的简单网站不会用到很复杂的功能，听说Flask比较适合小型个人网站，所以就用Flask了。
>菜鸟本人：你喜欢Django还是喜欢Flask？ <br>
>大神舍友：我永远喜欢PHP，PHP是世界上最好的语言（兴奋）！<br>
>菜鸟本人:……没救了……<br>
### 写一个“Hello World！”
咱们有个习惯，不管第一次学什么语言用什么框架，都想尽办法弄个`"Hello World！"`出来，也是老传统了。那好，这次也弄出来，而且字要大，气势要足。
根据新手教程，我们要先安装它，这简单，打开`cmd`先来一发：`pip install flask`，成了。
下面正式开始第一步，在Python里导入它，并创建一个应用实例：
```python
from flask import Flask
app = Flask(__name__)
```
看到代码就很疑惑，这个`__name__`是啥，为啥要把它传进去？当然，大家都知道`if __name__ == "__main__":`……诶！然后就反应过来了，这很明显就是定位应用位置的东西嘛！
然后继续按照新手教程敲：
```python
@app.route("/")
def index():
  return "<h1>Hello World!</h1>"
```
嗯？装饰器？啥玩意儿？这个函数就返回一个字符串？等等，这个字符串格式有点眼熟……等下，这就是传说中的html文本吗？！
是的，这就结束了，`flask`自带开发服务器，现在运行它，就可以看到大且气势足的`"Hello World！"`……吗？
不行！如果你是在`cmd`里调用的话，你要这样：
```text
$ set FLASK_APP = 你刚写的东西.py
$ flask run
* Serving Flask app "你刚写的东西"
Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)
```
