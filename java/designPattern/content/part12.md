### 设计模式 外观模式 一键电影模式

这个模式比较简单，嘿嘿，简单写一下。

老样子，先看 外观模式（Facade Pattern）定义：提供一个统一的接口，用来访问子系统中的一群接口，外观定义了一个高层的接口，让子系统更容易使用。其实就是为了方便客户的使用，把一群操作，封装成一个方法。

举个例子：我比较喜欢看电影，于是买了投影仪、电脑、音响、设计了房间的灯光、买了爆米花机，然后我想看电影的时候，我需要：

1、打开爆米花机

2、制作爆米花

3、将灯光调暗

4、打开投影仪

5、放下投影仪投影区

6、打开电脑

7、打开播放器

8、将播放器音调设为环绕立体声

...

尼玛，花了一笔钱，看电影还要这么多的步骤，太累了，而且看完还要一个一个关掉。

所有，我们使用外观模式解决这些复杂的步骤，轻松享受电影：

```
public class HomeTheaterFacade
{
	private Computer computer;
	private Player player;
	private Light light;
	private Projector projector;
	private PopcornPopper popper;
 
	public HomeTheaterFacade(Computer computer, Player player, Light light, Projector projector, PopcornPopper popper)
	{
		this.computer = computer;
		this.player = player;
		this.light = light;
		this.projector = projector;
		this.popper = popper;
	}
 
	public void watchMovie()
	{
		/**
		 *  1、打开爆米花机
			2、制作爆米花
			3、将灯光调暗
			4、打开投影仪
			5、放下投影仪投影区
			6、打开电脑
			7、打开播放器
			8、将播放器音调设为环绕立体声
		 */
		popper.on();
		popper.makePopcorn();
		light.down();
		projector.on();
		projector.open();
		computer.on();
		player.on();
		player.make3DListener();
	}
	
	public void stopMovie()
	{
		popper.off();
		popper.stopMakePopcorn();
		light.up();
		projector.close();
		projector.off();
		player.off();
		computer.off();
	}
}
```

可以看到，我们定义了一个类，然后可以让我一键享受看电影了，看完，一键关闭，享受多了。
外观模式：一般用于需要简化一个很大的接口，或者一群复杂的接口的时候。

这个模式比较容易理解，就不多说了，最后附上类图：


好了，恭喜你，你又学会了一个设计模式，外观模式（Facade Pattern）。
