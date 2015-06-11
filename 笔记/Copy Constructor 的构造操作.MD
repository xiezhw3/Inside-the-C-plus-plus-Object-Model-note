#Copy Constructor 的构造操作
有三种情况，会以一个 object 的内容作为另一个 class object 的初值。最明显的就是对一个 object 做一个显式的初始化操作，如：
	
	MyClass m;
	MyClass mT = m;
	
另两种情况是当 object 作为参数交给某个函数时：

	void fun(MyClass m);
	
以及当函数传回一个 class object 时：
	
	MyClass fun() {
		MyClass m;
		...
		return m;
	}
	

### Default Memberwise Initialization
当 class object 以 “相同 class 的另一个 object” 作为初值，其内部是以所谓的 default memberwise initialization 手法完成的，也就是把每一个内建的或派生的 data member（例如一个指针或数组）的值，从某个 object 拷贝一份到另一个 object 身上。不过它并不会拷贝其中的 member class object，`而是以递归的方式施行 memberwise initialization`。

`一个 class 可以通过拷贝初始化和赋值两种方式进行复制得到`

c++ standard 上说，如果 class 没有声明一个 copy constructor，就会有隐式的声明或隐式的定义出现，分为 trivial 和 nontrivial 两种。只有 nontrivial 的才会被合成于程序中。决定一个 copy constructor 是否为 trivial 的标准在于 class 是否展现出所谓的 “bitwise copy smatics”。

###bitwise copy semantics（位逐次拷贝）

	class Word {
	public:
		Word(const char*);
		~Word() { delete [] str}
		// ...
	private:
		int cnt;
		char *str;
	};

这种情况下并不需要合成一个 default copy constructor，因为上述声明展示了 “default copy sematics”。

##不要 Bitwise Copy Semantics
在如下四种情况下，class 不展现出 `bitwise copy smatics`

1.	当 class 内含一个 member object 而后者的 class 声明有一个 copy constructor时（无论是被 class 设计者显式声明，还是编译器合成）
2.	当 class 继承自一个 base class 而后者存在一个 copy constructor 时
3.	当 class 声明了一个或多个 virtual functions 时
4.	当 class 派生自一个继承串链，其中有一个或多个 virtual base functions 时

前两种情况下，编译器必须将 member 或 base calss 的 "copy constructor调用操作"安插到被合成的 copy constructor中。

> 下面是对 `不要 Bitwise Copy Semantics` 的后面两种情况的进一步说明

####重新设定 Virtual Table 指针
很显然，如果编译器对于每一个新产生的 class object 的vptr 不能成功而正确地设定好其初值，将导致可怕的后果。因此，当编译器导入一个 vptr 到 class 之中时，该 class 就不再展现 bitwise semantics 了。

	class ZoAnimal {
	public:
		ZoAnimal();
		virtual ~ZoAnimal();
	
		virtual void animate();
		virtual void draw();
		// ...
	private:
		// ZoAnimal 的 animate() 和 draw 所需要的数据
	};
	class Bear : public ZoAnimal {
	public:
		Bear();
		void animate(); // 虽未写明，但这个函数是 virtual 的
		void draw(); // 同上
		virtual void dance();
		// ...
	private:
		// ...
	};

在上面的代码中，ZoAnimal class object 以另一个 ZoAnimal class object 作为初始值，或者一个 Bear class object 以另一个 Bear class object 作为初始值，都可以直接靠 bitwise copy semantics 完成。`除了可能会有的 pointer member 之外`

比如：

	Bear yogi;
	Bear winnie = yogi;
	
在这里，因为一个 class 只有一张 vtbl，那么两个 class object 的 vptr 指向同一个 vtbl，所以此时可以用值拷贝。

但是当一个 base class object 以其 devided class 的 object 内容做初始化操作时，其 vptr 复制操作也必须保证安全。
例如：

	void draw(const ZoAnimal& zoey) { zoey.draw(); }
	void foo() {
		// franny 的 vptr 指向 ZoAnimal 的 virtual table,
		// 而非 Bear 的 virtual table (它由 yogi 的 vptr 指出)
		ZoAnimal franny = yogi;
		draw(yogi);  // 调用 Bera::draw()
		draw(franny); // 调用 ZoAnimal::draw()
	}

在这里，draw(franny); 调用的是 ZoAnimal::draw()。

####处理 Virtual Base Class Subobject
一个 class object 如果以另一个 object 作为初值，而后者有一个 virtual base class subobject，那么也会使 “bitwise copy semantics”失效。
每一个编译器对于虚拟继承的支持承诺，都代表让“devided class object中的virtual base class subobject位置”在执行期就准备妥当。维护位置的完整性是编译器的责任.














