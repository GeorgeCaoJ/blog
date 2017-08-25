---
layout: pageshow
title: 学习WPF和MVVM一段时间的总结
category: CSharp
description: 学习MVVM设计模式后第一次用于生产
---

## WPF的MVVM设计模式

从**winform**转变到**WPF**的过程，难点主要还是在MVVM的设计模式。当然，如果依然采用winform的涉及方式，在每个控件背后绑定事件的方式运用在wpf中，依然可行，但是假如GUI改版，其背后绑定的特别为此界面设计的事件不得不多数弃用。而MVVM最大的好处是将一切业务逻辑放在**ViewModel**中 ，将GUI的操作放在**view**中，将数据结构放在**Model**中，如图摘自[MSDN](https://msdn.microsoft.com/en-us/library/hh848246.aspx)  
![MVVM1]({{site.baseurl}}/img/CSharp/MVVM1.jpg)

## 实际使用

使用了[Prism](https://github.com/PrismLibrary/Prism)框架，省去了去构造实现`INotifyPropertyChanged`的基类，直接继承`BindableBase`

```csharp
namespace Prism.Mvvm
{
    //
    // 摘要:
    //     Implementation of System.ComponentModel.INotifyPropertyChanged to simplify models.
    public abstract class BindableBase : INotifyPropertyChanged
    {
        protected BindableBase();

        //
        // 摘要:
        //     Occurs when a property value changes.
        public event PropertyChangedEventHandler PropertyChanged;

        //
        // 摘要:
        //     Notifies listeners that a property value has changed.
        //
        // 参数:
        //   propertyName:
        //     Name of the property used to notify listeners. This value is optional and can
        //     be provided automatically when invoked from compilers that support System.Runtime.CompilerServices.CallerMemberNameAttribute.
        protected virtual void OnPropertyChanged([CallerMemberName] string propertyName = null);
        //
        // 摘要:
        //     Raises this object's PropertyChanged event.
        //
        // 参数:
        //   propertyExpression:
        //     A Lambda expression representing the property that has a new value.
        //
        // 类型参数:
        //   T:
        //     The type of the property that has a new value
        protected virtual void OnPropertyChanged<T>(Expression<Func<T>> propertyExpression);
        //
        // 摘要:
        //     Checks if a property already matches a desired value. Sets the property and notifies
        //     listeners only when necessary.
        //
        // 参数:
        //   storage:
        //     Reference to a property with both getter and setter.
        //
        //   value:
        //     Desired value for the property.
        //
        //   propertyName:
        //     Name of the property used to notify listeners. This value is optional and can
        //     be provided automatically when invoked from compilers that support CallerMemberName.
        //
        // 类型参数:
        //   T:
        //     Type of the property.
        //
        // 返回结果:
        //     True if the value was changed, false if the existing value matched the desired
        //     value.
        protected virtual bool SetProperty<T>(ref T storage, T value, [CallerMemberName] string propertyName = null);
    }
```
以及用来构造**Command**的基类`DelegateCommand`

```csharp
namespace Prism.Commands
{
    //
    // 摘要:
    //     An System.Windows.Input.ICommand whose delegates do not take any parameters for
    //     Prism.Commands.DelegateCommand.Execute and Prism.Commands.DelegateCommand.CanExecute.
    public class DelegateCommand : DelegateCommandBase
    {
        //
        // 摘要:
        //     Creates a new instance of Prism.Commands.DelegateCommand with the System.Action
        //     to invoke on execution.
        //
        // 参数:
        //   executeMethod:
        //     The System.Action to invoke when System.Windows.Input.ICommand.Execute(System.Object)
        //     is called.
        public DelegateCommand(Action executeMethod);
        //
        // 摘要:
        //     Creates a new instance of Prism.Commands.DelegateCommand with the System.Action
        //     to invoke on execution and a Func to query for determining if the command can
        //     execute.
        //
        // 参数:
        //   executeMethod:
        //     The System.Action to invoke when System.Windows.Input.ICommand.Execute(System.Object)
        //     is called.
        //
        //   canExecuteMethod:
        //     The System.Func`1 to invoke when System.Windows.Input.ICommand.CanExecute(System.Object)
        //     is called
        public DelegateCommand(Action executeMethod, Func<bool> canExecuteMethod);
        protected DelegateCommand(Func<Task> executeMethod);
        protected DelegateCommand(Func<Task> executeMethod, Func<bool> canExecuteMethod);

        //
        // 摘要:
        //     Factory method to create a new instance of Prism.Commands.DelegateCommand from
        //     an awaitable handler method.
        //
        // 参数:
        //   executeMethod:
        //     Delegate to execute when Execute is called on the command.
        //
        // 返回结果:
        //     Constructed instance of Prism.Commands.DelegateCommand
        public static DelegateCommand FromAsyncHandler(Func<Task> executeMethod);
        //
        // 摘要:
        //     Factory method to create a new instance of Prism.Commands.DelegateCommand from
        //     an awaitable handler method.
        //
        // 参数:
        //   executeMethod:
        //     Delegate to execute when Execute is called on the command. This can be null to
        //     just hook up a CanExecute delegate.
        //
        //   canExecuteMethod:
        //     Delegate to execute when CanExecute is called on the command. This can be null.
        //
        // 返回结果:
        //     Constructed instance of Prism.Commands.DelegateCommand
        public static DelegateCommand FromAsyncHandler(Func<Task> executeMethod, Func<bool> canExecuteMethod);
        //
        // 摘要:
        //     Determines if the command can be executed.
        //
        // 返回结果:
        //     Returns true if the command can execute, otherwise returns false.
        public virtual bool CanExecute();
        //
        // 摘要:
        //     Executes the command.
        public virtual Task Execute();
        //
        // 摘要:
        //     Observes a property that is used to determine if this command can execute, and
        //     if it implements INotifyPropertyChanged it will automatically call DelegateCommandBase.RaiseCanExecuteChanged
        //     on property changed notifications.
        //
        // 参数:
        //   canExecuteExpression:
        //     The property expression. Example: ObservesCanExecute((o) => PropertyName).
        //
        // 返回结果:
        //     The current instance of DelegateCommand
        public DelegateCommand ObservesCanExecute(Expression<Func<object, bool>> canExecuteExpression);
        //
        // 摘要:
        //     Observes a property that implements INotifyPropertyChanged, and automatically
        //     calls DelegateCommandBase.RaiseCanExecuteChanged on property changed notifications.
        //
        // 参数:
        //   propertyExpression:
        //     The property expression. Example: ObservesProperty(() => PropertyName).
        //
        // 类型参数:
        //   T:
        //     The object type containing the property specified in the expression.
        //
        // 返回结果:
        //     The current instance of DelegateCommand
        public DelegateCommand ObservesProperty<T>(Expression<Func<T>> propertyExpression);
    }
}
```

第一次使用MVVM设计模式，没有理解在多个**ViewModel**之间通信，所以不得不采用单一**ViewModel**和多个**view**，这样导致了一个ViewModel的臃肿和复杂，看似结构简单，但是实际的逻辑越来越混乱。在没有理解[Event Aggregator](https://msdn.microsoft.com/en-us/library/ff921122.aspx)如何使用的情况下，这是可行方案。  

## MVVM使用感受

最主要的感受是**MVVM**将UI和业务逻辑分离，UI就只写UI，不用像**WinForm**一样在每个事件背后，如果要获得UI某个`TextBox`的数据，得通过如下获取

```csharp
public void SomeButton_Clicked(object sender, EventArgs e)
{
	string text = textBox1.Text;
	DoSomeThings(text);
	...
}
```

同样，后台事件要更新前台UI数据时

```csharp
pubic void SomeButton_Clicked(object sender, EvnetArgs e)
{
	DoOtherThings();
	textBox1.Text = "Some Text";
}
```

这种硬编码的形式，遇到UI的重大变化，必须就将背后事件对应UI的控件名称全部更改才能继续运行。当软件达到一定复杂度，这样做就是灾难性的。  

而**MVVM**，使用了数据绑定，虽然增加了一点代码，但是带来的好处巨大。在**ViewModel**中先定义要绑定的数据

```csharp
private string name;
public string Name{
	get{return name;}
	set{
		if (name != value)
		{
			name = value;
			OnPropertyChanged("Name"); // 实现INotifyPropertyChanged接口
	}
}
}
```

然后在**view**中将其和**TextBox**数据绑定

```csharp
<TextBox Text="{Binding Name, Mode=TwoWay}">
```

这里的数据绑定方式是双向绑定，后台数据变化会自动通知前台UI线程更新数据，相反，前台UI线程更改了数据，后台的数据也会相应变化。这样，在实际数据更新时，不用去查看绑定的UI控件名称，也不用担心在其他线程更新控件数据时要用`oneControl.Invoke(Action action)`。

## 总结

第一次使用**MVVM**感受到的优点：

* 数据绑定，不用考虑具体UI控件的名称，不用手动更新UI数据。

* UI可操作性更大，支持**template**

* 业务逻辑和UI分离，界面改版方便

但同样带来了缺点：

* 代码量明显增加

* 对于小软件来说，开发没有**WinFrom**来的敏捷



