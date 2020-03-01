---
title: "关于Async和await的思考"
author: tom
date: 2020-03-01 20:56:38 +0800
CreateTime: 2020-03-01 20:44:18 +0800
categories: 
---
今天和大家分享一下C# async和await的一点想法。  
关于用法上，这篇文章不做敷述。网上有大量的博文介绍，本真只是将我理解这个技术过程中，自己感到比较关键的点与大家做分享。  
首先我们要记住一句话：<code>await 之后的代码将以委托的的形式在原有的线程上执行</code>。这句话非常重要。我之前一直纠结于为什么有时候界面阻塞，有时候没有阻塞。其实就是没有注意微软官方文档的这句话。我们有一段代买描述下：  
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
异步方法（事件）button1_Click被UI线程调用后，使用Task开始一个异步任务，使用await等待任务完成。之后在UI线程上将结果赋值给textBox1.Text。之前我一直纠结于为什么没有使用：Dispatcher.invoke(()=>{ textBox1.Text = await t})，实际上textBox1.Text = await t,可以理解为string s = await t；textBox1.Text = s,所以对控件textBox1赋值操作是在UI线程。
 
