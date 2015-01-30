---
layout:     post
title:      "如何在iOS中调用Lua"
subtitle:   "OC和Lua交互的原理过程，新手级别"
date:       2015-01-30 12:00:00
author:     "Shen Quan"
header-img: "img/post-bg-01.jpg"

---
# 如何在iOS中调用Lua

最近接触了Lua脚本语言，对iOS来说是个好东西，它可以以脚本形式调用iOS应用中你所写的函数，这使得在不升级应用的情况下改变内部业务逻辑成为可能，相关的框架在网上能搜索到，[**Wax**](https://github.com/probablycorey/wax)是比较有名气的，不过目前已经无人维护，其实，你也可以自己对Lua进行封装。

封装的事，以后再说，对于没接触过Lua的iOS同行来说，对Lua是如何在iOS中运行可能还没有概念，下面我将以一个例子来讲下Lua的iOS调用基本原理

## 初始化实例应用
首先新建个工程

![](https://github.com/devshen/devshen.github.io/raw/master/img/2015-01-30-Use_Lua_in_iOS/CreateNewProject.png)

再创建一个继承于```UIView```的新类```CubeView```

![](https://github.com/devshen/devshen.github.io/raw/master/img/2015-01-30-Use_Lua_in_iOS/CreateNewFile.png)

![](https://github.com/devshen/devshen.github.io/raw/master/img/2015-01-30-Use_Lua_in_iOS/CreateNewFileWithUIView.png)

在```CubeView.h```中输入以下代码:

```objective-c
#import <UIKit/UIKit.h>

typedef enum : NSUInteger {
	DirectionUp,
	DirectionDown,
	DirectionLeft,
	DirectionRight
} Direction;

@interface CubeView : UIView
{
	float cubeSpeed;
}
@property (nonatomic,strong) NSString *name;

- (id)initWithFrame:(CGRect)frame Speed:(float)speed Name:(NSString *)name;
- (void)goDirection:(Direction)direction;

@end
```

在```CubeView.m```输入以下代码:

```objective-c
#import "CubeView.h"

@implementation CubeView

- (id)initWithFrame:(CGRect)frame Speed:(float)speed Name:(NSString *)name;
{
	self = [super init];
	if (self) {
		[self setFrame:frame];
		cubeSpeed = speed;
		_name = name;
	}
	
	return self;
}

- (void)goDirection:(Direction)direction
{
	CGPoint newPoint = self.center;
	switch (direction) {
		case DirectionUp:
			newPoint.y -= cubeSpeed;
			break;
		case DirectionDown:
			newPoint.y += cubeSpeed;
			break;
		case DirectionLeft:
			newPoint.x -= cubeSpeed;
			break;
		case DirectionRight:
			newPoint.x += cubeSpeed;
			break;
		default:
			break;
	}
	[self setCenter:newPoint];
}

@end
```
代码很简单，就是创建了一个```CubeView```类，其拥有```speed```属性和```name```属性，通过```goDirection:```方法改变View的位置.

然后在```ViewController.m```文件中输入:

```objective-c
#import "ViewController.h"
#import "CubeView.h"

CubeView *cubeTarget;

@interface ViewController ()
@property (nonatomic,strong)NSTimer *timer;
@end

@implementation ViewController

- (void)viewDidLoad {
	[super viewDidLoad];
	
	cubeTarget = [[CubeView alloc]initWithFrame:CGRectMake(120,100,20,20)
											   Speed:10
												Name:@"Target"];
	[cubeTarget setBackgroundColor:[UIColor purpleColor]];
	[self.view addSubview:cubeTarget];
	
	self.timer = [NSTimer scheduledTimerWithTimeInterval:1.0
												  target:self
												selector:@selector(runLoop:)
												userInfo:nil
												 repeats:YES];
}

- (void)runLoop:(id)sender
{
	[cubeTarget goDirection:DirectionDown];
	NSLog(@"%@‘s center is on %@",cubeTarget.name,NSStringFromCGPoint(cubeTarget.center));

}

- (void)didReceiveMemoryWarning {
	[super didReceiveMemoryWarning];
}

@end
```
代码初始化一个```CubeView```实例，给了初始的位置和speed，并开启一个定时器，每隔1秒将```cubeTarget```往界面下方移动，运行代码，能看到如下输出:

```
2015-01-30 23:40:24.568 DVSLua[14694:2319079] Target‘s center is on {130, 120}
2015-01-30 23:40:25.564 DVSLua[14694:2319079] Target‘s center is on {130, 130}
2015-01-30 23:40:26.564 DVSLua[14694:2319079] Target‘s center is on {130, 140}
2015-01-30 23:40:27.563 DVSLua[14694:2319079] Target‘s center is on {130, 150}

```

好，这样前期的准备工作就完成了。

## Lua导入到应用

现在准备将**Lua**嵌入到工程中

1.首先，需要获取Lua源码[LuaResource](http://www.lua.org/ftp/lua-5.1-tar.gz)，本篇博文使用的是Lua5.1版本，5.2,5.3的版本改变了函数注册的方式，不适配本博文的代码。

2.下载后的文件解压后会看到名为```src```的文件夹，里面就是我们需要的，将```src```文件夹改名为```lua```，删除其中的```MakeFile```、```lua.c```、```luac.c```文件，将```lua```文件夹拖到工程中，记住不要选择```create external build system box```

![](https://github.com/devshen/devshen.github.io/raw/master/img/2015-01-30-Use_Lua_in_iOS/ImportLua-1.png)

![](https://github.com/devshen/devshen.github.io/raw/master/img/2015-01-30-Use_Lua_in_iOS/ImportLua-2.png)

这样Lua就嵌入到工程中了。

## Lua的Hello World

Lua提供了C API，通过这些接口，Lua和C被连接起来，对Lua的操作就是靠这些C API，在C环境和Lua环境中间，有一个Lua栈，Lua与C的交互其实就是通过对这个Lua栈的中转来实现的

在```ViewController.h```中导入Lua的头文件

```objective-c
#import <UIKit/UIKit.h>
#include "lua.h"
#include "lualib.h"
#include "lauxlib.h"

@interface ViewController : UIViewController
{
	lua_State *L;
}
@end
```
大家一定看到了```lua_State *L;```这个结构体变量，这个变量是干什么的呢，这是Lua的环境变量，所有的Lua操作，都是基于这个环境变量，类似于iOS绘图中的```context```

添加Lua的初始化方法到```ViewController.m```得```ViewDidLoad```中

```objective-c
	[self initLuaState];
```
实现初始化方法

```objective-c
- (void)initLuaState
{
	L = luaL_newstate();
	luaL_openlibs(L);
	lua_settop(L, 0);
	
	int err;	
	err = luaL_loadstring(L, "print("Hello, world")");	
	if (0 != err) {
		luaL_error(L, "cannot compile lua file: %s",lua_tostring(L, -1));
		return;
	}
	
	err = lua_pcall(L, 0, 0, 0);
	if (0 != err) {
		luaL_error(L, "cannot run lua file: %s",lua_tostring(L, -1));
		return;
	}
}
```
来仔细看下这个初始化方法

```L = luaL_newstate()``` 这是初始化了一个Lua的环境变量，可以看到之后的Lua操作都需要这个环境变量，这里你看到方法的前缀是```luaL_```，标准库的API是以```lua_```为前缀的，而```luaL_```是标准库的扩展库，```luaL_```前缀的函数一定能用```lua_```前缀的函数来实现；

```	luaL_openlibs(L);``` 这是加载所有的Lua库类

```lua_settop(L, 0);``` 这是将栈的栈顶索引设置为指定的数值(此处为0)，这个怎么理解，看下图

![](https://github.com/devshen/devshen.github.io/raw/master/img/2015-01-30-Use_Lua_in_iOS/LuaStack.png)

Lua栈的栈顶索引为-1，依次往下；而栈底为1，依次往上，比如说，一个栈原来有6个元素，调用```lua_settop(L, index)```设置index为5，就是把栈从下往上第5个(也就是“abc”字符串)作为栈顶，那么也就是删掉"111"这个栈顶元素，这是相对于栈底元素设置的；如果是相对于栈顶元素，要实现同样的小刚，就要设置索引为-2，也相当于删除掉栈顶元素。

回到刚才，```lua_settop(L, 0);```就是将栈底( index = 1 )下面的索引( index = 0 )作为栈顶，那就相当于删除栈内所有内容，所以这个函数的作用就是**清空栈内容**

```luaL_loadstring(L, "print("Hello, world")")``` 很明显，就是立刻执行```"print("Hello, world")"```Lua命令

再次运行程序，结果会打印出```Hello, world```

好，这是大家喜闻乐见的一串字符

## 使用Lua脚本文件
继续，在```ViewController.m```写以下代码

```objective-c
int cubeTarget_position(lua_State *L){
	lua_pushnumber(L, cubeTarget.center.x);
	lua_pushnumber(L, cubeTarget.center.y);
	return 2;
}
```
再看来下，```lua_pushnumber(L,a)```表示将变量**a**压入Lua栈，方法还返回了压入的数据数量

这个方法的目的很明显，现在我们有了能够通过stack将对象中心点位置传给Lua栈的函数，不过想要Lua脚本能够调用它，还需要将这个函数与Lua环境绑定起来，我们可以通过```luaL_register```函数，将一组方法与Lua绑定，那么，在```ViewController.m```中添加下面方法

```objective-c
const struct luaL_Reg cubeLib[] = {
	{"cubeP", cubeTarget_position},
	{NULL, NULL}
};

int luaopen_cubeLib (lua_State *L){
	luaL_register(L, "myLib", cubeLib);
	return 1;
}
```
其中```luaL_Reg```类型的数组，包含一组函数，这个数组通过```luaL_newlib```将函数传递给了Lua，这样，Lua脚本就能识别这些"注册"了的函数，```luaopen_cubeLib```方法需要在run脚本前调用，我们可以在```initLuaState```方法中的```lua_settop(L,0)```下写上

```objective-c
	luaopen_cubeLib(L);
```

下面我们来创建一个lua脚本
![](https://github.com/devshen/devshen.github.io/raw/master/img/2015-01-30-Use_Lua_in_iOS/CreateLua-1.png)
![](https://github.com/devshen/devshen.github.io/raw/master/img/2015-01-30-Use_Lua_in_iOS/CreateLua-2.png)

编辑Lua脚本如下

```c
function print_cube_position()

cube_x, cube_y = cubeLib.cubeP()

print(string.format("cube is at (%f, %f)", cube_x, cube_y))

end
```

现在需要用以下代码替换之前的```luaL_loadstring```内容(之前用来执行helloworld命令了)

```objective-c
NSString *luaFilePath = [[NSBundle mainBundle] pathForResource:@"Script" ofType:@"lua"];
err = luaL_loadfile(L, [luaFilePath cStringUsingEncoding:[NSString defaultCStringEncoding]]);
	
```
并且还要修改我们的run loop 方法

```objective-c
- (void)runLoop:(id)sender
{
	[cubeTarget goDirection:DirectionDown];
	
	lua_getglobal(L, "print_cube_position");
	
	int err = lua_pcall(L, 0, 0, 0);
	if (0 != err) {
		luaL_error(L, "run error: %s",
				   lua_tostring(L, -1));
		return;
	}
	
}
```
这里得```lua_getglobal(L, "print_cube_position")```指的是将Lua脚本中的变量```print_cube_position```方法放到栈顶。

```lua_pcall(L, 0, 0, 0)```则调用栈顶的函数。

运行，打印

```objective-c
cube is at (130.000000, 120.000000)
cube is at (130.000000, 130.000000)
cube is at (130.000000, 140.000000)
cube is at (130.000000, 150.000000)
cube is at (130.000000, 160.000000)
```
## 更进一步
接下来，我们更进一步，添加如下方法到```ViewController.m```中,

```objective-c
int go_right(lua_State *L){
	CubeView *sc = (__bridge CubeView *)(lua_touserdata(L, 1));
	[sc goDirection:DirectionRight];
	return 0;
}

int go_left(lua_State *L){
	CubeView *sc = (__bridge CubeView *)(lua_touserdata(L, 1));
	[sc goDirection:DirectionLeft];
	return 0;
}

int go_up(lua_State *L){
	CubeView *sc = (__bridge CubeView *)(lua_touserdata(L, 1));
	[sc goDirection:DirectionUp];
	return 0;
}

int go_down(lua_State *L){
	CubeView *sc = (__bridge CubeView *)(lua_touserdata(L, 1));
	[sc goDirection:DirectionDown];
	return 0;
}

int get_cube_position(lua_State *L){
	CubeView *sc = (__bridge CubeView *)lua_touserdata(L, 1);
	lua_pushnumber(L, sc.center.x);
	lua_pushnumber(L, sc.center.y);
	return 2;
}
```
这里我们用了新的lua方法```lua_touserdata```，它能够向Lua传递指针，这些指针是我们在OC中创建的。

下一步，将新写的方法加入到注册数组中去，并且都取好名字(取名字不容易)

```objective-c
const struct luaL_Reg cubeLib[] = {
	{"go_right", go_right},
	{"go_left", go_left},
	{"go_up", go_up},
	{"go_down", go_down},
	{"get_cube_Position",get_cube_position},
	{"cubeP", cubeTarget_position},
	{NULL, NULL}
};
```
在cubeTaregt下面新建一个```CubeView```对象

```objective-c
CubeView *otherCube;
```
在```ViewDidLoad```里初始化这个新的对象,给它加上拖拽手势

```objective-c
otherCube = [[CubeView alloc]initWithFrame:CGRectMake(100, 200, 20, 20) Speed:10 Name:@"Other"];
[otherCube setBackgroundColor:[UIColor greenColor]];
UIPanGestureRecognizer *panGesture = [[UIPanGestureRecognizer alloc]initWithTarget:self action:@selector(panAction:)];
[cubeTarget addGestureRecognizer:panGesture];
	
[self.view addSubview:otherCube];
```

```objective-c
- (void)panAction:(UIPanGestureRecognizer *)recognizer
{
	if (recognizer.state != UIGestureRecognizerStateEnded && recognizer.state != UIGestureRecognizerStateFailed)
	{
		CGPoint location = [recognizer locationInView:recognizer.view.superview];
		recognizer.view.center = location;
	}
}
```
新建一个Lua脚本```Chase.lua```

```c
function chase(other)

	cubeTarget_x, cubeTarget_y = myLib.cubeP()

	x, y = myLib.get_cube_Position(other)

	if cubeTarget_x < x then
		myLib.go_left(other)
	elseif cubeTarget_x > x then
		myLib.go_right(other)
	end

	if cubeTarget_y > y then
		myLib.go_down(other)
	elseif cubeTarget_y < y then
		myLib.go_up(other)
	end

end
```
修改```(void)initLuaState```方法

```objective-c
	NSString *luaFilePath = [[NSBundle mainBundle] pathForResource:@"Chase" ofType:@"lua"];
```
修改```(void)runLoop:(id)sender```方法

```objective-c
	lua_getglobal(L, "chase");
	lua_pushlightuserdata(L, (__bridge void *)(otherCube));
	int err = lua_pcall(L, 1, 0, 0);
	if (0 != err) {
		luaL_error(L, "run error: %s",
				   lua_tostring(L, -1));
		return;
	}
```
这里就使用了```lua_pushlightuserdata```来传数据.

运行，就会看到绿色方块会朝着紫色方块移动，当你拖拽紫色方块后，绿色方块也会改变方向，而这一系列行为逻辑并没有写在工程代码中，而是写在了Lua脚本中，而Lua脚本是可以通过网络下载进应用中，也就能够改变绿色方块的运行逻辑了。

# 总结
总结下iOS调用Lua的过程：

- 创建Lua环境```luaL_newstate```
- 加载基本库```luaL_openlibs```
- 将需要被Lua脚本使用的函数注册到Lua环境中```lua_register```
- 加载Lua脚本```luaL_loadfile```
- OC代码通过```lua_getglobal```获取到Lua脚本中的函数
- 通过```lua_pushlightuserdata```传递数据
- 再通过```lua_pcall```调用获取到的函数
- 在运行Lua脚本的函数时，脚本调用了OC注册到Lua环境中的函数
- 调用OC方法，改变界面或者业务逻辑


完整工程可以在这边现在[GitHub](https://github.com/devshen/DVSLua)下载，如果觉得有用，请Star，谢谢