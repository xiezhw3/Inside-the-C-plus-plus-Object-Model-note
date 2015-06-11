#支持“指向 Virtual Member Functions”的指针
考虑下面的程序片段：

	float (Point::*pmf)() = &Point::z();
	Point *ptr = new Point3d;
	
_pmf_ ，一个指向 member function 的指针，被设值为 _Point::z()_ （一个 virtual function）的地址。 _ptr_ 则被指定以一个 Point3d 对象。如果我们直接经由 _ptr_ 调用 _z()_:
	
	ptr->z();
	
被调用的是 Point3d::z();

那么如果通过 _pmf_ 间接调用呢？

	(ptr->*pmf)();
	
调用的仍然是 _Point3d::z()_

我们知道，对一个 nonstatic member function 取其地址，将获得该函数在内存中的地址。然而面对一个 virtual function，其地址在编译时期是未知的，所能知道的仅是 virtual function 在其相关的 virtual table 中的索引值。也就是说，对一个 virtual function 取其地址，所能获得的只是一个索引值。在编译器内部一般会转化为这样：

	(*ptr->vptr[(int)pmf])(ptr);
	
那么，pmf 就必须是一个能代表 nonvirtual 和 virtual 两种 member function 的指针。而这两种中一个代表内存地址，另一个代表 vtbl 中的索引值。因此，编译器必须定义 pmf，使它能够 <b>(1)</b>持有两种数值。<b>(2)</b>更重要的是其数值可以被区别代表内存地址还是 virtual table 中的索引值。
可以使用如下结构：

	(((int)pmf) & ~127)
	? 	// non-virtual invocation
	( *pmf )(ptr)
	: 	// virtual invoaction
	( * ptr->vptr[ (int)pmf ]( ptr ));
	
上面的方法的劣势是继承体系中最多只有 128 个 virtual function。

###在多重继承下，指向 member function 的指针

	struct _mptr {
		int delta;
		int index;
		union {
			ptrtofunc faddr;
			int v_offset;
		};
	};
	
index 持有 virtual table 索引，faddr 持有 nonvirtual member function 地址。当 index 不指向 vtbl时，会被设置为 -1. 那么如下的操作：

	(ptr->*pmf)();
	
就会转换为：

	( pmf.index < 0 )
	? 	// non-virtual invocation
	( *pmf.faddr )( ptr )
	:	// virtual invocation
	( *ptr->vptr[pmf.index] (ptr))
	
在上面的结构体中，delta字段表示 this 指针的 offset 值，而 v_offset 字段放的是一个 virtual（或多重继承中的第二或后继的）base class的 vptr 位置