# C/C++ 碎碎念

1. assert 与 exception 的区别: 
   
   assert 的原型是
    
        void assert(int expression)

    如果 expression 为真, 那么不会触发任何东西, 程序继续运行; 如果为假, 那么它先向 stderr 打印一条出错信息, 再通过调用 abort 来终止程序的运行. 程序终止了, 不会处理你的错误.

    assert 通常用在 debug 阶段, 用来判定一段程序的 “先决条件”, 满足了 assert 里面的条件, 才让程序继续, 否则一定是哪里错了, 程序员需要 debug, 直到不出错为止. launch 的debug 版本默认启用 assert.

    当代码到了 release 阶段, 一般不需要 assert 再进行判定了, 编译时加上宏定义 “#define NDEBUG”, 就可以把 assert 替换为空.

    exception 是程序运行过程中的异常处理, 异常就是可能会出现的 (而且不能通过debug方式避免的) 特殊情况, 比如尝试除以0, 硬盘没空间等等. 程序在处理这些异常的时候是运行着的, 没有终止. 异常处理的流程: throw (抛出) --》 try (检测) --》catch (捕获);

    貌似现在大部分情况不用 exception 了, 用 if 判断就好.


2. 二级指针作为“跳板”, 用来帮助 free 函数外部的 memory. 举例: img_t destruct
   
3. 函数参数为引用时, 必须在调用前初始化完成, 而且不能为NULL; 传指针是可以为NULL的.
   
4. malloc 与 free 成对使用, new 与 delete 成对使用. 二者行为上的区别是, new 申请内存时, 还会调用对象的构造函数, malloc 只会申请内存, delete 释放内存之前, 会调用对象的析构函数, free 只会释放内存.
   
5. 模板函数与函数指针
   
    函数指针是没法指向一个模板的, 因为 typedef 的时候, 模板还没有 type. 

    但是有一种 “在夹缝里” 的情况, 就是一个模板函数的返回类型和所有入参*都不带模板*, 只有函数体内部 “凭空” 用了模板. 这还真是模板函数, 它的使用方式也会有点不一样.

        template<typename T>
        void my_func(int a, int b){
            T* myPtr = (T*)malloc((a + b) * sizeof(T));
            // do something ...
            free(myPtr);
        }
   
    这时候, 不能通过入参的类型自动推断要用哪个实例, 但是可以通过一些条件判断决定调用哪个 type 的实例.

    问题来了, 如果在条件判断里面就直接调用该实例化的函数, 会有很多代码重复, 尤其是这个函数外面还有一两层 for loop 的时候. 条件判断里面套着循环, 而且每个条件判断底下的内容除了那个 type 不一样, 哪哪都一样. 这会增加 debug 的难度: 一旦要修改, 就要改好几处. 那么如何精简代码、并且让 “决定 type” 的动作与 “实际调用” 的动作分开 (这也是柯里化思想的体现), 这就需要函数指针.

    也正是因为这个“模板函数”的返回类型和入参都不带模板, 就可以理解为, 现在有一堆已经实例化的函数, 与普通函数无异, 它们的返回类型和入参的类型都一样. 这就可以用函数指针了.

    直接 typedef 函数指针:

        typedef void (*FP)(int, int);

    调用的时候这样:

        void test(){
            FP f = NULL;
            int a = 0, b = 0;
            if (...){
                f = my_func<uint8_t>;
            }
            else if (...){
                f = my_func<uint16_t>;
            }
            else if (...){
                f = my_func<uint32_t>;
            }
            else{
                f = my_func<int>;
            }

            // do something to a and b...
            f(a, b);
        }

6. 函数指针作为 struct 的成员

    声明的写法与 typedef 函数指针很像, 其实只是不用写 typedef 这个词而已.

        typedef struct{
            int height;
            int width;
            void (*f)(int, int); // pointer to function as a member
        } MyStruct_t;

    定义和使用的时候这样:

        void my_func_v2(int a){
            // do something...
        }

        void test(){
            // definition
            MyStruct_t sMyStruct;
            sMyStruct.f = my_func_v2; 

            // usage
            int a = 4;
            sMySruct.f(a);
        }

    注意:
    * 这里的函数指针虽然写法和 typedef 一样, 但是它*不是*一个 type, 只是一个 struct 里面的成员, 还是要依附于这个 struct 而存在的. 超过了这个 struct 的作用域, 这个函数指针“类型”也就不存在了 (在另外一个函数内, 若还想用到指向 my_func_v2 的指针, 那么, 还得搞个 MyStruct_t 的实体才行). 而 typedef 的函数指针如果写在当前的源文件, 那么在整个源文件都是有效的, 随时可以用. 
    * 上面的代码只是 MyStruct_t 的成员声明为函数指针, 并没有额外 typedef 函数指针类型, “sMyStruct.f = my_func_v2” 也起到*类似*前一段代码的 “FP f = my_func<uint8_t>” 的效果了 (只是作用域不同), 也就是说, 一个函数想要被指针指到, 不一定要 typedef 正儿八经的函数指针类型. 

7. “含有函数指针的 struct“ 的模板

    并不是一开始就想声明一个“含有函数指针的 struct“ 的模板, 而是