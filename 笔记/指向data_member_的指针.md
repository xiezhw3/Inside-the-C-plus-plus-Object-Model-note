#指向Data Member 的指针
首先，对于 vptr，编译器一般会把它放在对象的头或尾。对于如下代码：

	class Point3d {
	public:
		float _x, _y, _z;
	};

	int main(int argc, char const *argv[])
	{
		printf("%p\n", &Point3d::_x);
		printf("%p\n", &Point3d::_y);
		printf("%p\n", &Point3d::_z);
		cout << sizeof(float) << endl;
		return 0;
	}
	
输出结果为：

	0x8
	0xc
	0x10
	4
	
从上面结果可以看出，取某个坐标成员的地址，得到的是成员在 class object 中的偏移位置。

一般来说，如果去取data member 的地址，传回的值总会多1，也就是 1 5 9 ...， 这是为了区分“没有指向任何 data member”的指针，和一个指向“第一个 data member” 的指针。比如如下代码：

	float Point3d::*p1 = 0; // 【在测试的时候不提供初始化才会把p1的值设为0x0，直接=0时是0xfffffff】
	float Point3d::*p2 = &Point3d::x; // 此时没有vptr
	// 这里怎么区分？
	if (p1 == p2) {
		cout << " p1 & p2 contain the same value -- ";
		cout << " they must address the same member!" << endl;
	}
为了区分 p1 和 p2，每一个真正的 member offset 值都被加上 1.因此，不论编译器或者使用者都必须注意，在真正使用该值以指出一个 member 之前，请先减掉 1.
	
