#Nonstatic member function
c++的设计准则之一就是：nonstatic member function 至少必须和一般的 nonmember function有相同的效率。如下代码：

	float magnitude3d( const Point3d *_this ) { ... }
	float Point::magnitude3d() const { ... }
	
上面的调用中，选择 member function 不应该带来什么额外的负担。

在编译器内部，已经将“member函数实例”转换为对等的“nonmember函数实例”。对于成员函数调用，其转化步骤如下：

1. 改写函数的 signature（函数原型）以安插一个额外的参数到 member function 中，用以提供一个存取管道，使 class object 得以将此函数调用。该额外参数就是 this 指针：

		non-const nonstatic member 的扩张过程
		Point3d Point3d::magnitude(Point3d *const this);
		
		如果 member function 是 const，则变成：
		
		Point3d Point3d::magnitude(const Point3d* const this);
		
2. 将每一个“对 nonstatic data member 的存取操作”改为经由 thsi 指针来存取

		this->_x;
		
3. 将 member function 重新写成一个外部函数。将函数名经过“mangling”处理，使它在程序中成为独一无二的词汇：

		extern magnitude__7Point3dFv( register Point3d* const this);
		
那么
	
		obj.magnitude();
		ptr->magnitude();
		
就变成了：

		magnitude__7Point3dFv(&obj);	
		magnitude__7Point3dFv(ptr);	