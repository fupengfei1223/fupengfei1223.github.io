---
title: "说一说C#使用websocket的一些问题"
author: Tom
date: 2020-01-06 20:56:38 +0800
CreateTime: 2020-01-06 20:56:38 +0800
categories: 
---

本位主要记录了奔驰项目中websocket通讯改造遇到的问题。

## 背景
之前终端与服务器通讯采用的是tcp/ip模式，但甲方对安全性要求极高不开放tcp通讯所需要的端口，无奈之下兄弟们决定采用websocket与终端交互，这样可以使用已有的80端口。

## 改造
原先的Tcp客户端结构如下(部分)：

``` C#
    public class TcpStocketClient
    {
    	///<summary>
        /// 客户端socket对象
        /// </summary>
        public Socket Tcpsocket;

        /// <summary>
        /// 服务线程(负责socket接收)
        /// </summary>
        public Thread processor;

        public bool Start()
        { try
            {
        	Clear();
            Tcpsocket = new Socket(AddressFamily.InterNetwork,
            SocketType.Stream, ProtocolType.Tcp);
            IPAddress ip = IPAddress.Parse(this.IP);
            IPEndPoint ipep = new IPEndPoint(ip, this.Port);//IP和端口
            //Tcpsocket.Connect(ipep);
            IAsyncResult result = Tcpsocket.BeginConnect(ip, Port, null, null);
            result.AsyncWaitHandle.WaitOne(2000, true);
            if (!result.IsCompleted)
            {
                return false;
            }

            LastRecTime = DateTime.Now;
            processor = new Thread(new ThreadStart(Communication));
            processor.SetApartmentState(ApartmentState.STA);
            processor.IsBackground = true;
            processor.Start();
            return true;
           }
            catch (Exception ex)
            {
                this.isConnect = false;
                return false;
            }
            finally
            {
                this.communicationobserve_timer.Enabled = true;
            }
        }

        private void Communication()
        {
            try
            {
                while (true)
                {
                	 //已经精简
                	 int lengh = this.Tcpsocket.Receive(buffertemp);

                }
            }
            catch (Exception ex)
            {
                this.isConnect = false;
            }
    }
```
上面代码主要使用了Tcpsocket.BeginConnect() 和Tcpsocket.Receive(); 经过一番调研 发现微软提供了using System.Net.WebSockets.ClientWebSocket类，下面进行是最小变动改造:
```C#
public class TcpWebSocketClient
    {
    	 ///<summary>
        /// 客户端socket对象
        /// </summary>
        public ClientWebSocket Wbsocket;

        /// <summary>
        /// 取消编辑
        /// </summary>
        CancellationToken cancellation = new CancellationToken();

        /// <summary>
        /// 服务线程(负责socket接收)
        /// </summary>
        public Thread processor;


        /// <summary>
        /// 启动通讯
        /// </summary>
        public bool Start()
        {
            try
            {
                Clear();
                
                string url = string.Format("ws://{0}:{1}/cdmsA/webSocket?sn={2}", IP, Port,SN);
                Wbsocket = new System.Net.WebSockets.Managed.ClientWebSocket();
                IAsyncResult result = Wbsocket.ConnectAsync(new Uri(url), cancellation);
                result.AsyncWaitHandle.WaitOne(2000, true);
                if (!result.IsCompleted)
                {
                    LogUtil.LogHelper.NetLog("websocket连接超时");
                    return false;
                }

                LastRecTime = DateTime.Now;
                processor = new Thread(new ThreadStart(Communication));
                processor.SetApartmentState(ApartmentState.STA);
                processor.IsBackground = true;
                processor.Start();
                return true;
            }
            catch (Exception ex)
            {
                this.isConnect = false;
                LogUtil.LogHelper.ErrorForNet(ex);
                return false;
            }
            finally
            {
                this.communicationobserve_timer.Enabled = true;
            }
        }


         /// <summary>
        /// 接收服务发过来的数据
        /// </summary>
        private async void Communication()
        {
            try
            {
                while (true)
                {
                	 //  int lengh = this.Tcpsocket.Receive(buffertemp);
                        WebSocketReceiveResult r = await Wbsocket.ReceiveAsync(new ArraySegment<byte>(buffertemp), new CancellationToken());//接受数据
                        if (r.Count <= 0)
                        {

                        }
                }
 			}
            catch (Exception ex)
            {
                this.isConnect = false;
            }
    }
```
websocket连接地址的格式为 "ws:+url"
同过上面代码对比可以发现微软提供websocket框架提供了Wbsocket.ConnectAsync()和Wbsocket.ReceiveAsync()对应于tcp方式的Tcpsocket.BeginConnect() 和Tcpsocket.Receive();通过以上两处修改，可以实现websocket与服务器交互。


## Https
随着甲方需求的深入，服务器要求采用https协议 与之相应的websocket也需要调整：
```C#
 string url = string.Format("ws://{0}:{1}/cdmsA/webSocket?sn={2}", IP, Port,SN);
                if (ConfigurationManager.AppSettings["IsHttps"] == "1")
                {
                    ServicePointManager.ServerCertificateValidationCallback = new RemoteCertificateValidationCallback(CheckValidationResult);
                    ServicePointManager.SecurityProtocol = (SecurityProtocolType)3072;// SecurityProtocolType.Tls1.2; 
                    ServicePointManager.CheckCertificateRevocationList = true;
                    ServicePointManager.DefaultConnectionLimit = 100;
                    ServicePointManager.Expect100Continue = false;
                    url = string.Format("wss://{0}:{1}/cdmsA/webSocket?sn={2}", IP, Port,SN);
                }

 	    private static bool CheckValidationResult(object sender, X509Certificate certificate, X509Chain chain, SslPolicyErrors errors)
        {
            return true; //总是接受  
        }
```
与http方式相比，https需要设置证书。

## Win7下的问题
由于开发部门都是win10的系统，测试以上代码都是没有问题的，某天客户问道win7系统是不是连不上服务器，当时也没多想，毕竟使用的都是微软框架下的东西。后来找到win7机器测试了下果然爆出异常，查了相关资料发现微软官方文档明确说明了websocket不支持win7.....  后背一阵发凉。最后通过如下方法解决:
引用：System.Net.WebSockets.Client.Managed   websocket对象采用该库对象
```C#
///<summary>
/// 客户端socket对象
/// </summary>
public System.Net.WebSockets.Managed.ClientWebSocket Wbsocket;
```