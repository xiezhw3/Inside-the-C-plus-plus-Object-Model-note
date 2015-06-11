#为什么 static member function 不能有 const 修饰
在成员函数中，const 是用来修饰 this 指针的，也就是说，对一个成员函数使用 const 修饰符并不代表里面的所有变量都不能修改，而仅仅是 this 指针所包含的成员变量不能修改，其他的比如全局变量，局部变量都是可以修改的。比如：

	int global = 10;

	class Point2d
	{
	public:
		Point2d(float x = 0.0, float y = 0.0) {
			_x = x, _y = y;
		}

		const void fund() const {
			int local = 0;
			local += 10;
			global = 2;
			_x = 100;
		}

	private:
		float _x, _y;
	};
	
此时的报错信息为：

	a.cpp:31:6: error: read-only variable is not assignable
                _x = 100;
                ~~ ^
	1 error generated.
	
可以看出，此时的出错只有成员变量。

既然如此，也就是说 const 在成员函数中只修饰 this，而在 static member function 中并没有 this 指针，所以不需要 const 修饰。

另一个可以看出的是，在一般函数（非成员函数）里面，也是不能用 const 修饰的。

<hr>
static member function 的主要特性就是它没有 this 指针。也就有如下的特性：

-	它不能够直接存取其class中的nonstatic members
-	它不能够被声明为 const、volatile 和 virtual
-	它不需要经由 class object 才被调用

<hr>

如果取一个 static member function 的地址，获得的将是其在内存中的位置，也就是其地址。由于 static member function 没有 this 指针，所以其地址的类型并不是一个“指向 class member function的指针”，而是一个“nonmember 函数指针”