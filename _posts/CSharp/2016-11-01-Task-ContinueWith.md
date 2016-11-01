---
layout: pageshow
category: CSharp
title: C#中关于Task、Task.ContinueWith()和Task.WaitAll()的用法
description: 简单介绍Task类
---


## C#多线程编程

在C#的多线程编程中，一般可以使用[Thread Class](https://msdn.microsoft.com/en-us/library/system.threading.thread.aspx)来进行多线程运行，但**从**.net4.0**开始，引进了[Task Class](https://msdn.microsoft.com/en-us/library/system.threading.tasks.task.aspx)之后，官方更推荐用**Task**类来异步编程。
创建一个进程需要一定的开销和时间，特别是多个线程的时候，必须考虑创建和销毁线程需要的系统开销，这时就需要用到[Thread pool](https://msdn.microsoft.com/en-us/library/0ka9477y.aspx)线程池来管理线程，防止频繁的创建和销毁线程。但是，**.net4.0**之后，微软创建了一个优化的**Task**类，它默认会创建线程池来管理task，使用起来更加方便，系统开销更小。可参见[stackoverflow上关于Task和Thread的回答](http://stackoverflow.com/questions/4130194/what-is-the-difference-between-task-and-thread)，因此，学习和使用**Task**是很有必要的。

## Task使用方式

详细方法可以参阅[codeproject的文章](http://www.codeproject.com/Articles/996857/Asynchronous-programming-and-Threading-in-Csharp-N)，写的比较详细具体。简单点来说的话，**Task**类的使用可以分成3种。

* `Task.Factory.StartNew(Action action)`

* `Task.Run(Action action)`

* ```csharp
Task task = new Task(Action action);
task.Start();
```

当然还有带有返回值[Task<T>](https://msdn.microsoft.com/en-us/library/dd321424.aspx)类，用法差不多。
至于如何终止**Task**，[codeproject的文章](http://www.codeproject.com/Articles/996857/Asynchronous-programming-and-Threading-in-Csharp-N)中也介绍了，在此不再赘述。

## Task.ContinueWith()

[Task.ContinueWith](https://msdn.microsoft.com/en-us/library/dd270696.aspx)的定义是
> Creates a continuation that executes asynchronously when the target Task completes.
就是当线程执行完毕之后，再异步运行一段程序。因为，当**Task**在后台运行时，比如进行大量的数据处理，多数情况下我们无法预知线程何时运行结束，除非进行轮询，但显然不是一个高效的方法，微软又提供了适用的方法，
举个简单控制台例子

```csharp
using System;
using System.Collections.Generic;
using System.Text;
using System.Threading.Tasks;
using System.Threading;

namespace ConsoleApplication1
{
    class Program
    {
        static void Main(string[] args)
        {
            Console.WriteLine("Task ContinueWith test begins:");
            doTask();
            Thread.Sleep(1000);
            Console.WriteLine("this is a main thread");
            Console.ReadLine();
        }

        public static Task doTask()
        {
            return Task.Run(() => {
                Console.WriteLine("task step1");
            }).ContinueWith((preTask) => {
                Console.WriteLine("task step2");
            }).ContinueWith((preTask) => {
                Thread.Sleep(1000);
                Console.WriteLine("task step3");
            });

        }

    }
}
```

输出结果为

```
Task ContinueWith test begins:
task step1
task step2
this is a main thread
task stpe3
```

从实际用法来看，执行完`Task.Run()`之后接着异步运行`ContinueWith()`的任务。

## Task.WaitAll()的用法

进一步的，如何知道**Task**是否运行结束，可以使用[Task.WaitAll()](https://msdn.microsoft.com/en-us/library/dd270695.aspx)

> Waits for all of the provided Task objects to complete execution

运行一个简单的控制台例子

```csharp
using System;
using System.Collections.Generic;
using System.Text;
using System.Threading.Tasks;
using System.Threading;

namespace ConsoleApplication1
{
    class Program
    {
        static void Main(string[] args)
        {
            List<Task> taskList = new List<Task>();
            Console.WriteLine("Task WaitAll test begins:");
            for (int i=0; i < 5; i++)
            {
                Task task = doTask(i);
                taskList.Add(task);
            }
            Task.WaitAll(taskList.ToArray());
            Console.WriteLine("this is a main thread");
            Console.ReadLine();
        }

        public static Task doTask(int i)
        {
            return Task.Run(() => {
                Console.WriteLine("task step1 of " + i.ToString());
            }).ContinueWith((preTask) => {
                Console.WriteLine("task step2 of " + i.ToString());
            }).ContinueWith((preTask) => {
                Console.WriteLine("task step3 of " + i.ToString());
            });

        }

    }
}

```

创建了5个子线程，同时运行，运行结果是
[Task.WaitAll]({{site.baseurl}}/img/CSharp/Task.WaitAll.jpg)

 **Task.WaitAll**是等待所有的**Task**运行结束，但是会阻塞当前线程，如果是在**WinForm**编程的话，为了防止UI主线程被阻塞，应该创建一个子线程来等待。

