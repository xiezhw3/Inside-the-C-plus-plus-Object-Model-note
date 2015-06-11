#Exception Handling
当一个 exception 发生时，编译系统必须完成以下事情：

-	检验发生 throw 操作的函数
-	决定 throw 操作是否发生在 try 区段中
-	若是，编译系统必须把 exception type 拿来和每一个 catch 子句进行比较
-	如果比较后吻合，流程控制应该交到 catch 子句手中
-	如果 throw 的发生并不在 try 区段中，或没有一个 catch 子句吻合，那么系统必须 <b>(1)</b> 摧毁所有 activity local objects，<b>(2)</b> 从堆栈中将目前的函数 "unwind" 掉，<b>(3)</b> 进行到程序堆栈的下一个函数中去，然后重复上述步骤。