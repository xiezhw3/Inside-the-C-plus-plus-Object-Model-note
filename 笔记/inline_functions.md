#Inline Functions
一般而言，处理一个 inline 函数，有两个阶段：

1.	分析函数定义，以决定函数的“intrinsic inline ability”（本质的 inline 能力），这里意指与编译器相关
	
		如果函数因其复杂度，或因其构建问题，被判断为不可成为 inline，它会被转为一个 static 函数，并在“被编译模块”内产生对应的函数定义
		
2.	真正的 inline 函数扩展操作是在调用的那个点上，这会带来参数的求值操作以及临时性对象的管理。

###形式参数
在inline扩展期间，每一个形式参数都会被对应的实际参数取代，可以通过下面的例子来明白其操作：

	inline int min(int i, int j) {
		return i < j ? i : j;
	}

	inline int bar() {
		int minval;
		int val1 = 1024;
		int val2 = 2048;
		/* 1 */ minval = min(val1, val2);
		/* 2 */ minval = min(1024, 2048);
		/* 3 */ minval = min(foo(), bar() + 1);
	}

那么 1 会被替换为：

	minval = val1 < val2 ? val1 : val2;

2 会被替换为：

	minval = 1024;

3 则会引发参数的副作用。它需要导入一个临时性对象，以避免重复求值：

	int t1;
	int t2;
	minval = ( (t1 = foo() ), (t2 = bar() + 1) ), t1 < t2 ? t1 : t2;
	
###局部变量
一般而言，inline 函数中的每一个局部变量都必须被放在函数调用的一个封闭区段中，拥有一个独一无二的名称。如果 inline 函数以单一表达式扩展多次，则每次扩展都需要自己的一组局部变量。如果 inline 函数以分离的多个式子被扩展多次，那么只需一组局部变量，就可以重复使用。

inline 函数中的局部变量，再加上有副作用的参数，可能会导致大量临时性对象的产生。特别是如果它以单一表达式被扩展多次的话。