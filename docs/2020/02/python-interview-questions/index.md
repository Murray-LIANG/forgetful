# Python Interview Questions

## Python是怎么样一种语言，它有什么特点？Golang与Python相比，有什么特点？

## 如果让你用Python写一个并行执行的程序，你会怎样写？Python的多线程有什么优缺点？

## 以下程序的运行结果？Why？做什么样改动可以避免类似错误？
```python
def f(x,l=[]):
    for i in range(x):
        l.append(i*i)
    print l

f(2)
f(3,[3,2,1])
f(3)
```

## 什么是Python装饰器？写个装饰器的例子？如果装饰器需要参数化，该如何改？

## The difference between @classmethod, @staticmethod? And what is @property?

## 性能分析，如何证明？如何做代码分析？
```python
lst = [random.random() for _ in range(10000)]

def f1(lst):
    l1 = sorted(lst)
    l2 = [i for i in l1 if i<0.5]
    return [i*i for i in l2]

def f2(lst):
    l1 = [i for i in lst if i<0.5]
    l2 = sorted(l1)
    return [i*i for i in l2]

def f3(lst):
    l1 = [i*i for i in lst]
    l2 = sorted(l1)
    return [i for i in l1 if i<(0.5*0.5)]
```

## 什么是buffer protocol？什么是memoryview？

## Python3有哪些特性（改进）对程序的性能有提升？

## What are generators? How they work? What is Yield from?

## 什么是异步编程？asyncio in Python?

Eventloop, Coroutine, Task, Future.

## Python中单下划线和双下划线？

## The differences between lists and tuples?

## What is the default copy behavior? The diff between deep and shallow copy?

## What do *args, **kwargs mean?

## The diff between range and xrange?

## How are parameters passed? By value or by reference?

## Coding

### Two sum
给定一个list，里面全是数字，找出两个数和等于指定的数，返回这两个数在list中的index。

hints:
- Brute force

- Two-pass hash table

- One-pass hash table

### Three sum
给定一个list，里面全是数字，找出三个数的和等于0，返回这三个数在list中的index。

hint:
1. Sort
2. 

```java
public List<List<Integer>> threeSum(int[] num) {
    Arrays.sort(num);
    List<List<Integer>> res = new LinkedList<>(); 
    for (int i = 0; i < num.length-2; i++) {
        if (i == 0 || (i > 0 && num[i] != num[i-1])) {
            int lo = i+1, hi = num.length-1, sum = 0 - num[i];
            while (lo < hi) {
                if (num[lo] + num[hi] == sum) {
                    res.add(Arrays.asList(num[i], num[lo], num[hi]));
                    while (lo < hi && num[lo] == num[lo+1]) lo++;
                    while (lo < hi && num[hi] == num[hi-1]) hi--;
                    lo++; hi--;
                } else if (num[lo] + num[hi] < sum) lo++;
                else hi--;
           }
        }
    }
    return res;
}
```
