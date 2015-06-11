#virtual function 带来的 class 产生的膨胀作用
对于如下代码：
	
	class Point
	{
	public:
		Point(float x = 0.0, float y = 0.0) :
		 _x(x), _y(y) { }

		 // no copy constructor, copy operator, destructor defined...
		 virtual float z();

	protected:
		float _x, _y;
	};
	
编译器对 class 的膨胀操作如下：

*	我们所定义的 constructor 被附加了一些代码，以便将 vptr 初始化，这些代码必须放在任何 base class constructor 之后，但必须在任何 user code 之前：

		Point* Point::Point(Point* this, float x = 0.0, float y = 0.0) :
		 _x(x), _y(y) {
		 	// 设定vptr
		 	this->__vptr_Point = __vbtl_Point;

		 	// 扩展 member initialization list
		 	this->_x = x;
		 	this->_y = y;

		 	// 传回 this 对象
		 	return this;
		 }
		 
*	合成一个 copy constructor 和一个 copy assignment operator，而且其操作不再是 trival（但 implicit destructor 仍然是 trival）。

		Point* Point::Point(Point* this, const Point &rhs) {
		 	// 设定vptr
		 	this->__vptr_Point = __vbtl_Point;

		 	// 将 rhs 坐标中的连续位拷贝到 this 对象，或经由 member assignment 提供一个 member...
		 	// ...

		 	// 传回 this 对象
		 	return this;
		 }

编译器在优化状态下可能会把 object 的连续内容拷贝到另一个 object 上，而不会实现一个精确的“以成员为基础”的赋值操作。c++ standard 要求编译器尽量延迟 nontrivial members 的实际合成操作，直到真正遇上其使用场合为止。 
