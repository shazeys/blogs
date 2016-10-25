# 关于Rxjava理解

标签：rxjava 理解

根据我和朋友的聊天来说，Rxjava应用在Android是最广的，反倒是java方面，似乎不怎么被认同？因为我没做过什么java的项目，所以还是从Android的角度来说Rxjava吧。

---

###观察者模式

Android中，最常见的观察者模式就是View类的setOnClickListener了。
举个开关和光照的例子：选择不同颜色的开关，就会有不同颜色的光照射出去：

    public class Switch {
	    private String colorName;//开关的颜色
	    private Light light;//灯


	    public String getColorName() {
		    return colorName;
	    }

	    public void setColorName(String colorName) {
		    this.colorName = colorName;
		    if(light != null)
			    light.showLightColor(colorName);
	    }

	    //设置灯
	    public void setLight(Light light) {
		    this.light = light;
	    }

	    //定义一个灯照的接口
	    public interface Light{
		    void showLightColor(String color);//显示灯照结果
	    }
    }
    public class GcTest {
	
    	public static void main(String[] args) {
    		Light wall = new Light() {
    			public void showLightColor(String color) {
    				System.out.println(color+"灯光照在墙上");
    			}
    		};
    		Light table = new Light() {
    			public void showLightColor(String color) {
    				System.out.println(color+"灯光照在桌子上");
    			}
    		};
    		
    		Switch s = new Switch();
    		s.setLight(wall);
    		s.setColorName("红色");
    		s.setColorName("蓝色");
    		System.out.println("=================");
    
    		s.setLight(table);
    		s.setColorName("红色");
    		s.setColorName("蓝色");
    	}
    }
运行结果：![运行结果](https://github.com/shazeys/blogs/blob/master/Android/images/gc_result.png)

----
###向Rxjava迈出第一步
沿着上一步的套路，搞个通用点的观察者模式，我们可以试着写个泛型，可以用来操作不同的类：
    
    //包装类
    public class Wrapper<T> {
    	private T t;
    	public void nextStep(Dispose d) {
    		// 因为我们希望我们的操作可以自由一点，
    		// 所以我们把值抛给一个接口来处理
    		d.doSomeThing(t);
    	}
    	public Wrapper(T t) {
    		super();
    		this.t = t;
    	}
    }
    //操作接口
    public interface Dispose<P> {
    	void doSomeThing(P p);
    }
    //准备个实体类
    public class User {
    	private String name;
    	private int age;
    	private String hobby;
    	
    	public User(String name, int age, String hobby) {
    		super();
    		this.name = name;
    		this.age = age;
    		this.hobby = hobby;
    	}
    	public String getName() {
    		return name;
    	}
    	public void setName(String name) {
    		this.name = name;
    	}
    	public int getAge() {
    		return age;
    	}
    	public void setAge(int age) {
    		this.age = age;
    	}
    	public String getHobby() {
    		return hobby;
    	}
    	public void setHobby(String hobby) {
    		this.hobby = hobby;
    	}
    	@Override
    	public String toString() {
    		return "User：[name=" + name + ", age=" + age + ", hobby=" + hobby + "]";
    	}
    }
    //测试类
    public class Test {
    	public static void main(String[] args) {
    		Wrapper w = new Wrapper(new User("小明", 16, "罚站"));
    		w.nextStep(new Dispose<User>() {
    			public void doSomeThing(User p) {
    				//打印小明的信息
    				System.out.println(p.toString());
    				//给小明长一岁
    				p.setAge(p.getAge()+1);
    				//给小明换个还好
    				p.setHobby("撩妹");
    				System.out.println(p.toString());
    			}
    		});
    	}
    }

放图片太麻烦，我粘贴一下结果好了：
    User：[name=小明, age=16, hobby=罚站]
    User：[name=小明, age=17, hobby=撩妹]

---
###Rxjava更近一步

到了这里，我们可以看到，通过这个包装类和操作接口，我们可以用这样的形式，做很多自由的操作，但是如果我们想将上面的操作后的实体进行下一步的使用怎么办呢？只需要改造一下这两个类：

    //为了使代码更方便，我们给Wrapper抽出一个接口：
    public interface IWrapper<T> {
	    IWrapper<T> nextStep(Dispose t);
    }
    //改造后的Wrapper
    public class Wrapper<T> implements IWrapper<T>{
    	private T t;
    	public IWrapper<T> nextStep(Dispose d) {
    		// TODO Auto-generated method stub
    		return d.doSomeThing(t);
    	}
    	public Wrapper(T t) {
    		super();
    		this.t = t;
    	}
    }
    //改造后的操作类
    //P代表parameter，R代表return、result
    public interface Dispose<P,R> {
    	IWrapper<R> doSomeThing(P p);
    }
接下来就是可以链式调用的测试类：
    
    //测试类
    public class Test {
    	public static void main(String[] args) {
    		Wrapper<User> w = new Wrapper(new User("小明", 16, "罚站"));
    		w.nextStep(new Dispose<User, User>() {//和前面的示例一样
    			public IWrapper<User> doSomeThing(User p) {
    				//打印小明的信息
    				System.out.println(p.toString());
    				//给小明长一岁
    				p.setAge(p.getAge()+1);
    				//给小明换个还好
    				p.setHobby("撩妹");
    				System.out.println(p.toString());
    				//打印小分割线
    				System.out.println("===============");
    				return new Wrapper<User>(p);
    			}
    		})
    		.nextStep(new Dispose<User, User>() { //一组新的逻辑，建立在前面的结果之上
    			public IWrapper<User> doSomeThing(User p) {
    				//打印小明初始的信息
    				System.out.println(p.toString());
    				// 给小明换个名字，改成30岁，并且修改爱好
    				p.setName("韩梅梅");
    				p.setAge(30);
    				p.setHobby("撩汉");
    				//打印小明修改的信息
    				System.out.println(p.toString());
    				return new Wrapper<User>(p);
    			}
    		})
    		.nextStep(null)//如果需要，继续其他逻辑
    		;
    	}
    }
    
运行结果：![结果](https://github.com/shazeys/blogs/blob/master/Android/images/brx_result.png)

到了这里，我们基本上通过观察者模式，写出了一个自己的类似的Rxjava的链式玩意，这里只是说明一下大概意思。

而真实的Rxjava提供的大量的操作方法，各种吊炸天

说了这些我也不知道我要说什么了，好了，就到这吧。
