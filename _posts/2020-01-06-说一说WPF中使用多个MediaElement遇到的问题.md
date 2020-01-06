---
title: "说一说WPF中使用多个MediaElement遇到的问题"
author: Tom
date: 2020-01-06 20:56:38 +0800
CreateTime: 2020-01-06 20:56:38 +0800
categories: 
---
## 背景
一款广告播放器的需求：可以循环播放多个视频当每个视频播完后立即播放下一个，中间不得有黑屏过程。
##第一版方案
申明一个MediaElement 当每个视频播放完成后（MediaEnded）事件中指定下一个视频地址 这边就不贴出代码了
结果：每个视频交替时会有长时间空白期原因：1.设置MediaElement.source后视频缓冲需要时间（VlC就没有这问题  但是VLC在wpf下只能离屏渲染播放炒鸡卡 故放弃）。2.设置soucre后调用play()会出现不发调用成功的情况（应该正在缓冲加载，这个时候调用play()失效）。
<code>放弃第一方案</code>

## 第二方案
声明多个MediaElement每个初始化时设置source，当播放摸个视频时，显示对应的MediaElement并调用play()，否则隐藏并pause();
结果：程序打开时开的要命，内存飙升。多个视频同时缓冲 崩溃。。。。
方案优化：
```C#
/// <summary>
    /// MainWindow3.xaml 的交互逻辑
    /// </summary>
    public partial class MainWindow3 : Window
    {
        public MainWindow3()
        {
            InitializeComponent();
        }

        public int currentindex = 0;

        public List<string> datalist = new List<string>() { AppDomain.CurrentDomain.BaseDirectory + "1.mp4", AppDomain.CurrentDomain.BaseDirectory + "2.mp4", AppDomain.CurrentDomain.BaseDirectory + "3.mp4", AppDomain.CurrentDomain.BaseDirectory + "4.mp4" };

        private void Window_Loaded(object sender, RoutedEventArgs e)
        {
            MediaElement media1 = new MediaElement();
            //media1.Clock.CurrentProgress.
            media1.LoadedBehavior = MediaState.Manual;
            media1.UnloadedBehavior = MediaState.Stop;
            media1.Source = new Uri(AppDomain.CurrentDomain.BaseDirectory + "1.mp4", UriKind.Absolute);
            media1.MediaFailed += MediaFailed;
            media1.MediaEnded += MediaEnded;
            media1.Width = 400;
            media1.Height = 300;
            media1.Visibility = Visibility.Hidden;

            MediaElement media2 = new MediaElement();
            //media1.Clock.CurrentProgress.
            media2.LoadedBehavior = MediaState.Manual;
            media2.UnloadedBehavior = MediaState.Stop;
            //media2.Source = new Uri(AppDomain.CurrentDomain.BaseDirectory + "3.mp4", UriKind.Absolute);
            media2.MediaFailed += MediaFailed;
            media2.MediaEnded += MediaEnded;
            media2.Width = 400;
            media2.Height = 300;
            media2.Visibility = Visibility.Hidden;


            MediaElement media3 = new MediaElement();
            //media1.Clock.CurrentProgress.
            media3.LoadedBehavior = MediaState.Manual;
            media3.UnloadedBehavior = MediaState.Stop;
            //media2.Source = new Uri(AppDomain.CurrentDomain.BaseDirectory + "3.mp4", UriKind.Absolute);
            media3.MediaFailed += MediaFailed;
            media3.MediaEnded += MediaEnded;
            media3.Width = 400;
            media3.Height = 300;
            media3.Visibility = Visibility.Hidden;

            MediaElement media4 = new MediaElement();
            //media1.Clock.CurrentProgress.
            media4.LoadedBehavior = MediaState.Manual;
            media4.UnloadedBehavior = MediaState.Stop;
            //media2.Source = new Uri(AppDomain.CurrentDomain.BaseDirectory + "3.mp4", UriKind.Absolute);
            media4.MediaFailed += MediaFailed;
            media4.MediaEnded += MediaEnded;
            media4.Width = 400;
            media4.Height = 300;
            media4.Visibility = Visibility.Hidden;

            this.maincanvas.Children.Add(media1);
            this.maincanvas.Children.Add(media2);
            this.maincanvas.Children.Add(media3);
            this.maincanvas.Children.Add(media4);
        }


        public async void StartPlayResource()
        {
            await Task.Factory.StartNew(() =>
            {
                double endtime = 10;
                Dispatcher.Invoke(delegate
                {
                    MediaElement v = this.maincanvas.Children[currentindex] as MediaElement;
                    v.Visibility = Visibility.Visible;
                    if (v.NaturalDuration.HasTimeSpan)
                    {
                        endtime = v.NaturalDuration.TimeSpan.TotalSeconds;
                    }
                    v.Stop();
                    v.Play();
                });
                LoadNextSource(endtime);
            });
        }

        private void LoadNextSource(double seconds)
        {
            Task.Factory.StartNew(() =>
            {
                int delaytime = 10;
                if (seconds > 3)
                {
                    delaytime = (int)(seconds - 3);
                }
                else
                {
                    delaytime = 2;
                }
                Thread.Sleep(delaytime * 1000);
                Dispatcher.Invoke(delegate
                {
                    if (currentindex < datalist.Count - 1)
                    {
                        MediaElement media = this.maincanvas.Children[currentindex + 1] as MediaElement;
                        if (media.Source == null)
                        {
                            media.Source = new Uri(datalist[currentindex + 1]);
                        }
                    }
                });
            });
        }

        private void MediaFailed(object sender, ExceptionRoutedEventArgs e)
        {
            try
            {
                MessageBox.Show("报错");
            }
            catch (Exception ex)
            {

            }
        }

        private void MediaEnded(object sender, RoutedEventArgs e)
        {
            try
            {
                UIElement uiitem = sender as UIElement;
                uiitem.Visibility = Visibility.Hidden;
                if (currentindex == datalist.Count - 1)
                {
                    currentindex = 0;
                }
                else
                {
                    currentindex++;
                }
                StartPlayResource();
            }
            catch (Exception ex)
            {

            }
        }

        private void Button_Click(object sender, RoutedEventArgs e)
        {
            try
            {
                currentindex = 0;
                StartPlayResource();
            }
            catch (Exception ex)
            {

                MessageBox.Show(ex.Message);
            }
        }
    }
```
初始化的时候第一个视频赋值source 每个视频快结束是单独线程加载下一个视屏。
测试结果:播放没有问题，程序打开流畅。 其实可以在每个时候播放完成后释放资源，不然视频多了仍然会内存爆棚。
<code>这边有一个很大的坑  在AMD独显机器上会爆硬件错误，随后只要运行视频程序（如QQ影音  迅雷影音 mediaplayer）播放视屏就会死机 至少AMD的R系列是这样。查了所有资料，仍然说不清是显卡问题还是解码器问题，但本人觉得是AMD显卡问题，而且不同版本的显卡驱动下表现得状态不一样<code>


最后感谢[德熙](https://blog.lindexi.com/)帮我解决AMD显卡问题，方案是使用了他们厂封装的DirectShowLib-2005的WPF控件替代了MediaElement
