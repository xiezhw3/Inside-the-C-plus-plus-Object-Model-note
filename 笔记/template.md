#Template
member functions（至少对于那些未被使用过的）不应该被“实例化”。只有在member functions被使用的时候，C++ standard 才要求它们被实例化。目前的编译器并不精确遵循这个要求。之所以由使用者来主导实例化，主要有两个原因：

-	空间和时间效率的考虑，如果 class 里面有 100 个 member functions，但你的程序只针对某个类型使用了其中两个，针对另一个类型使用其中5个，那么其他193个函数都“实例化”将会花费大量的时间和空间
-	尚未实现的机能。并不是一个 template 实例化的所有类型就一定能够完整支持一组 member functions 所需要的所有运算符。如果只“实例化”那些真正用到的member functions，template 就能够支持那些原本可能会造成编译时期错误的类型

比如：
	
	Point< float > *p = new Point< Point >;
	
在上面代码中，只有_(1)_ Point template 的 float 实例，new运算符，default constructor 需要被实例化。虽然 new 是这个 class 的一个 implicitly static member，以至于它不能够直接处理任何一个 nonstatic members。但它还是依赖于真正的 template 类型，因为它的第一个参数 size_t 代表 class 的大小。

函数在什么时候实例化，目前流行两种策略：

-	在编译的时候。那么函数将“实例化”于 p 存在的那个文件中。
-	在链接的时候。那么编译器会被一些辅助工具重新激活。template 函数实例可能放在这一文件中、别的文件中或一个分离的位置。

###Template 中的名称决议法
首先必须区分在 c++ standard 中的两种意义：<b>(1)</b>是"Scope of the template defination"，也就是“定义出template”的程序端。 <b>(2)</b>是"scope of the template instantiation"，也就是“实例化 template”的程序端。

	// scope of the template definition
	extern double foo( double );

	template < class type >
	class ScoopeRules
	{
	public:
		void invariant() {
			_member = foo(_val);
		}

		type type_dependent() {
			return foo(_member);
		}
		//...
	private:
		int _val;
		type _member;
	};

第二种情况举例如下：

	// scope of the template instantiation
	extern int foo(int);
	//...
	ScoopeRules< int > sr0;
	
如果我们有一个函数调用操作：
	
	// scope of the template instantiation
	sr0.invariant();
	
那么调用的究竟是哪一个 _foo()_ 的函数实例呢？

	// 调用的是哪一个 foo() 函数实例？
	_member = foo(_val);
