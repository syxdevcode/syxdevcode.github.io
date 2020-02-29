---
title: DotNet内存泄漏(Memory Leak)-事件(Event)
date: 2018-12-13 11:06:53
tags:
- DotNet
- CSharp
- 内存泄漏(Memory Leak)
- 委托与事件
- CSharp基础
categories: 
- 委托与事件
---
# DotNet内存泄漏(Memory Leak)-事件(Event)

通过[事件(Event)，绝大多数内存泄漏（Memory Leak）的元凶[上篇]](http://www.cnblogs.com/artech/archive/2009/12/03/1616507.html)这篇文章的实例，学习内存泄漏。

[代码下载](https://pan.baidu.com/s/1_zrGMRwiyB6Kky8t5kgFlA) 
提取码：xazz

![微信截图_20181213135838.png](/img/微信截图_20181213135838.png)

&emsp;&emsp;我们这个应用程序叫做TodoListManager，因为通过它可以实时查看属于用户的“待办事宜（Todolist）”。这是一个GUI的应用，有两个Windows Form组成：左侧的窗体是一个程序的主界面（为了简单起见，我甚至没有将其做成MDI窗体），点击Todo List菜单项，右面的Form被显示出来：所有的代码事宜将会全部列出，为了保证记录的实时显示，每隔5秒钟数据自动刷新一次。

&emsp;&emsp;首先定义表示每一项TotoList Item定义了一个相应的类型：MyEvent（不是我们谈到的导致内存泄漏的事件）。Event仅仅包含简单的属性：主题（Subject），截至日期（DueDate）和相应的描述性文字（Description），Event定义如下：

```csharp
namespace WindowsFormsApp1
{
    public class MyEvent
    {
        public string Subject { get; set; }
        public DateTime DueDate { get; set; }
        public string Description { get; set; }
        public MyEvent(string subject, DateTime dueDate, string desc)
        {
            if (string.IsNullOrEmpty(subject))
            {
                throw new ArgumentNullException("subject");
            }
            this.Subject = subject;
            this.DueDate = dueDate;
            this.Description = desc ?? string.Empty;
        }
    }
}
```

&emsp;&emsp;然后将所有逻辑（实际上仅仅是定期获取TodoList列表而已）定义在下面一个叫做TodoListManager的类型中。将其定义成Singleton的形式，并采用`System.Threading.Timer`实现定时地获取Todo List的操作。

```csharp
namespace WindowsFormsApp1
{
    public class TodoListManager
    {
        private static readonly TodoListManager instance = new TodoListManager();
        public event EventHandler<TodoListEventArgs> TodoListChanged;
        private Timer todoListRefreshSchedler;
        private TodoListManager()
        {
            todoListRefreshSchedler = new Timer
            (
                state =>
                {
                    if (null == TodoListChanged)
                    {
                        return;
                    }

                    TodoListChanged(null, new TodoListEventArgs(GetTodolist()));
                }
           , null, 0, 5000);
        }
        public static TodoListManager Instance
        {
            get
            {
                return instance;
            }
        }
        private List<MyEvent> GetTodolist()
        {
            var list = new List<MyEvent>();
            list.Add(new MyEvent("Meeting with Testing Team", DateTime.Today.AddDays(2), "NIL"));
            list.Add(new MyEvent("Deliver progress report to manager ", DateTime.Today.AddDays(7), "NIL"));
            return list;
        }
    }
}
```

&emsp;&emsp;对于Timer的每一个轮询，都会处触发一个类型为`EventHandler<TodoListEventArgs>`的事件，通过注册这个事件，可以通过类型为TodoListEventArgs的事件参数得到最新的TodoList的列表，TodoListEventArgs定义如下：

```csharp
public class TodoListEventArgs : EventArgs
{
    public IEnumerable<MyEvent> TodoList
    { get; private set; }
    public TodoListEventArgs(IEnumerable<MyEvent> todoList)
    {
        if (null == todoList)
        {
            throw new ArgumentNullException("todoList");
        }

        this.TodoList = todoList;
    }
}
```

&emsp;&emsp;在窗体Load的时候注册TodoListManager的TodoListChanged事件，并将获取到的TodoList列表绑定到DataGridView上面。由于TodoListManager异步工作的原因，借助了SynchronizationContext这么一个对象实现对数据的绑定。

```csharp
using System;
using System.Threading;
using System.Windows.Forms;

namespace WindowsFormsApp1
{
    public partial class ToDoListForm : Form
    {
        public static SynchronizationContext SynchronizationContext { get; private set; }

        public ToDoListForm()
        {
            InitializeComponent();
        }

        private void ToDoListForm_Load(object sender, EventArgs e)
        {
            SynchronizationContext = SynchronizationContext.Current;
            TodoListManager.Instance.TodoListChanged += TodoListManager_TodoListChanged;
        }

        private void TodoListManager_TodoListChanged(object sender, TodoListEventArgs e)
        {
            SynchronizationContext.Post(
                state =>
                {
                    BindingSource bindingSource = new BindingSource();
                    bindingSource.DataSource = e.TodoList;
                    this.dataGridViewTodoList.DataSource = bindingSource;
                }, null);
        }
    }
}
```

在整个应用级别定义了一个静态的System.Threading.Timer，让它每隔半秒调用一次GC.Collect()。

```csharp
using System;
using System.Windows.Forms;

namespace WindowsFormsApp1
{
    internal static class Program
    {
        private static System.Threading.Timer gcScheduler = new System.Threading.Timer
            (state => GC.Collect(), null, 0, 500);

        /// <summary>
        /// 应用程序的主入口点。
        /// </summary>
        [STAThread]
        private static void Main()
        {
            Application.EnableVisualStyles();
            Application.SetCompatibleTextRenderingDefault(false);
            Application.Run(new MainForm());
        }
    }
}
```

查看内存泄漏，需要借助响应的Memory Profiling工具:

JetBrains的`dotTrace`（免费），
RedGate的`ANTS Memory Profiler`(收费)

&emsp;&emsp;ANTS Memory Profiler通过这样的原理来确定你的应用程序是否有泄漏问题：如果你怀疑某个操作会导致应该被GC回收的对象没有被回收，那么你在之前对内存分配情况拍一张快照（Snapshot），然后执行该操作，在操作完成并确定GC完成相应的回收操作后，在拍一张快照。通过对比，找出多余的对象，并根据具体的情况分析该对象是否应该被GC回收，如果是的，怎意味着你的程序存在着内存泄漏问题。

通过ANTS Memory Profiler启动我们的应用程序后，在一开始的时候我们拍摄一张反映程序初始状态的内存快照，然后选择File\Todo List打开TodoListForm，等待一定的时间，再将TodoListForm关闭。为了让GC有充分的时间进行垃圾回收，不妨再作相应的等待，然后拍下第二张快照。在Class List视图中，你会发现原本应该被垃圾回收的TodoListForm窗体对象还存在于内存之中。

![微信截图_20181213144749.png](/img/微信截图_20181213144749.png)

&emsp;&emsp;通过上图可以看得，该对象被TodoListManager的一个类型为`EventHandler<TodoListEventArgs>`的事件引用，这个对象实际上是一个Delegate对象，而TodoListForm作为这个Delegate对象的Target。通过上面给出的代码，我们不难想出是由于在TodoListForm实现了对TodoListManager的TotoListChanged事件注册导致了TodoListManager不能被垃圾回收。

&emsp;&emsp;对于GUI应用可视化树形结构来说，一个窗体被关闭，照例说它应该成为垃圾对象，GC在执行垃圾回收的时候就可以将其清楚的。但是，由于该对象注册了一个事件到一个生命周期很长的对象（在本例中，TodoManager是一个Singletone对象，具有和整个应用程序一样的生命周期），它就是被这么一个对象长期引用，进而阻止 GC对其的回收工作。

&emsp;&emsp;短暂生命周期注册事件到长期生命周期对象上，在该对象被Dispose的时候，应该解除事件的注册。你可以通过实现System.IDisposable接口，将解除事件注册的操作放在Dispose方法中。对于本里来说，你可以将相应的操作注册到Form的Closing、Closed或者Disposed事件中。比如在下面代码中，我为TodoListForm添加了如下一个Closing事件处理程序：

```csharp
using System;
using System.Threading;
using System.Windows.Forms;

namespace WindowsFormsApp1
{
    public partial class ToDoListForm : Form
    {
        public static SynchronizationContext SynchronizationContext { get; private set; }

        public ToDoListForm()
        {
            InitializeComponent();
        }

        private void ToDoListForm_Load(object sender, EventArgs e)
        {
            SynchronizationContext = SynchronizationContext.Current;
            TodoListManager.Instance.TodoListChanged += TodoListManager_TodoListChanged;
        }

        private void TodoListManager_TodoListChanged(object sender, TodoListEventArgs e)
        {
            SynchronizationContext.Post(
                state =>
                {
                    BindingSource bindingSource = new BindingSource();
                    bindingSource.DataSource = e.TodoList;
                    this.dataGridViewTodoList.DataSource = bindingSource;
                }, null);
        }

        private void ToDoListForm_FormClosing(object sender, FormClosingEventArgs e)
        {
            TodoListManager.Instance.TodoListChanged -= TodoListManager_TodoListChanged;
        }
    }
}
```

修改之后，按照上面的流程利用`ANTS Memory Profiler`在第二个快照中，你将再也看不到TodoListForm的身影（如下图）。

![微信截图_20181213145711.png](/img/微信截图_20181213145711.png)

参考：

[事件(Event)，绝大多数内存泄漏（Memory Leak）的元凶[上篇]](http://www.cnblogs.com/artech/archive/2009/12/03/1616507.html)