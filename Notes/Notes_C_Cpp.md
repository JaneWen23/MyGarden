# C/C++ 碎碎念

1. assert 与 exception 的区别: 
   
   assert 的原型是
    ``` cpp
    void assert(int expression)
    ```

    如果 `expression` 为真, 那么不会触发任何东西, 程序继续运行; 如果为假, 那么它先向 `stderr` 打印一条出错信息, 再通过调用 `abort` 来终止程序的运行. 程序终止了, 不会处理你的错误.

    `assert` 通常用在 debug 阶段, 用来判定一段程序的 “先决条件”, 满足了 `assert` 里面的条件, 才让程序继续, 否则一定是哪里错了, 程序员需要 debug, 直到不出错为止. launch 的debug 版本默认启用 assert.

    当代码到了 release 阶段, 一般不需要 assert 再进行判定了, 编译时加上宏定义 `#define NDEBUG`, 就可以把 assert 替换为空.

    exception 是程序运行过程中的异常处理, 异常就是可能会出现的 (而且不能通过debug方式避免的) 特殊情况, 比如尝试除以0, 硬盘没空间等等. 程序在处理这些异常的时候是运行着的, 没有终止. 异常处理的流程: throw (抛出) --》 try (检测) --》catch (捕获);

    貌似现在大部分情况不用 exception 了, 用 if 判断就好.


2. 二级指针作为“跳板”, 用来帮助 free "函数外部的 memory":
    
    首先 free 涉及到对地址的操作 (毕竟 free 之后, 该地址的空间访问权没了), 需要格外小心. 如果一个函数的入参是一个一级指针, 我们又只对这个指针做 dereference 的操作, 只关心它所指向的内容, 那就没有什么好在意, 对指针指向的内容的操作, 就跟直接对它指向的变量操作一样. 当涉及到指针的地址, 事情就不一样了.
    
    传递指针是把实参的值(实参是指针, 其值是一个地址) 复制给了形参的值, 一级指针的情况下, 如果想 free, 那么函数体内只能 free 了形参, 形参不能访问自己原来的地址了, 不会影响到实参, 实参自己的地址还能访问, 无解.
    
    结合下面的例子看, 如果是二级指针, 也就是形参指向的地址 (addr A) 不是要free的那个, 而 addr A 指向的那个地址 (addr B) 才是要 free 的那个, 也就是通过 addr A 跳转了一下. 这样形参顺着 addr A 找到了真正想 free 的那个, 直接在 addr B 上操作了, 不是操作形参自己的地址, 就达成了目的:

    ``` cpp
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
    ```
   
3. 引用作为参数 vs 指针作为参数
   
   函数参数为引用时, 必须在调用前初始化完成, 而且不能为NULL; 传指针是可以为NULL的.
   
   如果一个已有的函数 (比如库函数) 的入参是引用, 但我们只有指针, 转化的方法是直接在入参处把指针 dereference.
   
4. `malloc` 与 `free` 成对使用, `new` 与 `delete` 成对使用. 二者行为上的区别是, `new` 申请内存时, 还会调用对象的构造函数, `malloc` 只会申请内存, `delete` 释放内存之前, 会调用对象的析构函数, `free` 只会释放内存.
   
5. 模板函数与函数指针
   
    函数指针是没法指向一个模板的, 因为 typedef 的时候, 模板还没有 type. 

    但是有一种 “在夹缝里” 的情况, 就是一个模板函数的返回类型和所有入参**都不带模板**, 只有函数体内部 “凭空” 用了模板. 这还真是模板函数, 它的使用方式也会有点不一样.

    ```cpp
    template<typename T>
    void my_func(int a, int b){
        T* myPtr = (T*)malloc((a + b) * sizeof(T));
        // do something ...
        free(myPtr);
    }
    ```
   
    这时候, 不能通过入参的类型自动推断要用哪个实例, 但是可以通过一些条件判断决定调用哪个 type 的实例.

    问题来了, 如果在条件判断里面就直接调用该实例化的函数, 会有很多代码重复, 尤其是这个函数外面还有一两层 for loop 的时候. 条件判断里面套着循环, 而且每个条件判断底下的内容除了那个 type 不一样, 哪哪都一样. 这会增加 debug 的难度: 一旦要修改, 就要改好几处. 那么如何精简代码、并且让 “决定 type” 的动作与 “实际调用” 的动作分开 (这也是柯里化思想的体现), 这就需要函数指针.

    也正是因为这个“模板函数”的返回类型和入参都不带模板, 就可以理解为, 现在有一堆已经实例化的函数, 与普通函数无异, 它们的返回类型和入参的类型都一样. 这就可以用函数指针了.

    直接 typedef 函数指针:
    ```cpp
    typedef void (*FP)(int, int);
    ```

    调用的时候这样:
    ``` cpp
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
    ```

6. 函数指针作为 struct 的成员

    声明的写法与 typedef 函数指针很像, 其实只是不用写 typedef 这个词而已.
    ```cpp
    typedef struct{
        int height;
        int width;
        void (*f)(int, int); // pointer to function as a member
    } MyStruct_t;
    ```

    定义和使用的时候这样:
    ```cpp
    void my_func_v2(int a){
        // do something...
    }

    void test(){
        // definition
        MyStruct_t sMyStruct;
        sMyStruct.f = my_func_v2; 

        // usage
        int a = 4;
        sMyStruct.f(a);
    }
    ```

    注意:
    * 这里的函数指针虽然写法和 typedef 一样, 但是它**不是**一个 type, 只是一个 struct 里面的成员, 还是要依附于这个 struct 而存在的. 超过了这个 struct 的作用域, 这个函数指针也就不存在了 (在另外一个函数内, 若还想用到指向 `my_func_v2` 的指针, 那么, 还得搞个 `MyStruct_t` 的实体才行). 而 typedef 的函数指针如果写在当前的源文件, 那么在整个源文件都是有效的, 随时可以用. 
    * 上面的代码只是 `MyStruct_t` 的成员声明为函数指针, 并没有在全局 typedef 函数指针类型, `sMyStruct.f = my_func_v2` 也起到**类似**前一段代码的 `FP f = my_func<uint8_t>` 的效果了 (只是作用域不同), 也就是说, 一个函数想要被指针指到, 不一定要 typedef 正儿八经的全局的函数指针类型. 

7. “含有函数指针的 struct“ 的模板

    并不是一开始就想声明一个“含有函数指针的 struct“ 的模板, 而是被一个需求倒逼出来的.

    需求是这样的, 有 2 个入参和返回类型都带模板的模板函数, 它们的参列表形式都一样:
    ```cpp
    template<typename T>
    T formula_v1(T a, T b){
        return a + b;
    }

    template<typename T>
    T formula_v2(T a, T b){
        return a * b;
    }
    ```
    想在另外一个函数内, 从以上两个模板选择一个, 进行实例化.

    这回不能直接 typedef 函数指针了, 因为它们都是板上钉钉的模板, 指针无法指向模板. 硬要 typedef 函数指针也很麻烦, 因为所有的 type 都要 typedef 一遍. 那么怎样才能 ”选到“ 一个模板函数呢? 这时候就 ”逼“ 出了 struct 的模板:
    ```cpp
    template <typename T>
    struct Formulas_T{
        T (*formula)(T, T);
    };
    ```

    注意这里没有typedef, 因为还没有type; struct 内部是函数指针, 是带 T 的.
    
    有了这个 struct 的模板, 在需要的地方定义一个 struct 实体, 这样, 里面的函数指针的 type 也就跟着定下来了. 定了 type, 就可以像上面的例子一样指定函数名了.
    ```cpp
    void test(){
        // definition
        Formulas_T<uint32_t> sMyFormula;
        sMyFormula.formula = formula_v1;

        // usage
        uint32_t a = 23, b = 30;
        uint32_t c = sMyFormula.formula(a, b);
    }
    ```
    
    这里 `sMyFormula.formula = formula_v1` 不用写成 `sMyFormula.formula = formula_v1<uint32_t>`, 当然如果写了也没错, 推荐不写.

    这个方法的精髓就是用 struct 的模板包装了函数指针, 使得 struct 内的函数指针可以有不确定的 type, 使用的时候分 “两步走”, 先定 type, 再定函数名字.

    不过缺点也是有的, 作用域嘛, 跟上一条讲的一样.

8.  函数的作用域 vs 函数指针(一个变量)的作用域

    在 C/C++ 中, 函数默认是全局的 (只要它没在哪个函数内部被定义), 作用域是整个文件域, 在源文件里可随地使用. 而上述 `Formulas_T<uint32_t> sMyFormula` 的作用域是局部的, 当超出 `sMyFormula` 的作用域时, 只是不可以通过 `sMyFormula.formula` 来找到 `formula_v1<uint32_t>` 了, 函数还是在的. 只要它的入口地址保存在指针中, 什么时候想用这个函数还是可以用的.  

9.  不确定 type 的函数指针作为模板函数的参数

    其实不是真的把 “不确定 type 的函数指针” 作为参数传进来, 而是以 void* 形式传进来的, 不过, 既然已经是 void* 了, 说它是不确定 type 不过分吧.
    ```cpp
    template<typename T>
    void perform_formula(void* fp){
        Formulas_T<T> sMyFormula;
        sMyFormula.formula = ??? // should get some info from fp
    }
    ```

    为什么要把它作为参数传进来, 是遇到了这样的情形: 在 `perform_formula<T>` 内, 需要选一个函数模板, 其实也就是要定下来两个东西, type 和函数名字: type 是随着 `perform_formula<T>` 的实例化就能自动定下来的, 但是函数名字自己确定不了, 要“外面”的 (就是那个 `void* fp`) 告诉我才行. 
    
    这并不是说, 函数名字是从外面传进来的, 因为毕竟只传进来了 void* (这是出于接口统一的考虑), 要想得到名字(也就是能用的函数入口地址), 还需要一点加工.
    
    补充一下, 这个要调用的函数连 type 带名字, 已经在外面定义好了, 只是需要在 `perform_formula<T>` 里面才调用:
    ```cpp
    int main(){
        // definition
        Formulas_T<uint32_t> sExternalFormula;
        sExternalFormula.formula = formula_v2;
        
        //call
        perform_formula<uint32_t>((void*)sExternalFormula.formula); 

        return 0;
    }
    ```
    对于 `perform_formula<T>` 来说, 传参的时候, type 被隐去了, 不过问题不大, type 是可知的, 把名字搞定就可以了. 可是它不能直接等于 fp, 类型不同, 不能直接用等号连接.

    所以问题变成了: 如何把 `void* fp` 强制类型转换成我们需要的类型. 写代码的时候, T 还不确定, 就不能显式地写出类型. “隐式”的类型表示, 也就是要反推 `sMyFormula.formula` 的类型, 其实用 `decltype` 就好了:
    ```cpp
    sMyFormula.formula = (decltype(sMyFormula.formula))fp;
    ```

    这样, 在 `perform_formula<T>` 函数内部, 就通过 `decltype` “重新选到” 了想要的模板函数.

10. void* 指向任意的函数
    
    上面一条说, 那个要被调用函数是事先在 “外面” 定义好的, 然后又转为 void* 型, 作为参数传递. 既然要转为 void*, 我们其实不用把定义写得那么一板一眼(不用把类型写得那么完备), 直接定义一个 void* 也可以:
    ```cpp
    void* fp = formula_v2<uint32_t>;
    // equivalent to:
    // Formulas_T<uint32_t> sExternalFormula;
    // sExternalFormula.formula = formula_v2;
    // void* fp = (void*)sExternalFormula.formula;
    ```
    反正对编译器来说, `formula_v2<uint32_t>` 就已经提供了名字和type, 就知道是哪一个函数了, 用 `void*` 指向它, 没毛病.

    其实, 任意一个函数, 在**没有定义函数指针**的情况下, 都可以被 `void*` 指向. 这就是为什么当我们只需要一个 `void*` 的时候, 不需要写 `Formulas_T<uint32_t> sExternalFormula` 来实例化一个 “含有函数指针的 struct” 然后指向 `formula_v2<uint32_t>`.

    这是不是意味着 `Formulas_T<uint32_t> sExternalFormula` 就一无是处了呢? 不是的. 当我们真正要调用 `formula_v1<uint32_t>` 的时候, 还是需要 type 的, `void*` 是不能让你的函数被调用的, 要强制类型转换才行.  

11. 模板也要柯里化
 
    <!-- 如果有两个 type 需要确定, 可以分两个函数确定, 不然他俩的排列组合要死人的 -->
    假设有这样一个模板函数, 需要两个type, 并且入参不含type:
    ```cpp
    template<typename Tsrc, typename Tdst>
    void func(int a, int b){
        //...
    }
    ```
    要想定义函数指针, 指向实例化的函数, 根据条件判断, 决定指针指向哪个实例.

    指针好说, 因为入参和返回值都不带 type. 条件判断的时候, 如果把所有的 type 排列组合都列一遍, 是要死人的. 用柯里化的思想优雅地解决问题:
    ```cpp
    // (continued)
    typedef void (*FP)(int, int);

    template<typename T>
    void perform_func(int a, int b){
        FP f = NULL;
        if (...){
            f = func<T, uint8_t>
            // uint8_t will be passed to "Tdst"
        }
        else if (...){
            f = func<T, uint16_t>
            // uint16_t will be passed to "Tdst"
        }
        else if (...){
            f = func<T, uint32_t>
            // uint32_t will be passed to "Tdst"
        }

        f(a, b);
    }

    typedef void (*FP_2)(int, int);

    int main(){
        int a = 32;
        int b = 34;
        FP_2 g = NULL;
        if (...){
            g = perform_func<uint8_t>; 
            // uint8_t will be eventually passed to "Tsrc"
        }
        else if (){
            g = perform_func<uint16_t>; 
            // uint16_t will be eventually passed to "Tsrc"
        }
        else if (){
            g = perform_func<uint32_t>;
            // uint16_t will be eventually passed to "Tsrc"
        }

        g(a, b);
    }
    ```
    就是分两步选择, 第一步选择是在 `main()` 中, 先定下来其中一个 type, 第二步选择是在 `perform_func()` 中, 定下来另一个 type. 这样就避免了列出所有的排列组合.

12. 模板要放在头文件, 如果想要被外部调用的话. (如果不想被外部调用, 就放在源文件)


<!-- option + Z :触发软换行 -->

13. 当返回一个 const char* 时, 发生了什么:

    例子:  
    ```cpp
    const char* get_image_format_name(const IMAGE_FMT imageFormat){
        switch (imageFormat){
            case MONO:{
                return "MONO";
            }
            // ...
        }
    }

    int main(){
        std::cout<<"imageFormat = "<< get_image_format_name(MONO)<<"\n";
        return 0;
    }
    ```
    这是正确的, 顺便提一下 cout 输出的时候, `get_image_format_name(MONO)` 是常量字符串的首地址, 但还是把整个字符串打印出来了, 这是因为 cout 的 `<<` 重载了, `cout<<`一个`char*`型的指针变量会被翻译为输出该指针指向的字符串.

    复习一下, 一个由 C/C++ 编译的程序占用内存分为以下几个部分:  
    1) 栈区 stack -- 由编译器自动分配释放, 存放函数的参数值, 局部变量的值等. 其操作方式类似于数据结构中的栈.  
    2) 堆区 heap -- 一般由程序员自动扽配释放, 若程序员不释放, 程序结束时可能由操作系统回收. 它与数据结构的堆是两回事, 分配方式类似于链表.  
    3) 全局区(静态区) static -- 全局变量和静态变量的存储都在这里, 初始化的全局变量和静态变量放在一块区域, 未初始化的放在相邻的另一块区域. 程序结束后由操作系统释放.  
    4) 文字常量区 -- 常量字符串放在这, 程序结束后由操作系统释放.  
    5) 程序代码区 -- 存放函数体的二进制代码.
    
    本例中的 `"MONO"` 存放在文字常量区, 所以在 main 中调用 `get_image_format_name()` 时, 返回的 `const char*` 指向的内容仍然存在.

    不过还是尽量用 `std::string` 吧. 一定要 C style 的时候再转化为 `const char*`.

14. std::vector
    vector is FILO(First in last out)(stack)? can only push_back() and pop_back().
    the last element v.back() is equal to *(v.end()-1), note: end() will get the pointer at last element position + 1.

    the last element in stack: s.top(), not back().

    std::queue
    std::queue q;
    q.front() // no q[0]; is it iterator (pointer) or object????????
    q.emplace(), or q.push()// no insert(), no push_back()
    q.pop() // no pop_front()
    循环队列: q.push(q.front()); q.pop();

    what is an iterator:

15. std::unordered_map, std::pair, std::unordered_set, std::any
    std::pair.first // no pair.first()
    std::pair.second // no pair.second()
    std::set    insert() 插入同样的键两次, 第二次会被忽略; count() 返回值只有0或1
    to traverse a set, you can only use `auto it = s.begin(); it != s.end(); ++it;` no s[0], no s.at()
    
16. DAG 与 topological sort
17. 类的构造函数显式调用(一种是vector(n), 一种是new[]什么的), 类的继承
18. 同一个类的不同构造函数嵌套调用: 只按照正常的函数调用方式是不行的, 会出现内存泄漏问题, 要加上 ”new (this)“ 关键字. (为什么?)
    `new` 的用法, 以及当 `new` 一个对象的时候, 发生了什么
    new 3种用法, 一个是 new operator, 一个是 operator new (是函数), 一个是 placement new;
    new 一个对象就是在堆区分配内存, 不用new的创建对象就是在栈上分配内存, 栈上能分配的内存比较小; 堆上创建的对象好像是匿名的, 栈上创建的有名字;
    既然在堆区分配了, 就必须在用完的时候用 delete 删掉;
    一般来说，使用new申请空间时，是从系统的“堆”（heap）中分配空间。申请所得的空间的位置是根据当时的内存的实际使用情况决定的。但是，在某些特殊情况下，可能需要在已分配的特定内存创建对象，这就是所谓的“定位放置new”（placement new）操作. (既可以放置在栈上也可以放置在堆上)
    所谓placement new就是在用户指定的内存位置上构建新的对象，这个构建过程不需要额外分配内存，只需要调用对象的构造函数即可。(不分配内存, 只调用构造函数)
    和其他普通的new不同的是，它在括号里多了另外一个参数。比如：
    ```cpp
    Widget * p = new Widget; //ordinary new
    pi = new (ptr) int; //placement new
    ```

    一般的new, 比如  A* pa = new A;  实际上有三个过程: 
    (1)调用operator new分配内存，operator new (sizeof(A)) 
    (2)调用构造函数生成类对象，A::A() (初始化)
    (3)返回相应指针 
    事实上，分配内存这一操作就是由operator new(size_t)来完成的，如果类A重载了operator new，那么将调用A::operator new(size_t )，否则调用全局::operator new(size_t )，后者由C++默认提供.
    placement new，本质上是对operator new的重载，定义于#include <new>中。它不分配内存，调用合适的构造函数在ptr所指的地方构造一个对象，之后返回实参指针ptr

    分配空间不是构造函数被调用的时候分配的, 而是进入大括号的时候分配的, 构造函数只是初始化这一块空间.
    

19.  现在推荐在构造函数抛出异常（但千万不要在析构函数抛出异常）。构造函数抛出异常的话，申请的内存会自动被清掉，所以不用担心内存泄露。另外，C++ 要抛异常一定要配合使用 RAII，这一条同样也适用于构造函数抛异常。有了 RAII 就不怕泄露啦。主的好处就是用这个对象的时候不用担心是否初始化成功，只要是有这么一个对象，它就是可用的。

作者：陈乐群
链接：https://www.zhihu.com/question/430968645/answer/1583198219
来源：知乎
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

20. class, static function meaning??????    类的成员函数 const 加在后面/前面/前后都加
21. 
    vector of vector 初始化:

    ```cpp
    int h = 10;
    int w = 12;
    vector<vector<int>> vStatus(h, vector<int>(w, 0));
    ```


18. c++ 的 struct 也可以定义函数, 默认是 public 的; 其中比较实用的是构造函数.

```cpp
  // Definition for a binary tree node:
  struct TreeNode {
       int val;
       TreeNode *left;
       TreeNode *right;
       TreeNode() : val(0), left(nullptr), right(nullptr) {}
       TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
       TreeNode(int x, TreeNode *left, TreeNode *right) : val(x), left(left), right(right) {}
  };
```

19. 高级的 for loop
    
   一种是  
   ```cpp
   for (auto it = hash.begin(); it != hash.end(); ++it){
        // do something
   }
   ```
   还有一种是
   ```cpp
   std::vector<std::string>& emails;
   for (auto &email : emails){
        for (char c: email){
            // do something
        }
   }
   ```
    
    至于什么时候用 auto, 作为 iterator 的时候就用; 或者使用第三方库的时候, 不是很确定什么type, 也不是很在意什么type, 就用一下的情况.