---
title: "Python Concurrent Module"
date: 2020-02-10T16:37:31+08:00
lastmod: 2020-02-10T16:37:31+08:00
draft: false
show_in_homepage: true
description_as_summary: true
license: ""

tags: [
    "python",
    "concurrent",
]

comment: true
toc: true
auto_collapse_toc: true
---

GIL会使得Python程序无法通过多线程实现并行计算，IO密集型稍微好一些，因为IO blocking会释放GIL，但是计算密集型无法通过多线程充分利用多核CPU的优势，因此可以利用concurrent.futures模块的ProcessPoolExecutor来利用多核CPU来提升执行速度。

## ProcessPoolExecutor

利用multiprocessing模块提供的底层机制，在主解释器之外生成多个子解释器，然后在主解释器和子解释器之间使用socket传输参数和结果等数据，具体如下：

1. 把列表中或者iteration传给map，其中每一项都会传给map指定的执行函数。
2. 用pickle对数据进行序列化，将其转成二进制。
3. 通过本地socket，将序列化的数据从主解释器所在的进程，发送到子解释器所在的进程。
4. 在子进程中，用pickle对二进制数据进行反序列化，将其还原成Python对象。
5. 引入包含执行函数的Python模块。
6. 在每个子进程中并行地对各自的输入数据进行计算。
7. 对计算结果进行序列化操作，将其转变成字节。
8. 将这些字节通过socket复制到主进程中。
9. 主进程对这些字节执行反序列化操作，还原成Python对象。
10. 最后，把每个子进程所算出的计算结果合并到一个列表中，返回给调用者。

multiprocessing开销大，因为在主进程子进程之间通信，会进行序列化和反序列化的操作。

## ProcessPoolExecutor::map的实现

```python

    def map(self, fn, *iterables, **kwargs):
        """Returns a iterator equivalent to map(fn, iter).

        Args:
            fn: A callable that will take as many arguments as there are
                passed iterables.
            timeout: The maximum number of seconds to wait. If None, then there
                is no limit on the wait time.

        Returns:
            An iterator equivalent to: map(func, *iterables) but the calls may
            be evaluated out-of-order.

        Raises:
            TimeoutError: If the entire result iterator could not be generated
                before the given timeout.
            Exception: If fn(*args) raises for any values.
        """
        timeout = kwargs.get('timeout')
        if timeout is not None:
            end_time = timeout + time.time()

        fs = [self.submit(fn, *args) for args in itertools.izip(*iterables)]

        # Yield must be hidden in closure so that the futures are submitted
        # before the first iterator value is required.
        def result_iterator():
            try:
                for future in fs:
                    if timeout is None:
                        yield future.result()
                    else:
                        yield future.result(end_time - time.time())
            finally:
                for future in fs:
                    future.cancel()
        return result_iterator()
```