### with语法

底层代码包含2部分，__enter 和 __exit.

即当你调用这个方法，他会先帮你自动执行enter，之后执行你的代码块，在你的代码处理结束后，他会自动执行exit。
比如读文件:
with open('','r')as f:
 f.read

 对应完整代码是
 f = open('', 'r')
data = f.read()
f.close()

这里f已经重写了enter和exit  如果你自定义对象没有这2个，会导致报错.
