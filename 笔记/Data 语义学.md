Data 语义学
-
在如下代码中，在 64 位机子上的结果是：

	#include <iostream>
	
	using namespace std;
	
	class X {};
	class Y : virtual public X {};
	class Z : virtual public X {};
	class M : public Y, public Z {};
	
	int main(int argc, char const *argv[])
	{
		cout << sizeof (X) << ' ' << sizeof (Y) << ' ' << sizeof (Z) << ' ' << sizeof (M) << endl;
	
		return 0;
	}

>1 8 8 16

首先来说 X 的结果，大小为 1 的原因是，编译器在 class 里面安插进一个char，这使得这一 class 的两个 objects 得以在内存中配置独一无二的地址。（另外一个可能原因是，对于一个 struct，模数最小是 1，不可能是0）

Y，Z大小受到三个因素的影响;

1.	语言本身所造成的额外负担，比如支持 virtual
2.	编译器对于特殊情况所提供的优化处理	`Virtual base class X subobject 的1 bytes 大小也出现在 class Y 和 Z 身上。传统上它被放在 devided class 的固定部分的尾端`
3.	Alignment 的限制

Empty virtual base class 提供一个 virtual interface，没有定义任何数据。某些新近的编译器(比如我现在用的 GCC4.8.1 MAC)提供特殊的处理。在这个策略下，一个 empty virtual base class 被视为 devided class object 最开头部分，也就是说它没有花费任何额外的空间。也就节省了上面的 2 中的 1 byte

`一个 virtual base class 只会在 devided class 中存在一份实例，不管它在 class 继承体系中共出现多少次`

在这里 class M 的大小是 16，它的大小由以下几点决定:

-	被大家共享的唯一一个 class X 实例，大小为 1 byte
-	Base class Y 的大小，减去“因 virtual base class X 而配置”的大小，结果是 8 bytes，class Y 一样
-	classM 自己的大小：0 byte
-	class M 的 alignment 数量

这里加起来应该是 17，对齐后应该是 24, 但实际结果是 16，原因是前面说的编译器的原因，没有在 class M 的 object 中加上 X 的 1 byte。

#Data member 的绑定
在 c++ standard 中说明，如果一个 inline 函数在 class 声明之后立刻被定义的话，那么还是对其评估求值（evaluate），也就是说，对于 member functions 本体的分析，会直到整个 class 的声明出现了才开始。因此在一个 inline member function 躯体之内的一个 data member 绑定操作，会在整个 class 声明之后才完成：

	extern int x;

	class A {
	public:
		float x() {return x; }
	private:
		float x;
	};
	
这里返回的 x 是成员变量。

但是，这对于 member function 的argumentlist 并不为真。Argument list 中的名称还是会在它们第一次遭遇时被适当地决议完成。如：

	#include <iostream>
	#include <cstdio>
	
	using namespace std;
	
	typedef int length;
	
	class A {
	public:
		A (length num_) {
			num = num_;
			cout << num << endl;
		}
	private:
		typedef double length;
		length num;
	};
	
	int main(int argc, char const *argv[])
	{
		A a(12.2);
	
		return 0;
	}

在这里构造函数中的 length 用的是 global 的那个 length. 但是定义 num 的时候用的是 double 的length。
>为了解决这个问题，需要使用某种防御性程序风格，就是 “总把`nested type声明`放在 class 的起始处”

##Data member 的布局
c++ standard 要求，在同一个 access section 中，member 的排列只需要符合“较晚出现的 members 排列在 clas object 中较高的地址”即可，并不一定要连续排列。介于members之间的东西可能是边界填补（alignment）。

c++ standard 也允许编译器多个 access sections 之中的 data member 自由排列，不必在乎它们出现在 class 中的顺序，`但是目前没有编译器这样做`，比如：
	
	class Point3d{
	private:
		float x;
	private:
		float y;
	private:
		float z;
	};
那么 x, y, z 的顺序可以不定。视编译器而定。

##Data member 的存取
成员变量的存取代价，需要看成员变量的性质，以及类本身的类型。

####Static Data Members
被编译器提出与 class 之外，并被视为一个 global 变量。每一个 member 的存取许可以及与 class 的关联，并不会招致任何空间上或执行时间上的额外负担——不论是在个别的 class object 还是在 static data member 本身。

每一个 static 打他、 member 只有一个实例，存放在程序的 data segment 中。每次程序存取 static member 时，就会被内部转化为对该唯一 extern 实例的直接存取操作。

	origin.chunkSize = 250;
	-->
	Point3d::chunkSize = 250;

从指令执行的观点来看，这是 c++ 语言中“通过一个指针和通过一个对象来存取 member，结论完全相同”的唯一情况。这his因为 static member 其实并不在 object 之中，存取 static member 不需要通过 class object。'.' 只是一种便宜行事。

####Nonstatic Data Members
Nonstatic data members 直接存放在每一个 class object 中。除非经由显式或隐式的 class object，否则无法直接存取它们。

欲对一个 nonstatic data member 进行存取操作，编译器需要把 class object 的起始地址加上 data member 的偏移位置。比如：

	origin.y = 0.0;
	-->
	&origin + (&Point3d::y - 1);
	
