# python多线程

线程基于进程，在操作系统中进程是分配计算机资源的单位，一个进程中的线程可以共享进程中的资源。

面向gpt学习：

Python中的多线程模块是`threading`，以下是一些常用的方法和参数：

1. `threading.Thread(target=None, args=(), kwargs={}, *, daemon=None)`

   - `target`: 要执行的函数名。
   - `args`: 传递给目标函数的参数元组。
   - `kwargs`: 传递给目标函数的关键字参数字典。
   - `daemon`: 布尔值，表示线程是否为守护线程。默认为`None`，表示继承父线程的守护状态。

2. `start()`

   - 启动线程。

3. `join(timeout=None)`

   - 等待线程终止。如果提供了`timeout`参数，则最多等待指定的秒数。

4. `is_alive()`

   - 返回线程是否处于活动状态。

5. `getName()`

   - 返回线程的名称。

6. `setName(name)`

   - 设置线程的名称。

7. `ident`

   - 线程的标识符，是一个非零整数。

8. `activeCount()`

   - 返回当前活动的线程数量。

9. `currentThread()`

   - 返回当前线程的实例。

10. `main_thread()`

    - 返回主线程的实例。

11. `enumerate()`

    - 返回一个包含所有当前活动线程的列表。

12. `settrace(func)`

    - 为所有线程设置一个跟踪函数。

13. `setprofile(func)`

    - 为所有线程设置一个分析函数。

    ```python
    import threading
    import time
    
    def print_numbers():
        for i in range(10):
            print(i)
            time.sleep(1)
    
    def print_letters():
        for letter in 'abcdefghij':
            print(letter)
            time.sleep(1.5)
    
    # 创建两个线程
    t1 = threading.Thread(target=print_numbers)
    t2 = threading.Thread(target=print_letters)
    
    # 启动线程
    t1.start()
    t2.start()
    
    # 等待线程结束
    t1.join()
    t2.join()
    
    print("All threads finished.")
    
    ```

1. `getName()`方法已被弃用，建议使用`name`属性代替。将代码中的`thread1.getName()`和`thread2.getName()`替换为`thread1.name`和`thread2.name`。
2. `activeCount()`方法已被弃用，建议使用`active_count()`方法代替。将代码中的`threading.activeCount()`替换为`threading.active_count()`。
3. `currentThread()`方法已被弃用，建议使用`current_thread()`方法代替。将代码中的`threading.currentThread()`替换为`threading.current_thread()`

```python
import threading
import time

# 定义一个函数，作为线程的目标
def print_numbers():
    for i in range(int(10)):
        time.sleep(1)
        print(i)

# 定义另一个函数，作为线程的目标
def print_letters():
    for letter in 'asdfghj':
        time.sleep(1.5)
        print(letter)

# 创建两个线程对象
thread1 = threading.Thread(target=print_numbers, name="PrintNumbers")
thread2 = threading.Thread(target=print_letters, name="PrintLetters")

# 启动线程
thread1.start()
thread2.start()

# 等待线程结束
thread1.join(timeout=3)
thread2.join(timeout=2)

# 检查线程是否存活
print("Thread 1 is alive:", thread1.is_alive())
print("Thread 2 is alive:", thread2.is_alive())

# 获取线程名称和标识符
print("Thread 1 name:",thread1.name)
print("Thread 2 name:", thread2.name)
print("Thread 1 identifier:", thread1.ident)
print("Thread 2 identifier:", thread2.ident)

# 获取当前活动的线程数量
print("Active threads count:", threading.active_count())

# 获取当前线程和主线程的实例
current_thread = threading.current_thread()
main_thread = threading.main_thread()
print("Current thread:", current_thread)
print("Main thread:", main_thread)

# 列出所有活动线程
all_threads = list(threading.enumerate())
print("All active threads:", all_threads)

# 设置跟踪函数（这里仅作演示，实际使用时需要实现相应的trace_func）
def trace_func(*args):
    print("Trace function called with args:", args)

threading.settrace(trace_func)

# 设置分析函数（这里仅作演示，实际使用时需要实现相应的profile_func）
def profile_func(*args):
    print("Profile function called with args:", args)

threading.setprofile(profile_func)

```

## 线程池

Python线程池主要使用`concurrent.futures`模块中的`ThreadPoolExecutor`类。以下是一些常用的方法：

1. `__init__(self, max_workers=None)`: 初始化线程池，`max_workers`参数表示线程池中的最大线程数。如果不指定，默认为系统的CPU核心数。

2. `submit(fn, *args, **kwargs)`: 提交一个任务到线程池，`fn`是要执行的函数，`*args`和`**kwargs`是传递给函数的参数。返回一个`concurrent.futures.Future`对象，可以用来获取任务的结果或异常。

3. `map(func, iterable, timeout=None)`: 类似于内置的`map()`函数，将`func`应用于`iterable`中的每个元素。返回一个迭代器，包含每个元素应用`func`后的结果。`timeout`参数表示等待结果的最长时间。

4. `shutdown(wait=True)`: 关闭线程池，不再接受新的任务。如果`wait`参数为`True`，则等待所有已提交的任务完成。

5. `shutdown(wait=False)`: 立即关闭线程池，不等待已提交的任务完成。

6. `with ThreadPoolExecutor(max_workers=None) as executor:`: 使用上下文管理器，可以自动调用`shutdown()`方法来关闭线程池。

示例代码：

```python
from concurrent.futures import ThreadPoolExecutor
import time

def task(n):
    time.sleep(n)
    return n

# 创建一个线程池，最大线程数为3
with ThreadPoolExecutor(max_workers=3) as executor:
    # 提交任务并获取Future对象
    futures = [executor.submit(task, i) for i in range(1, 4)]

    # 获取任务结果
    results = [future.result() for future in futures]
    print("Results:", results)
```

`ThreadPoolExecutor`类中的`submit(fn, *args, **kwargs)`方法和`map(func, iterable, timeout=None)`方法的主要区别在于它们的使用场景和返回值。

1. `submit(fn, *args, **kwargs)`方法：
   - 用于提交一个可调用的函数（如函数、方法、lambda表达式等）到线程池中执行，并返回一个`concurrent.futures.Future`对象。
   - 可以传递任意数量的位置参数和关键字参数给被调用的函数。
   - 适用于单个任务的异步执行。
   - 返回一个`Future`对象，可以通过该对象获取任务执行结果或异常信息。
2. `map(func, iterable, timeout=None)`方法：
   - 用于将一个函数应用于一个可迭代对象的每个元素，并将结果收集到一个迭代器中。
   - 适用于多个任务的并行执行，其中每个任务都是对可迭代对象的一个元素应用给定的函数。
   - 返回一个迭代器，包含应用函数后的结果。
   - 不支持传递额外的位置参数和关键字参数给被调用的函数。
   - 如果提供了`timeout`参数，那么当超过指定的时间时，会抛出`concurrent.futures.TimeoutError`异常。

总结：`submit`方法适用于单个任务的异步执行，而`map`方法适用于多个任务的并行执行。此外，`submit`方法允许传递额外的参数给被调用的函数，而`map`方法则不允许。