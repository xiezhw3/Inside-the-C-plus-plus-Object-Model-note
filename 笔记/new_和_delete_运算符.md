#new 和 delete 运算符
当一个 base class 指针指向一个 derived class objects所组成的数组(且 derived class objects 比其 base 大)。当两者的析构函数都是虚拟函数的时候，不能够直接 
	
	delete []ptr;
	
这样的方式来完成空间的释放。前面我们知道，对象数组细狗的时候调用的是 vec_delete，那么这里传递给 vec_delete 的析构函数就会是基类的析构函数，且对象大小也会是 base class object 的大小，这样释放就会出错了。解决的方法是在程序员层面：
	
	for (int ix = 0; ix < elem_count; ++ix) {
		Point3d *p = &((Point3d*)ptr)[ix];
		delete p;
	}
基本上，程序员必须迭代走过整个数组，将 delete 运算符施行与每一个数组元素上。

##Placement Operator new 的语意
它的调用方式为：

	Point2w *ptw = new(arena) Point2w;
	
arena 类型为 _void*_ ，指向内存的一个区块用以放置新产生出来的 Point2w object。它是预先定义好的，但它的实现中仅仅是传回“获得的指针”。

	void*
	operator new(size_t, void* p) {
		return p;
	}
	
但是，事实上这只是所发生的操作的一半而已，另一半无法由程序员产生出来。

placement _new_ operator 所扩充的另一半是将 Point2w constructor 自动施行于 arena 所指的地址上：

	Point2w *ptw = (Point2w*)arena;
	if (ptw != 0)
		ptw->Point2w::Point2w();
		
这一份代码决定了 objects 被放置在哪里。编译系统保证 object 的 constructor 会施行于其上。

在使用 placement new 的时候要注意不能直接 delete 了 ptw，一般是调用其析构函数，因此 arena 所指空间还能继续使用。【新版本的c++中，直接的 delete 已经可以完成析构和释放的工作，而不用特意调用 placement delete】
>
另外，如果要重复使用一块空间 arena，编译器并不会自动进行对象的析构，所以程序员一般需要保证进行对象的析构。

一般而言，placement new operator 并不支持多态。被交给 new 的指针，应该适当地指向一块预先分配好的内存。如果 derived class 比起 base class 大，例如：

	Point2w *p2w = new (area) Point3w;
Point3w 的 constructor 将会导致严重的破坏。

还有一个问题是如下代码：

	struct Base { int j; virtual void f(); };
	struct Dreived : Base { void f(); };
	void fooBar() {
		Base b;
		b.f();  // Base::f() 被调用
		b.~Base();
		new (&b) Dreived;
		b.f();	// 哪一个 f() 被调用？
	}
>
在这里，Base 和 Dreived 的大小是一样的，所以在内存中是安全的，但是，c++标准中，上述行为是`未定义行为`，但是大部分编译器的结果是 Base::f()，而不是我们所想的 Drevied::f()
