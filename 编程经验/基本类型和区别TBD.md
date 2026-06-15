python :int, float, bool, str, tuple, frozenset, bytes
java   : byte, short, int, long, float, double, char, boolean
python所有东西都是对象，所有东西都是引用，即便是java中基本数据类型int,bool也都是引用， python会在自己的常量堆里新建好数字，之后用对象指针指向这个值
比如a=2 ，a还是对象，a的值还是一个内存地址而不是真正意义上的'2'。
java的基本类型创建后存储的是实打实的值，比如a=2， a存的值就是2，而不是内存地址。 java也提供了自动拆箱装箱的机制，即int->integer, 这里拆装箱指的是自动帮你吧基本类型转换成对象，或者对象转化成基本数字类型。
装箱后的对象就跟python的对象一样了, 顺便说一下自动拆装箱意义，python中万物是对象，引用类型只能存对象，再python中你可以直接list.add(int). 但是java中由于存在基本数字类型，他们并不是对象，那么在没有拆装箱之前，
如果你想list.add(1). 你必须写成list.add(new integer(1)) 即把1新建成对象，而不能直接引用已有的基本类型。自动拆箱之后，list.add(1)是合法的，因为JVM自动帮你做了int和integer的转换，节省了很多事情。

string 有点意思，你每次对string做的增加修改实际上都是创建了一个新的string,java有StringBuilder 或 StringBuffer来做可变的string,python并没有这种东西。

list, dict, set, bytearray

String, ArrayList, Thread,数组

list vs arraylist:
都是数组，可存储任意类型，都可以添加或者删除，存储都是引用对象（即存储内存地址）。
其中java还有个传统数组xxx[]，也是可变的不过存的是基本数据类型，即直接存具体的值而不是内存地址。

dict vs hashmap:

set vs HashSet

列表:有点类似arraylist，可变长度，不限类型，增删改查，
元祖:创造后不可改变的一组数据，提供index(查询特定值位置）和count（计算一个值出现次数）

