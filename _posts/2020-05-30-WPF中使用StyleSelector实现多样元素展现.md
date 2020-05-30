---
title: "WPF中使用StyleSelector实现多样元素展现"
author: tom
date: 2020-05-30 20:56:38 +0800
CreateTime: 2020-05-30 20:44:18 +0800
categories: 
---
在《WPF高级编程》中作者介绍StyleSeletor时，使用了ListView控件展现了器功能；这篇文章将使用ItemControl介绍StyleSelector展现它更强大的编辑能力。 
首相我们介绍在XAML里面定义如下：


```XML
 <UserControl x:Class="AGVDisplay.UserControls.DiagramControl"
             xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006" 
             xmlns:d="http://schemas.microsoft.com/expression/blend/2008" 
             xmlns:local="clr-namespace:AGVDisplay.UserControls"
              xmlns:s="clr-namespace:AGVDisplay.StyleSelectors"
              xmlns:ucdp="clr-namespace:UICommon.UserControls.DrawParts;assembly=UICommon"
                xmlns:att="clr-namespace:UICommon.AttachedProperties;assembly=UICommon" 
              xmlns:arrows="clr-namespace:UICommon.UserControls.DrawParts.Arrows;assembly=UICommon"
              xmlns:con="clr-namespace:UICommon.Converters;assembly=UICommon"
             mc:Ignorable="d" 
             xmlns:i="http://schemas.microsoft.com/expression/2010/interactivity"
             xmlns:hc="https://handyorg.github.io/handycontrol"
             d:DesignHeight="450" d:DesignWidth="800">
    <Border>
        <ItemsControl ItemsSource="{Binding Items}" ClipToBounds="True" ItemContainerStyleSelector="{x:Static s:DesignerItemsControlItemStyleSelector.Instance}">
            <ItemsControl.Resources>
                <!--地标-->
                <Style x:Key="designerItemlandmarkStyle"
                                TargetType="{x:Type ContentPresenter}">
                    <Setter Property="Canvas.ZIndex"
                                    Value="6000" />
                    <Setter Property="Canvas.Top"
                                    Value="{Binding Top}" />
                    <Setter Property="Canvas.Left"
                                    Value="{Binding Left}" />
                    <Setter Property="Width"
                                    Value="{Binding ItemWidth}"/>
                    <Setter Property="Height"
                                    Value="{Binding ItemHeight}"/>
                    <Setter Property="SnapsToDevicePixels"
                                    Value="True" />
                    <Setter Property="LayoutTransform">
                        <Setter.Value>
                            <ScaleTransform CenterX="0.5" CenterY="0.5" ScaleY="{Binding Parent.Zoom}" ScaleX="{Binding Parent.Zoom}"></ScaleTransform>
                        </Setter.Value>
                    </Setter>
                    <Setter Property="ContentTemplate">
                        <Setter.Value>
                            <DataTemplate>
                                <Border x:Name="selectedborder" BorderThickness="2"   CornerRadius="20"  >
                                    <Border.BorderBrush>
                                        <SolidColorBrush Color="#1F47A6" Opacity="{Binding Opacity}"></SolidColorBrush>
                                    </Border.BorderBrush>
                                    <Border.Background>
                                        <SolidColorBrush Color="#3B6CCF" Opacity="{Binding Opacity}"></SolidColorBrush>
                                    </Border.Background>
                                    <TextBlock HorizontalAlignment="Center" VerticalAlignment="Center" TextWrapping="Wrap"  Text="{Binding  LandCode}" Foreground="#C6DC2E" FontSize="12"/>
                                </Border>
                            </DataTemplate>
                        </Setter.Value>
                    </Setter>
                </Style>

                <!--储位-->
                <Style x:Key="designerItemstorageStyle"
                                TargetType="{x:Type ContentPresenter}">
                    <Setter Property="Canvas.ZIndex"
                                    Value="4000" />
                    <Setter Property="Canvas.Top"
                                    Value="{Binding Top}" />
                    <Setter Property="Canvas.Left"
                                    Value="{Binding Left}" />
                    <Setter Property="Width"
                                    Value="{Binding ItemWidth}"/>
                    <Setter Property="Height"
                                    Value="{Binding ItemHeight}"/>
                    <Setter Property="SnapsToDevicePixels"
                                    Value="True" />
                    <Setter Property="LayoutTransform">
                        <Setter.Value>
                            <ScaleTransform CenterX="0.5" CenterY="0.5" ScaleY="{Binding Parent.Zoom}" ScaleX="{Binding Parent.Zoom}"></ScaleTransform>
                        </Setter.Value>
                    </Setter>
                    <Setter Property="ContentTemplate">
                        <Setter.Value>
                            <DataTemplate>
                                <Border x:Name="selectedborder" BorderThickness="3"  CornerRadius="20">
                                    <Border.BorderBrush>
                                        <SolidColorBrush Color="#53a6c1" Opacity="0.3 "></SolidColorBrush>
                                    </Border.BorderBrush>
                                    <Border.Background>
                                        <SolidColorBrush Color="{Binding ContentColor}" Opacity="0.2 "></SolidColorBrush>
                                    </Border.Background>
                                    <Grid>
                                        <Grid  x:Name="PART_LockDecorator" Margin="3" >
                                            <Path Data="{StaticResource LockGeometry}"  Stretch="Fill" Fill="#5f6c82"></Path>
                                        </Grid>
                                        <TextBlock HorizontalAlignment="Center" VerticalAlignment="Center" TextWrapping="Wrap"  Text="{Binding  StorageCode}" Foreground="#C6DC2E" FontSize="12"/>
                                    </Grid>
                                </Border>
                                <DataTemplate.Triggers>
                                    <DataTrigger Value="True"
                                                            Binding="{Binding IsLocked}">
                                        <Setter TargetName="PART_LockDecorator"
                                                        Property="Visibility"
                                                        Value="Visible" />
                                    </DataTrigger>
                                </DataTemplate.Triggers>
                            </DataTemplate>
                        </Setter.Value>
                    </Setter>
                </Style>

                <!--文本-->
                <Style x:Key="designerItemtextStyle"
                                TargetType="{x:Type ContentPresenter}">
                    <Setter Property="Canvas.ZIndex"
                                    Value="3000" />
                    <Setter Property="Canvas.Top"
                                    Value="{Binding Top}" />
                    <Setter Property="Canvas.Left"
                                    Value="{Binding Left}" />
                    <Setter Property="att:SelectionProps.EnabledForSelection"
                                    Value="False" />
                    <Setter Property="att:ItemConnectProps.EnabledForConnection"
                                    Value="False" />
                    <Setter Property="Width"
                                    Value="{Binding ItemWidth}"/>
                    <Setter Property="Height"
                                    Value="{Binding ItemHeight}"/>
                    <Setter Property="SnapsToDevicePixels"
                                    Value="True" />
                    <Setter Property="LayoutTransform">
                        <Setter.Value>
                            <ScaleTransform CenterX="0.5" CenterY="0.5" ScaleY="{Binding Parent.Zoom}" ScaleX="{Binding Parent.Zoom}"></ScaleTransform>
                        </Setter.Value>
                    </Setter>
                    <Setter Property="ContentTemplate">
                        <Setter.Value>
                            <DataTemplate>
                                <Border x:Name="selectedborder" BorderThickness="3"  CornerRadius="5">
                                    <Border.BorderBrush>
                                        <SolidColorBrush Color="{Binding BordergroundColor}" Opacity="0.5"></SolidColorBrush>
                                    </Border.BorderBrush>
                                    <Border.Background>
                                        <SolidColorBrush Color="{Binding BackgroundColor}" Opacity="{Binding BgOpacity}"></SolidColorBrush>
                                    </Border.Background>
                                    <Grid>
                                        <TextBlock HorizontalAlignment="Center" VerticalAlignment="Center" TextWrapping="Wrap"  
                                                       Text="{Binding  ContentText}"
                                                       FontSize="{Binding FontSize}"    >
                                            <TextBlock.Foreground>
                                                <SolidColorBrush Color="{Binding ForegroundColor}"></SolidColorBrush>
                                            </TextBlock.Foreground>
                                        </TextBlock>
                                    </Grid>
                                </Border>
                            </DataTemplate>
                        </Setter.Value>
                    </Setter>
                </Style>

                <!--图片-->
                <Style x:Key="designerItemimageStyle"
                                TargetType="{x:Type ContentPresenter}">
                    <Setter Property="Canvas.ZIndex"
                                    Value="3000" />
                    <Setter Property="Canvas.Top"
                                    Value="{Binding Top}" />
                    <Setter Property="Canvas.Left"
                                    Value="{Binding Left}" />
                    <Setter Property="att:SelectionProps.EnabledForSelection"
                                    Value="False" />
                    <Setter Property="att:ItemConnectProps.EnabledForConnection"
                                    Value="False" />
                    <Setter Property="Width"
                                    Value="{Binding ItemWidth}"/>
                    <Setter Property="Height"
                                    Value="{Binding ItemHeight}"/>
                    <Setter Property="SnapsToDevicePixels"
                                    Value="True" />
                    <Setter Property="LayoutTransform">
                        <Setter.Value>
                            <ScaleTransform CenterX="0.5" CenterY="0.5" ScaleY="{Binding Parent.Zoom}" ScaleX="{Binding Parent.Zoom}"></ScaleTransform>
                        </Setter.Value>
                    </Setter>
                    <Setter Property="ContentTemplate">
                        <Setter.Value>
                            <DataTemplate>
                                <Border x:Name="selectedborder" BorderThickness="1"  CornerRadius="2">
                                    <Grid  >
                                        <Grid.Background>
                                            <ImageBrush ImageSource="{Binding Image}" ></ImageBrush>
                                        </Grid.Background>
                                    </Grid>
                                </Border>
                            </DataTemplate>
                        </Setter.Value>
                    </Setter>
                </Style>

                <!--充电桩-->
                <Style x:Key="designerItemchargeStyle"
                                TargetType="{x:Type ContentPresenter}">
                    <Setter Property="Canvas.ZIndex"
                                    Value="3000" />
                    <Setter Property="Canvas.Top"
                                    Value="{Binding Top}" />
                    <Setter Property="Canvas.Left"
                                    Value="{Binding Left}" />
                    <Setter Property="att:SelectionProps.EnabledForSelection"
                                    Value="True" />
                    <Setter Property="att:ItemConnectProps.EnabledForConnection"
                                    Value="False" />
                    <Setter Property="Width"
                                    Value="{Binding ItemWidth}"/>
                    <Setter Property="Height"
                                    Value="{Binding ItemHeight}"/>
                    <Setter Property="SnapsToDevicePixels"
                                    Value="True" />
                    <Setter Property="LayoutTransform">
                        <Setter.Value>
                            <ScaleTransform CenterX="0.5" CenterY="0.5" ScaleY="{Binding Parent.Zoom}" ScaleX="{Binding Parent.Zoom}"></ScaleTransform>
                        </Setter.Value>
                    </Setter>
                    <Setter Property="ContentTemplate">
                        <Setter.Value>
                            <DataTemplate>
                                <Border x:Name="selectedborder" BorderThickness="1"  CornerRadius="2">
                                    <Grid  >
                                        <Grid.Background>
                                            <ImageBrush ImageSource="{Binding Image}" ></ImageBrush>
                                        </Grid.Background>
                                        <TextBlock Margin="2" Text="{Binding ChargeCode}" Foreground="{StaticResource DarkWarningBrush}" VerticalAlignment="Bottom" HorizontalAlignment="Right" FontSize="20" ></TextBlock>

                                    </Grid>
                                </Border>
                            </DataTemplate>
                        </Setter.Value>
                    </Setter>
                </Style>

                <!--连接线-->
                <Style  x:Key="connectorItemStyle" TargetType="{x:Type ContentPresenter}">
                    <Setter Property="Canvas.ZIndex"
                                    Value="5000" />
                    <Setter Property="att:SelectionProps.EnabledForSelection"
                                    Value="True" />
                    <Setter Property="SnapsToDevicePixels"
                                    Value="True" />
                    <Setter  Property="ContentTemplate">
                        <Setter.Value>
                            <DataTemplate>
                                <arrows:AdjustableArrowBezierCurve  x:Name="arrows"  Stroke="#00C0FF"
                                            IsArrowClosed="True"
                                          ArrowLength="{Binding ArrowLength}"
                                          ArrowEnds="{Binding ConnectorDir,Converter={x:Static con:Direction2ArrowEndsConverter.Instance},UpdateSourceTrigger=PropertyChanged}"     
                                           ShowControl="False"
                                            StrokeThickness="{Binding StrokeThickness}"
                                           StartPoint="{Binding StartPoint}"
                                           EndPoint="{Binding EndPoint}" ControlPoint1="{Binding ControlPoint1,Mode=TwoWay}" ControlPoint2="{Binding ControlPoint2,Mode=TwoWay}">

                                </arrows:AdjustableArrowBezierCurve>
                            </DataTemplate>
                        </Setter.Value>
                    </Setter>
                </Style>

                <!--机械手-->
                <Style x:Key="designerItemmachineStyle"
                                TargetType="{x:Type ContentPresenter}">
                    <Setter Property="Canvas.ZIndex"
                                    Value="3000" />
                    <Setter Property="Canvas.Top"
                                    Value="{Binding Top}" />
                    <Setter Property="Canvas.Left"
                                    Value="{Binding Left}" />
                    <Setter Property="att:SelectionProps.EnabledForSelection"
                                    Value="True" />
                    <Setter Property="att:ItemConnectProps.EnabledForConnection"
                                    Value="False" />
                    <Setter Property="Width"
                                    Value="{Binding ItemWidth}"/>
                    <Setter Property="Height"
                                    Value="{Binding ItemHeight}"/>
                    <Setter Property="SnapsToDevicePixels"
                                    Value="True" />
                    <Setter Property="LayoutTransform">
                        <Setter.Value>
                            <ScaleTransform CenterX="0.5" CenterY="0.5" ScaleY="{Binding Parent.Zoom}" ScaleX="{Binding Parent.Zoom}"></ScaleTransform>
                        </Setter.Value>
                    </Setter>
                    <Setter Property="ContentTemplate">
                        <Setter.Value>
                            <DataTemplate>
                                <Border x:Name="selectedborder" BorderThickness="1"  CornerRadius="2">
                                    <Grid  >
                                        <Grid.Background>
                                            <ImageBrush ImageSource="{Binding Image}" ></ImageBrush>
                                        </Grid.Background>
                                    </Grid>
                                </Border>
                            </DataTemplate>
                        </Setter.Value>
                    </Setter>
                </Style>

                <!--AGV-->
                <Style x:Key="designercarStyle"  TargetType="{x:Type ContentPresenter}">
                    <Setter Property="Canvas.ZIndex"
                                    Value="8000" />
                    <Setter Property="Canvas.Top"
                                    Value="{Binding Top}" />
                    <Setter Property="Canvas.Left"
                                    Value="{Binding Left}" />
                    <Setter Property="att:SelectionProps.EnabledForSelection"
                                    Value="False" />
                    <Setter Property="att:ItemConnectProps.EnabledForConnection"
                                    Value="False" />
                    <Setter Property="Width"
                                    Value="{Binding ItemWidth}"/>
                    <Setter Property="Height"
                                    Value="{Binding ItemHeight}"/>
                    <Setter Property="SnapsToDevicePixels"
                                    Value="True" />
                    <Setter Property="LayoutTransform">
                        <Setter.Value>
                            <ScaleTransform CenterX="0.5" CenterY="0.5" ScaleY="{Binding Parent.Zoom}" ScaleX="{Binding Parent.Zoom}"></ScaleTransform>
                        </Setter.Value>
                    </Setter>
                    <Setter Property="ContentTemplate">
                        <Setter.Value>
                            <DataTemplate>
                                <Grid>
                                    <i:Interaction.Triggers>
                                        <i:EventTrigger EventName="MouseEnter">
                                            <i:InvokeCommandAction Command="{Binding ShowRouteCommand}" CommandParameter="{Binding AgvID}"/>
                                        </i:EventTrigger>
                                        <i:EventTrigger EventName="MouseLeave">
                                            <i:InvokeCommandAction Command="{Binding HiddenRouteCommand}" CommandParameter="{Binding AgvID}"/>
                                        </i:EventTrigger>
                                    </i:Interaction.Triggers> 
                                    <Viewbox>
                                        <Canvas Width="60" Height="30" Margin="10" ClipToBounds="False"  HorizontalAlignment="Center" VerticalAlignment="Center" >
                                            <Border  Background="#CDC8C8" CornerRadius="2" Width="60" Height="30"  />
                                            <Border Width="4" Height="7" BorderThickness="1" BorderBrush="#9499a1" Background="#384a65" Canvas.Left="4" Canvas.Top="5" />
                                            <Border Height="25" Width="2" CornerRadius="1"  Canvas.Top="2" Canvas.Left="-1" Background="Black"/>
                                            <Rectangle  Width="3" Height="4" Fill="#00CC00" HorizontalAlignment="Left"  Canvas.Left="5"  Canvas.Top="15"/>
                                            <Rectangle  Width="3" Height="4" Fill="#F40A0A" HorizontalAlignment="Left"  Canvas.Left="5"  Canvas.Top="20"/>
                                            <Rectangle   Width="30" Height="30" Fill="#987306" RadiusX="5" RadiusY="5" Canvas.Left="20" Canvas.Top="0"/>
                                            <Rectangle   Width="3" Height="30" Fill="#FFCC00" Canvas.Top="0" Canvas.Left="33"/>
                                            <Rectangle   Width="30" Height="3" Fill="#FFCC00" Canvas.Top="14" Canvas.Left="20"/>
                                        </Canvas>
                                    </Viewbox>
                                    <Ellipse Height="60" Width="60" RenderTransformOrigin="0.5 0.5"   >
                                        <Ellipse.Fill>
                                            <SolidColorBrush Color="{Binding StateColor}"></SolidColorBrush>
                                        </Ellipse.Fill>
                                        <Ellipse.OpacityMask>
                                            <RadialGradientBrush>
                                                <GradientStop Offset="0" Color="Transparent" />
                                                <GradientStop Offset="1" Color="Black" />
                                            </RadialGradientBrush>
                                        </Ellipse.OpacityMask>
                                        <Ellipse.RenderTransform>
                                            <ScaleTransform />
                                        </Ellipse.RenderTransform>
                                        <Ellipse.Triggers>
                                            <EventTrigger RoutedEvent="Loaded">
                                                <BeginStoryboard>
                                                    <Storyboard RepeatBehavior="Forever">
                                                        <DoubleAnimation Storyboard.TargetProperty="(UIElement.RenderTransform).(ScaleTransform.ScaleX)" BeginTime="0" Duration="0:0:1" From="1" To="2" EasingFunction="{StaticResource SineEaseOut}" />
                                                        <DoubleAnimation Storyboard.TargetProperty="(UIElement.RenderTransform).(ScaleTransform.ScaleY)" BeginTime="0" Duration="0:0:1" From="1" To="2" EasingFunction="{StaticResource SineEaseOut}" />
                                                        <DoubleAnimation Storyboard.TargetProperty="Opacity" BeginTime="0" Duration="0:0:1" From="1" To="0" EasingFunction="{StaticResource SineEaseOut}" />
                                                    </Storyboard>
                                                </BeginStoryboard>
                                            </EventTrigger>
                                        </Ellipse.Triggers>
                                    </Ellipse>

                                    <Grid Background="Transparent">
                                        <!--<Grid.ToolTip>
                                            <ToolTip></ToolTip>
                                        </Grid.ToolTip>-->
                                        <TextBlock Text="{Binding AgvID}" FontSize="30" HorizontalAlignment="Center" VerticalAlignment="Center"></TextBlock>
                                    </Grid>
                                </Grid>
                               
                            </DataTemplate>
                        </Setter.Value>
                    </Setter>
                </Style>
            </ItemsControl.Resources>
            <ItemsControl.ItemsPanel>
                <ItemsPanelTemplate>
                    <ucdp:DesignerCanvas Background="Transparent" >
                    </ucdp:DesignerCanvas>
                </ItemsPanelTemplate>
            </ItemsControl.ItemsPanel>
        </ItemsControl>
    </Border>
</UserControl>

```  
上面代码中，我们定义一个ItemControl,并且指定ItemPanel为一个Canvas。  
接着我们定义自己的StyleSelector:  

```csharp
public class DesignerItemsControlItemStyleSelector : StyleSelector
    {
        static DesignerItemsControlItemStyleSelector()
        {
            Instance = new DesignerItemsControlItemStyleSelector();
        }

        public static DesignerItemsControlItemStyleSelector Instance
        {
            get;
            private set;
        }


        public override Style SelectStyle(object item, DependencyObject container)
        {
            ItemsControl itemsControl = ItemsControl.ItemsControlFromItemContainer(container);
            if (itemsControl == null)
                throw new InvalidOperationException("DesignerItemsControlItemStyleSelector : Could not find ItemsControl");

            if (item is DesignerItemLandMarkViewModel)
            {
                return (Style)itemsControl.FindResource("designerItemlandmarkStyle");
            }
            if (item is ConnectorViewModel)
            {
                return (Style)itemsControl.FindResource("connectorItemStyle");
            }
            if (item is DesignerItemStorageViewModel)
            {
                return (Style)itemsControl.FindResource("designerItemstorageStyle");
            }
            if (item is DesignerItemTextViewModel)
            {
                return (Style)itemsControl.FindResource("designerItemtextStyle");
            }
            if (item is DesignerItemImageViewModel)
            {
                return (Style)itemsControl.FindResource("designerItemimageStyle");
            }
            if (item is DesignerItemChargeViewModel)
            {
                return (Style)itemsControl.FindResource("designerItemchargeStyle");
            }
            if (item is DesignerItemMachineViewModel)
            {
                return (Style)itemsControl.FindResource("designerItemmachineStyle");
            }
            if (item is DesignerItemCarViewModel)
            {
                return (Style)itemsControl.FindResource("designercarStyle");
            }
            return null;
        }
    }
```  
最终运行效果如下图：   
![图片2](/assets/bandicam20200530215254.gif)