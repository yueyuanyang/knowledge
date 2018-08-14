## 设计模式 命令模式

### 设计模式 命令模式 之 管理智能家电

继续设计模式哈，今天带来命令模式，二话不说，先看定义：

定义：将“请求”封装成对象，以便使用不同的请求、队列或者日志来参数化其他对象。命令模式也支持可撤销的操作。

这尼玛定义，看得人蛋疼，看不明白要淡定，我稍微简化一下：将请求封装成对象，将动作请求者和动作执行者解耦。好了，直接用例子来说明。

需求：最近智能家电很火热啊，未来尼玛估计冰箱都会用支付宝自动买东西了，，，，假设现在有电视、电脑、电灯等家电，现在需要你做个遥控器控制所有家电的开关，要求做到每个按钮对应的功能供用户个性化，对于新买入家电要有非常强的扩展性。

这个需求一看，尼玛要是没有什么个性化、扩展性还好说啊，直接针对每个遥控器的按钮onClick，然后在里面把代码写死就搞定了，但是个性化怎么整，还要有扩展性。。。

好了，下面命令模式出场，命令模式的核心就是把命令封装成类，对于命令执行者不需要知道现在执行的具体是什么命令。

#### 1、首先看下我们拥有的家电的API：
```
/**
 * 门
 * @author zhy
 *
 */
public class Door
{
	public void open()
	{
		System.out.println("打开门");
	}
 
	public void close()
	{
		System.out.println("关闭门");
	}
 
}

```
```
/**
 * 电灯
 * @author zhy
 *
 */
public class Light
{
	public void on()
	{
		System.out.println("打开电灯");
	}
 
	public void off()
	{
		System.out.println("关闭电灯");
	}
}
```

```
/**
 * 电脑
 * @author zhy
 *
 */
public class Computer
{
	public void on()
	{
		System.out.println("打开电脑");
	}
	
	public void off()
	{
		System.out.println("关闭电脑");
	}
}

```
**看来我们有电灯、电脑、和门，并且开关的接口的设计好了。接下来看如何把命令封装成类:**

```
public interface Command
{
	public void execute();
}

```

```

/**
 * 关闭电灯的命令
 * @author zhy
 *
 */
public class LightOffCommond implements Command
{
	private Light light ; 
	
	public LightOffCommond(Light light)
	{
		this.light = light;
	}
 
	@Override
	public void execute()
	{
		light.off();
	}
 
}

```

```
/**
 * 打开电灯的命令
 * @author zhy
 *
 */
public class LightOnCommond implements Command
{
	private Light light ; 
	
	public LightOnCommond(Light light)
	{
		this.light = light;
	}
 
	@Override
	public void execute()
	{
		light.on();
	}
 
}

```

```
/**
 * 开电脑的命令
 * @author zhy
 *
 */
public class ComputerOnCommond implements Command
{
	private Computer computer ; 
	
	public ComputerOnCommond( Computer computer)
	{
		this.computer = computer;
	}
 
	@Override
	public void execute()
	{
		computer.on();
	}
 
}

```

```

/**
 * 关电脑的命令
 * @author zhy
 *
 */
public class ComputerOffCommond implements Command
{
	private Computer computer ; 
	
	public ComputerOffCommond( Computer computer)
	{
		this.computer = computer;
	}
 
	@Override
	public void execute()
	{
		computer.off();
	} 
}

```

好了，不贴那么多了，既然有很多命令，按照设计原则，我们肯定有个超类型的Command，然后各个子类，看我们把每个命令（请求）都封装成类了。接下来看我们的遥控器。

```
**
 * 控制器面板，一共有9个按钮
 * 
 * @author zhy
 * 
 */
public class ControlPanel
{
	private static final int CONTROL_SIZE = 9;
	private Command[] commands;
 
	public ControlPanel()
	{
		commands = new Command[CONTROL_SIZE];
		/**
		 * 初始化所有按钮指向空对象
		 */
		for (int i = 0; i < CONTROL_SIZE; i++)
		{
			commands[i] = new NoCommand();
		}
	}
 
	/**
	 * 设置每个按钮对应的命令
	 * @param index
	 * @param command
	 */
	public void setCommand(int index, Command command)
	{
		commands[index] = command;
	}
 
	/**
	 * 模拟点击按钮
	 * @param index
	 */
	public void keyPressed(int index)
	{
		commands[index].execute();
	}
 
}
```

```
/**
 * @author zhy
 *
 */
public class NoCommand implements Command
{
	@Override
	public void execute()
	{
 
	}
 
}

```
注意看到我们的遥控器有9个按钮，提供了设置每个按钮的功能和点击的方法，还有注意到我们使用了一个NoCommand对象，叫做空对象，这个对象的好处就是，我们不用执行前都判断个if(!=null)，并且提供了一致的操作。

最后测试一下代码：

```
public class Test
{
	public static void main(String[] args)
	{
		/**
		 * 三个家电
		 */
		Light light = new Light();
		Door door = new Door();
		Computer computer = new Computer();
		/**
		 * 一个控制器，假设是我们的app主界面
		 */
		ControlPanel controlPanel = new ControlPanel();
		// 为每个按钮设置功能
		controlPanel.setCommand(0, new LightOnCommond(light));
		controlPanel.setCommand(1, new LightOffCommond(light));
		controlPanel.setCommand(2, new ComputerOnCommond(computer));
		controlPanel.setCommand(3, new ComputerOffCommond(computer));
		controlPanel.setCommand(4, new DoorOnCommond(door));
		controlPanel.setCommand(5, new DoorOffCommond(door));
 
		// 模拟点击
		controlPanel.keyPressed(0);
		controlPanel.keyPressed(2);
		controlPanel.keyPressed(3);
		controlPanel.keyPressed(4);
		controlPanel.keyPressed(5);
		controlPanel.keyPressed(8);// 这个没有指定，但是不会出任何问题，我们的NoCommand的功劳
	}
}
```

可以看到任意按钮可以随意配置任何命令，再也不需要尼玛的变一下需求改代码了，随便用户怎么个性化了。其实想白了，这里的设置我们还可以配置到一个配置文件中，完全的解耦有木有。



好了，用户对于这个按钮可能还不是太满意，用户希望夜深人静的时候，能够提供个按钮直接关门、关灯、开电脑，，，，大家懂的，，，我们稍微修改下代码，满足他

定义一个命令，用户干一些列的事，可配置，且与原来的命令保持接口一致：

```
/**
 * 定义一个命令，可以干一系列的事情
 * 
 * @author zhy
 * 
 */
public class QuickCommand implements Command
{
	private Command[] commands;
 
	public QuickCommand(Command[] commands)
	{
		this.commands = commands;
	}
 
	@Override
	public void execute()
	{
		for (int i = 0; i < commands.length; i++)
		{
			commands[i].execute();
		}
	}
 
}


```

好了，已经满足屌丝的需求了。我们测试看看。

```
// 定义一键搞定模式
		QuickCommand quickCommand = new QuickCommand(new Command[] { new DoorOffCommond(door),
				new LightOffCommond(light), new ComputerOnCommond(computer) });
		System.out.println("****点击一键搞定按钮****");
		controlPanel.setCommand(8, quickCommand);
		controlPanel.keyPressed(8);
```

是不是很完美。



最后，继续来谈谈命令模式，命令模式就是把命令封装成对象，然后将动作请求者与动作执行者完全解耦，上例中遥控器的按钮和电器一毛钱关系都没吧。

还记得定义中提到了队列，命令模式如何用于队列呢，比如饭店有很多个点菜的地方，有一个做菜的地方，把点菜看作命令，做菜看作命令执行者，不断有人点菜就相当于把菜加入队列，对于做菜的只管从队列里面取，取一个做一个。

定义中还提到了日志，日志一般用于记录用户行为，或者在异常时恢复时用的，比如每个命令现在包含两个方法，一个执行execute，一个undo（上例中为了方便大家理解，没有写undo），我们可以把用户所有命令调用保存到日志中，比如用户操作不当了，电器异常了，只需要把日志中所有的命令拿出来执行一遍undo就完全恢复了，是吧，就是这么个意思。




