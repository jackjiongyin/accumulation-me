# API 和 ABI

## API

程序编程接口。 一个API是不同代码片段的连接纽带。它定义了一个函数的参数，函数的返回值，以及一些属性比如继承是否被允许。 因此API是用来约束编译器的：一个API是给编译器的一些指令，它规定了源代码可以做以及不可以做哪些事。在说到API的时候，也会涉及到函数的条件，行为以及出错环境。从这个角度看， 一个API也可以看作是被人使用：一个API是给程序员的一些指令，它规定了函数需要什么以及会做什么。


## ABI

程序二进制接口。 一个ABI是不同二进制片段的连接纽带。 它定义了函数被调用的规则：参数在调用者和被调用者之间如何传递，返回值怎么提供给调用者，库函数怎么被应用，以及程序怎么被加载到内存。 因此ABI是用来约束链接器的：一个ABI是无关的代码如何在一起工作的规则。 一个ABI也是不同进程如何在一个系统中共存的规则。 举例来说，在Linux系统中，一个ABI可能定义信号如何被执行，进程如何调用syscall，使用大端还是小端，以及栈如何增长。从这个角度看，一个API是用来约束在一个特定架构上操作系统的一系列规则.***即规定系统应该怎样运行*


## REF

	Hello! 
	
	2013/12/31 aCayF: 
	> 1.手工筛选API,那这个筛选源来自哪里呢?也像我这样预先展开头文件吗? 
	> 
	
	真正的 ABI 源自 ABI 的创建者。作为用户，严格来说只能自己封装一层作为自己信任的 ABI. 
	当然，现实世界中也可以尽量只使用最核心最不容易变化的 API，同时尽可能脱掉各种上层的 C 类型信息，尽量使用所谓的 opaque 
	types 和基本 C 数据类型（最好是确定数据长度的）。 
	
	比如 Nginx 官方明确表示 Nginx 目前并不提供 ABI，这就是为什么 ngx_lua 模块里面自己用 C 封装了一层 ABI，供 
	lua-resty-core 库通过 FFI 来调用。 
	
	> 2.API和ABI有什么区别吗? 
	> ----针对这个问题我搜过wiki,目前知道API是源代码层面的接口,而ABI是机器码层面的接口,常指调用外部动态库时的接口,可以理解ABI是API编译后的版本 
	> 
	
	ABI 是更为稳固更为底层的接口。比如纯 C++ 编写的库严格来说就根本不存在真正意义上的 ABI，因为 C++ name 
	mangling、C++ 异常处理等等在二进制层面上都不存在标准的可以互操作的标准，不同编译器的实现细节很可能是不兼容的。 
	
	> 3.为什么要筛选API,使得它比较像简单和可靠的ABI? 
	> ----针对这个问题,在我搜过google之后,我的看法是,一来是由于库的提供者会对库进行升级,二来是库的使用者他们使用的库的版本也会存在差异,所以我们要尽量挑选binary-compatible的ABI,望指正:) 
	> 
	
	是的。FFI 层面上一般调用的是 ABI 而不是 API. C 库的 API 都是通过 C 文件提供的，期望用户使用真正的 C 
	编译器工具链来编译用户代码。而使用 LuaJIT FFI 时，在 Lua VM 一侧并没有直接使用 C 编译器工具链，也没有使用库的 C 
	头文件。所以你应当首先确定 ABI，而不是 C 等高级编程语言层面上的 API. 因为 API 对于 FFI 来说其实并没有太大的意义。 
	
	> 4.如何能判断出这个API生成的ABI是简单和可靠的呢? 
	> ----这里引申一个问题就是原型相同的API在相同平台上生成的ABI是一致的吗? 
	> 
	
	最简单的一条规则就是 ABI 与具体的 C 等高级语言的编译器的具体实现无关，与 C 层面上的宏控制选项无关。要足够的稳定。否则当 C 
	库的编译选项或者使用的编译器发生变化时，你的 Lua FFI 库就会出问题。 
	
	> 5.用C先自己封装一下又是什么意思? 
	> ----我的理解是封装出一个比较像简单和可靠ABI的API(前提是库的源代码已经提供了吧,封装后再重新编译一次),春哥有示例可以借鉴一下吗? 
	> 
	
	你可以参考 ngx_lua 模块源码中的那些以 ngx_http_ffi_ 开头的 C 函数的原型和实现。这些函数都被设计成 nginx 
	某种意义上的 ABI，供 lua-resty-core 库通过 FFI 使用。当然，你也可以同时参考 lua-resty-core 
	库的具体实现。 
	
	Regards, 
	-agentzh 