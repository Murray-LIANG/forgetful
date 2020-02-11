# Python GIL, Multi-Threading, Multi-Processing


# References
https://blog.csdn.net/AnyThingFromBigban/article/details/73611386

# GIL 与 Python 线程的纠葛
GIL是什么东西？它对我们的python程序会产生什么样的影响？我们先来看一个问题。
运行下面这段python程序，CPU占用率是多少？

```python
# 请勿在工作中模仿，危险:)
def dead_loop():
    while True:
        pass

dead_loop()
```
答案是什么呢，占用100%CPU？那是单核！还得是没有超线程的古董CPU。
在我的双核CPU上，这个死循环只会吃掉我一个核的工作负荷，也就是只占用50%CPU。
那如何能让它在双核机器上占用100%的CPU呢？答案很容易想到，用两个线程就行了，
线程不正是并发分享CPU运算资源的吗。可惜答案虽然对了，但做起来可没那么简单。
下面的程序在主线程之外又起了一个死循环的线程。

```python
import threading

def dead_loop():
    while True:
        pass

# 新起一个死循环线程
t = threading.Thread(target=dead_loop)
t.start()

# 主线程也进入死循环
dead_loop()

t.join()
```
按道理它应该能做到占用两个核的CPU资源，可是实际运行情况却是没有什么改变，还是只占了50%CPU不到。
这又是为什么呢？难道python线程不是操作系统的原生线程？
打开system monitor一探究竟，这个占了50%的python进程确实是有两个线程在跑。
那这两个死循环的线程为何不能占满双核CPU资源呢？其实幕后的黑手就是GIL。

# GIL 的迷思：痛并快乐着
GIL 的全称为 Global Interpreter Lock ，意即全局解释器锁。在 Python 语言的主流实现 CPython 中，
GIL 是一个货真价实的全局线程锁，**在解释器解释执行任何 Python 代码时，都需要先获得这把锁才行**，
在**遇到 I/O 操作时会释放这把锁**。如果是纯计算的程序，**没有 I/O 操作，解释器会每隔 100 次操作**
就释放这把锁，让别的线程有机会执行（这个次数可以通过sys.setcheckinterval 来调整）。
所以虽然**CPython 的线程库直接封装操作系统的原生线程**，但 CPython 进程做为一个整体，
**同一时间只会有一个获得了 GIL 的线程在跑**，其它的线程都处于等待状态等着 GIL 的释放。
这也就解释了我们上面的实验结果：虽然有两个死循环的线程，而且有两个物理 CPU 内核，
但因为 GIL 的限制，两个线程只是做着分时切换，总的 CPU 占用率还略低于 50%。

看起来 python 很不给力啊。GIL 直接导致 CPython 不能利用物理多核的性能加速运算。那为什么会有这样
的设计呢？我猜想应该还是历史遗留问题。多核 CPU 在 1990 年代还属于类科幻，Guido van Rossum 在创
造 python 的时候，也想不到他的语言有一天会被用到很可能 1000+ 个核的 CPU 上面，一个全局锁搞定多
线程安全在那个时代应该是最简单经济的设计了。简单而又能满足需求，那就是合适的设计（对设计来说，应该
只有合适与否，而没有好与不好）。怪只怪硬件的发展实在太快了，摩尔定律给软件业的红利这么快就要到头
了。短短 20 年不到，代码工人就不能指望仅仅靠升级 CPU 就能让老软件跑的更快了。在多核时代，编程的免
费午餐没有了。如果程序不能用并发挤干每个核的运算性能，那就意谓着会被淘汰。对软件如此，对语言也是一
样。那 Python 的对策呢？

Python 的应对很简单，以不变应万变。在最新的 python 3 中依然有 GIL。之所以不去掉，原因嘛，不外以下几点：

## 欲练神功，挥刀自宫：

CPython 的 GIL 本意是用来保护所有全局的解释器和环境状态变量的。如果去掉 GIL，就需要多个更细粒度
的锁对解释器的众多全局状态进行保护。或者采用 Lock-Free 算法。无论哪一种，要做到多线程安全都会比单
使用 GIL 一个锁要难的多。而且改动的对象还是有 20 年历史的 CPython 代码树，更不论有这么多第三方的
扩展也在依赖 GIL。对 Python 社区来说，这不异于挥刀自宫，重新来过。

## 就算自宫，也未必成功：

有位牛人曾经做了一个验证用的 CPython，将 GIL 去掉，加入了更多的细粒度锁。但是经过实际的测试，对单
线程程序来说，这个版本有很大的性能下降，只有在利用的物理 CPU 超过一定数目后，才会比 GIL 版本的性
能好。这也难怪。单线程本来就不需要什么锁。单就锁管理本身来说，锁 GIL 这个粗粒度的锁肯定比管理众多
细粒度的锁要快的多。而现在绝大部分的 python 程序都是单线程的。再者，从需求来说，使用 python 绝不
是因为看中它的运算性能。就算能利用多核，它的性能也不可能和 C/C++ 比肩。费了大力气把 GIL 拿掉，反
而让大部分的程序都变慢了，这不是南辕北辙吗。

难道 Python 这么优秀的语言真的仅仅因为改动困难和意义不大就放弃多核时代了吗？其实，不做改动最最重要
的原因还在于：不用自宫，也一样能成功！

## 其它神功
那除了切掉 GIL 外，果然还有方法让 Python 在多核时代活的滋润？让我们回到本文最初的那个问题：
如何能让这个死循环的 Python 脚本在双核机器上占用 100% 的 CPU？其实最简单的答案应该是：
**运行两个 python 死循环的程序**！也就是说，用**两个**分别占满一个 CPU 内核的 **python 进程**
来做到。确实，多进程也是利用多个 CPU 的好方法。只是进程间内存地址空间独立，互相协同通信要比多线程
麻烦很多。有感于此，Python 在 2.6 里新引入了**multiprocessing**这个多进程标准库，让多进程的
python 程序编写简化到类似多线程的程度，大大减轻了 GIL 带来的不能利用多核的尴尬。

这还只是一个方法，如果不想用多进程这样重量级的解决方案，还有个更彻底的方案，放弃 Python，改用
C/C++。当然，你也不用做的这么绝，只需要**把关键部分用 C/C++ 写成 Python 扩展**，其它部分还是用
Python 来写，让 Python 的归 Python，C 的归 C。一般计算密集性的程序都会用 C 代码编写并通过扩展
的方式集成到 Python 脚本里（如 NumPy 模块）。**在扩展里就完全可以用 C 创建原生线程，而且不用锁 GIL**，
充分利用 CPU 的计算资源了。不过，写 Python 扩展总是让人觉得很复杂。好在 Python 还有另一种与 C
模块进行互通的机制 : ctypes

## 利用 ctypes 绕过 GIL
ctypes 与 Python 扩展不同，它可以让 Python 直接调用任意的 C 动态库的导出函数。你所要做的只是
用 ctypes 写些 python 代码即可。最酷的是，**ctypes 会在调用 C 函数前释放 GIL**。所以，我们可
以通过 ctypes 和 C 动态库来让 python 充分利用物理内核的计算能力。让我们来实际验证一下，这次我们
用 C 写一个死循环函数

```cpp
extern"C"
{
  void DeadLoop()
  {
    while (true);
  }
}
```
用上面的 C 代码编译生成动态库 libdead_loop.so （Windows 上是 dead_loop.dll），
接着就要利用 ctypes 来在 python 里 load 这个动态库，分别在主线程和新建线程里调用其中的 DeadLoop:

```python
from ctypes import *
from threading import Thread

lib = cdll.LoadLibrary("libdead_loop.so")
t = Thread(target=lib.DeadLoop)
t.start()

lib.DeadLoop()
```
这回再看看 system monitor，Python 解释器进程有两个线程在跑，而且双核 CPU 全被占满了，
ctypes 确实很给力！需要提醒的是，GIL 是被 ctypes 在调用 C 函数前释放的。但是 Python 解释器还
是会在执行任意一段 Python 代码时锁 GIL 的。如果你使用 Python 的代码做为 C 函数的 callback，
那么只要 Python 的 callback 方法被执行时，GIL 还是会跳出来的。比如下面的例子：

```cpp
extern"C"
{
  typedef void Callback();
  void Call(Callback* callback)
  {
    callback();
  }
}
```
```python
from ctypes import *
from threading import Thread

def dead_loop():
    while True:
        pass

lib = cdll.LoadLibrary("libcall.so")
Callback = CFUNCTYPE(None)
callback = Callback(dead_loop)

t = Thread(target=lib.Call, args=(callback,))
t.start()

lib.Call(callback)
```
注意这里与上个例子的不同之处，这次的死循环是发生在 Python 代码里 (DeadLoop 函数) 而 C 代码只是
负责去调用这个 callback 而已。运行这个例子，你会发现 CPU 占用率还是只有 50％ 不到。
GIL 又起作用了。

其实，从上面的例子，我们还能看出 ctypes 的一个应用，那就是用 Python 写自动化测试用例，
通过 ctypes 直接调用 C 模块的接口来对这个模块进行黑盒测试，哪怕是有关该模块 C 接口的多线程安全方
面的测试，ctypes 也一样能做到。

# 结语
虽然 CPython 的线程库封装了操作系统的原生线程，但却因为 GIL 的存在导致多线程不能利用多个 CPU 
内核的计算能力。好在现在 Python 有了易经筋（multiprocessing）, 吸星大法（C 语言扩展机制）
和独孤九剑（ctypes），足以应付多核时代的挑战，GIL 切还是不切已经不重要了，不是吗。
