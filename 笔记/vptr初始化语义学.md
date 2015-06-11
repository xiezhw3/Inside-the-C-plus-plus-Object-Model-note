#vptr初始化语义学
我们假设有如下调用关系：

	class Point3d : virtual public Point { ... };
	class Vertex : virtual public Point { ... };
	class Vertex3d : public Point3d, public Vertex { ... };
	class PVertex : public Vertex3d { ... };
	
假设这个继承体系中每一个 class 都定义了一个 virtual function _size()_ ，此函数负责传回 class 的大小。
那么，如果我们有如下的调用：

	Point3d::Point3d(float x, float y, float z)
		: _x(x), _y(y), _z(z) {
		if (spyOn)
			cerr << "Withi Point3d::Point3d()"
				 << "size: " << size() << endl;
	}
	
那么在定义 PVertex object 时，前述的 5 个 constructor 应该调用哪个 class 的 size 呢？

c++语言规则告诉我们，在 Point3d constructor 中调用 _size()_ 函数，必须被决议为 Point3d::_size()_ 而不是  PVertex::_size()_。更一般地说，在一个 class(本例问 Point3d)的 constructor(和 destructor)中，经由构造函数的对象(本例为 PVertex)来调用一个 virtual function，其函数实例应该是在此 class(本例为Point3d)中有作用的那个。
我们知道，constructor 的调用顺序是：有根源而末端(bottom up)、由内而外(inside out)。在 PVertex constructor 执行完毕之前，PVertex 并不是一个完整对象；Point3d constructor 执行之后，只有 Point3d subobject 构造完毕。

那么如何实现？
因为调用操作限制必须在 constructor(或 destructor)中直接调用，所以很显然可以用静态方式决议，这里不可以用虚拟机制。但是如果 size 里面又调用虚拟函数的话呢？这里面也应该是静态静态决议的。

######另一种好的解决办法：
	
我们知道，虚拟函数的决议用的是 vptr，那么只要控制 vptr 的初始化和设定操作就能达到目的了。我们对 vptr 的初始化操作为：
	
	在 base class constructor 调用操作之后，但是在程序员供应的代码或是"member initialization list 中所列的 members 初始化操作"之前
	
令每一个 base class constructor设定其对象的 vptr，使它指向相关的 vtbl 后，构造中的对象就可以严格而正确地变成“构造过程中所幻化出来的每一个 class”的对象。也就是说，一个 PVertex 对象会先形成一个 Point 对象、一个 Point3d对象、一个 Vertex 对象、一个 Vertex3d 对象，然后才成为一个 PVertex 对象。constructor 的执行算法如下：

1.	在 derived class constructor 中，“所有 virtual base classes”及“上一层 base class”的constructor 会被调用
2.	上述完成后，对象的vptr(s)被初始化，指向相关的 vtbl(s)
3.	如果有 member initialization list 的话，将在 constructor 体内扩展开来。这必须在 vptr 被设定之后才做，以免有一个 virtual member function 被调用。
4.	最后，执行程序员所提供的代码

由上可以得出下面的结论：在 class 的constructor 的 member initialization list 中调用该 class 的一个虚拟函数，因为 vptr 保证能够在 member initialization list 被扩展之前设定好，所有是安全的。但是可能出现该虚拟函数依赖未设定好的其他 members 的情况，在这种情况下，是不安全的。