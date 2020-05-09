---
title: "async和await 线程上下文的问题"
author: tom
date: 2020-05-09 20:56:38 +0800
CreateTime: 2020-05-09 20:44:18 +0800
categories: 
---
大家都知道使用async和await可以极大的方便开发人员使用异步编程，但是有些时候，我们可能会困扰某些代码是在什么线程上执行的，特别是在界面图形变成时，如果在UI线程之外处理调用控件则会引起异常。  
下面我们看一段控制台代码：  

```csharp
public class A
    {

        public void show()
        {
            Console.WriteLine($"一    ：{Thread.CurrentThread.ManagedThreadId}");
            M1();
            Console.WriteLine($"四    ：{Thread.CurrentThread.ManagedThreadId}");
        }
        private async Task M1()
        {
            Task t = Task.Run(() =>
            {
                Console.WriteLine($"二    ：{Thread.CurrentThread.ManagedThreadId}");
                Thread.Sleep(5000);
            });
            await t;
            Console.WriteLine($"三    ：{Thread.CurrentThread.ManagedThreadId}");
        }

    }
```  
在main方法实例化A的对象，并且调用show方法，那么我们可以得到如下图的执行顺序：  
![图片1](/assets/TIM截图20200509225638.jpg)  
由图中看出来 await之后的代码都是在task的线程里面执行的。事实上，网上很多教程都是给出这个结论或者解释：await之后的代码封装成委托，在task执行完成后被调用。但是这个结论并不完全正确，下面请看这样一段代码：  
```csharp
private async void button1_Click(object sender, EventArgs e)
{
    var t = Task.Run(() => {
        Thread.Sleep(5000);
        return "Hello I am TimeConsumingMethod";
    });
    textBox1.Text = await t;
}
```  
使用wpf程序测试一下这个代码，我们会发现代码可以执行，点击按钮触发事件后界面没有卡死，并且在5秒之后，文本成功赋值。 大家可能汇友疑问，按照我们上面的结论，对textbox操作是在task的线程中执行的，也就是说在非UI线程中操作了图形控件，那么实际上真的是这样吗？我们把上面的代码改造下：  
```csharp
 private   void button_Click(object sender, RoutedEventArgs e)
        {
            Console.WriteLine($"一    ：{Thread.CurrentThread.ManagedThreadId}");
            M1();
            Console.WriteLine($"四    ：{Thread.CurrentThread.ManagedThreadId}");
           
        }


        private  async  Task M1()
        {
            Task  t = Task.Run(() =>
            {
                Console.WriteLine($"二    ：{Thread.CurrentThread.ManagedThreadId}");
                Thread.Sleep(5000); 
            });
          
            await t;
            
            Console.WriteLine($"三    ：{Thread.CurrentThread.ManagedThreadId}");
   

        }

```
运行代码后结果如下图：  
![图片1](/assets/TIM截图20200509225559.jpg)  
我们发现wpf中当主线程启动一个task，并await, 当task代码执行完成之后，await之后代码是在主线程执行的，这样就解释了上面为什么await之后操作ui控件没有异常。那么wpf是如何控制await之后的代码回到主线程执行的呢？这里面其实用到了线程上下文的概念，wpf winform asp.net 都实现了自己的线程上下文机制。有关线程上下文的介绍在[德熙](https://blog.lindexi.com/)的博客里面有详细介绍。
