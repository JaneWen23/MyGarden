# C/C++ 碎碎念

1. assert 与 exception 的区别: 
   
   assert 的原型是
    
        void assert(int expression)

    如果 expression 为真, 那么不会触发任何东西, 程序继续运行; 如果为假, 那么它先向 stderr 打印一条出错信息, 再通过调用 abort 来终止程序的运行. 程序终止了, 不会处理你的错误.

    assert 通常用在 debug 阶段, 用来判定一段程序的 “先决条件”, 满足了 assert 里面的条件, 才让程序继续, 否则一定是哪里错了, 程序员需要 debug, 直到不出错为止. launch 的debug 版本默认启用 assert.

    当代码到了 release 阶段, 一般不需要 assert 再进行判定了, 编译时加上宏定义 “#define NDEBUG”, 就可以把 assert 替换为空.

    exception 是程序运行过程中的异常处理, 异常就是可能会出现的 (而且不能通过debug方式避免的) 特殊情况, 比如尝试除以0, 硬盘没空间等等. 程序在处理这些异常的时候是运行着的, 没有终止. 异常处理的流程: throw (抛出) --》 try (检测) --》catch (捕获);

    貌似现在大部分情况不用 exception 了, 用 if 判断就好.


2. 二级指针作为“跳板”, 用来帮助 free "函数外部的 memory":
    
    首先 free 涉及到对地址的操作 (毕竟 free 之后, 该地址的空间访问权没了), 需要格外小心. 如果一个函数的入参是一个一级指针, 我们又只对这个指针做 dereference 的操作, 只关心它所指向的内容, 那就没有什么好在意, 对指针的内容的操作, 就跟直接对它指向的变量操作一样. 当涉及到指针的地址, 事情就不一样了.
    
    传递指针是把实参的地址复制给了形参的内容, 一级指针的情况下, 如果想 free, 那么函数体内只能 free 了形参, 形参不能访问自己原来的地址了, 不会影响到实参, 实参自己的地址还能访问, 无解.
    
    结合下面的例子看, 如果是二级指针, 也就是形参指向的地址 (addr A) 不是要free的那个, 而 addr A 指向的那个地址 (addr B) 才是要 free 的那个, 也就是通过 addr A 跳转了一下. 这样形参顺着 addr A 找到了真正想 free 的那个, 直接在 addr B 上操作了, 不是操作形参自己的地址, 就达成了目的:

        // define
        void free_data(int** ptr_pData){
            // i.e., ptr_pData = &pData = "addr A",
            // *ptr_pData = *(&pData) = pData = "addr B", 
            assert(ptr_pData != NULL);
            free(*ptr_pData);
            *ptr_pData = NULL;
        }

        int main(){
            int* pData = (int*)malloc(100 * sizeof(int));
            //call
            free_data(&pData);
            return 0;
        }
   
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

    并不是一开始就想声明一个“含有函数指针的 struct“ 的模板, 而是被一个需求倒逼出来的.

    需求是这样的, 有 2 个入参和返回类型都带模板的模板函数, 它们的参列表形式都一样:

        template<typename T>
        T formula_v1(T a, T b){
            return a + b;
        }

        template<typename T>
        T formula_v2(T a, T b){
            return a * b;
        }

    想在另外一个函数内, 从以上两个模板选择一个, 进行实例化.

    这回不能直接 typedef 函数指针了, 因为它们都是板上钉钉的模板, 指针无法指向模板. 硬要 typedef 函数指针也很麻烦, 因为所有的 type 都要 typedef 一遍. 那么怎样才能 ”选到“ 一个模板函数呢? 这时候就 ”逼“ 出了 struct 的模板:

        template <typename T>
        struct Formulas_T{
            T (*formula)(T, T);
        };

    注意这里没有typedef, 因为还没有type; struct 内部是函数指针, 是带 T 的.
    
    有了这个 struct 的模板, 在需要的地方定义一个 struct 实体, 这样, 里面的函数指针的 type 也就跟着定下来了. 定了 type, 就可以像上面的例子一样指定函数名了.

        void test(){
            // definition
            Formulas_T<uint32_t> sMyFormula;
            sMyFormula.formula = formula_v1;

            // usage
            uint32_t a = 23, b = 30;
            uint32_t c = sMyFormula.formula(a, b);
        }
    
    这里 “sMyFormula.formula = formula_v1” 不用写成 “sMyFormula.formula = formula_v1<uint32_t>”, 当然如果写了也没错, 推荐不写.

    这个方法的精髓就是用 struct 的模板包装了函数指针, 使得 struct 内的函数指针可以有不确定的 type, 使用的时候分 “两步走”, 先定 type, 再定函数名字.

    不过缺点也是有的, 作用域嘛, 跟上一条讲的一样.

8. 不确定 type 的函数指针作为模板函数的参数

    其实不是真的把 “不确定 type 的函数指针” 作为参数传进来, 而是以 void* 形式传进来的, 不过, 既然已经是 void* 了, 说它是不确定 type 不过分吧.

        template<typename T>
        void perform_formula(void* fp){
            Formulas_T<T> sMyFormula;
            sMyFormula.formula = ??? // should get some info from fp
        }

    为什么要把它作为参数传进来, 是遇到了这样的情形: 在 perform_formula< T> 内, 需要选一个函数模板, 其实也就是要定下来两个东西, type 和函数名字: type 是随着 perform_formula< T> 的实例化就能自动定下来的, 但是函数名字自己确定不了, 要“外面”的 (就是那个 void* fp) 告诉我才行. 
    
    这并不是说, 函数名字是从外面传进来的, 因为毕竟只传进来了 void* (这是出于接口统一的考虑), 要想得到名字(也就是能用的函数入口地址), 还需要一点加工.
    
    补充一下, 这个要调用的函数连 type 带名字, 已经在外面定义好了, 只是需要在 perform_formula< T> 里面才调用:

        int main(){
            // definition
            Formulas_T<uint32_t> sExternalFormula;
            sExternalFormula.formula = formula_v2;
            
            //call
            perform_formula<uint32_t>((void*)sExternalFormula.formula); 

            return 0;
        }
        
    对于 perform_formula< T> 来说, 传参的时候, type 被隐去了, 不过问题不大, type 是可知的, 把名字搞定就可以了. 可是它不能直接等于 fp, 类型不同, 不能直接用等号连接.

    所以问题变成了: 如何把 void* fp 强制类型转换成我们需要的类型. 写代码的时候, T 还不确定, 就不能显式地写出类型. “隐式”的类型表示, 也就是要反推 sMyFormula.formula 的类型, 其实用 decltype 就好了:

        sMyFormula.formula = (decltype(sMyFormula.formula))fp;

    这样, 在 perform_formula< T> 函数内部, 就通过 decltype “重新选到” 了想要的模板函数.