# 记一次PHP语法问题追查过程

## 问题

[V2EX](https://www.v2ex.com/ "V2EX")上有这样的帖子《[给 phper 出一道基本的面试题, 做错了得加强基础了](https://www.v2ex.com/t/349774 "给 phper 出一道基本的面试题, 做错了得加强基础了")》，答案是啥？`6+59+7`?`6+516`？,动手查看如下：

![](http://i.imgur.com/eQwCVWi.png)

答案是：**13**，why?


## 追查

### 使用vld（[安装和使用](http://blog.csdn.net/21aspnet/article/details/7002644 "安装和使用")）查看opcode代码如下

![](http://i.imgur.com/j4Hl6jo.png)

*php版本：7.1.15*

可以看到，编译后的`opcode`的代码，`ADD`的左操作数为`6+%2B+59`，原因如图所示

![](http://i.imgur.com/dewlplA.png)

加号（`+`)和字符串连接号(`.`)位于同一级，且结合顺序为**从左到右**，所以`.`先结合，就成为`6+%2B+59`

### 查看`opcode`定义
查看在`zend/zend_vm_def.h`中`ZEND_ADD`的定义

	ZEND_VM_HANDLER(1, ZEND_ADD, CONST|TMPVAR|CV, CONST|TMPVAR|CV)
	{
	    USE_OPLINE
	    zend_free_op free_op1, free_op2;
	    zval *op1, *op2, *result;
	
	    op1 = GET_OP1_ZVAL_PTR_UNDEF(BP_VAR_R);
	    op2 = GET_OP2_ZVAL_PTR_UNDEF(BP_VAR_R);
	    if (EXPECTED(Z_TYPE_INFO_P(op1) == IS_LONG)) {
			...
	    } else if (EXPECTED(Z_TYPE_INFO_P(op1) == IS_DOUBLE)) {
			...
	    }
	
	    SAVE_OPLINE();
	    if (OP1_TYPE == IS_CV && UNEXPECTED(Z_TYPE_INFO_P(op1) == IS_UNDEF)) {
	        op1 = GET_OP1_UNDEF_CV(op1, BP_VAR_R);
	    }
	    if (OP2_TYPE == IS_CV && UNEXPECTED(Z_TYPE_INFO_P(op2) == IS_UNDEF)) {
	        op2 = GET_OP2_UNDEF_CV(op2, BP_VAR_R);
	    }
	    add_function(EX_VAR(opline->result.var), op1, op2);
	  	...
	}
	
	-- 省略部分逻辑处理代码

可以看到，当操作数不为`long`或者`double`的时候，会执行`GET_OP1_UNDEF_CV(op1, BP_VAR_R);`方法， 这个预定义常量定义在`zend/zend_execute.c`中，其结构会将操作`op1`转换为`zend_string`类型

然后将处理好的参数传入到`add_function`中，此方法定义在`zend/zend_operators.c`中，具体如下


	ZEND_API int ZEND_FASTCALL add_function(zval *result, zval *op1, zval *op2) /* {{{ */
	{
	    zval op1_copy, op2_copy;
	    int converted = 0;
	
	    while (1) {
	        switch (TYPE_PAIR(Z_TYPE_P(op1), Z_TYPE_P(op2))) {
	            case TYPE_PAIR(IS_LONG, IS_LONG):
	                ...
	            case TYPE_PAIR(IS_LONG, IS_DOUBLE):
	        		...
	            case TYPE_PAIR(IS_DOUBLE, IS_LONG):
	             	...
	            case TYPE_PAIR(IS_DOUBLE, IS_DOUBLE):
	                ...
	            case TYPE_PAIR(IS_ARRAY, IS_ARRAY):
	                ...
	            default:
	                if (Z_ISREF_P(op1)) {
	                    op1 = Z_REFVAL_P(op1);
	                } else if (Z_ISREF_P(op2)) {
	                    op2 = Z_REFVAL_P(op2);
	                } else if (!converted) {
	                    ZEND_TRY_BINARY_OBJECT_OPERATION(ZEND_ADD, add_function);
	
	                    if (EXPECTED(op1 != op2)) {
	                        zendi_convert_scalar_to_number(op1, op1_copy, result, 0);
	                        zendi_convert_scalar_to_number(op2, op2_copy, result, 0);
	                    } else {
	                        zendi_convert_scalar_to_number(op1, op1_copy, result, 0);
	                        op2 = op1;
	                    }
	                    converted = 1;
	                } else {
	                    zend_throw_error(NULL, "Unsupported operand types");
	                    return FAILURE; /* unknown datatype */
	                }
	        }
	    }
	}

当参数不为`array,long,double`的时候，会调用`zendi_convert_scalar_to_number`转换数字，改函数定义在同一文件下：

	#define zendi_convert_scalar_to_number(op, holder, result, silent)  \
    if (Z_TYPE_P(op) != IS_LONG) {                                  \
        if (op==result && Z_TYPE_P(op) != IS_OBJECT) {              \
            _convert_scalar_to_number(op, silent);                  \
        } else {                                                    \
                case IS_STRING:                                     \
                    if ((Z_TYPE_INFO(holder)=is_numeric_string(Z_STRVAL_P(op), Z_STRLEN_P(op), &Z_LVAL(holder), &Z_DVAL(holder), silent ? 1 : -1)) == 0) {  \
                        ZVAL_LONG(&(holder), 0);                    \
                        if (!silent) {                              \
                            zend_error(E_WARNING, "A non-numeric value encountered");   \
                        }                                           \
                    }                                               \
                    (op) = &(holder);                               \
                    break;                                          \
 
            }                                                       \
        }                                                           \
    }

可以看到，在进行加法操作的时候，如果遇到操作数为字符串的时候，会返回遇到第一个非数字字符前的所有字符所组成的数字

## 结论
原帖说这是个phper的基础面试题目，但是笔者认为实际开发中并没有人会这么写，如果有，那更多的说明是个人代码习惯的问题了

## 扩展阅读
* [PHP内核分析：Zend虚拟机](http://developer.51cto.com/art/201703/533446.htm "PHP内核分析：Zend虚拟机")
* [PHP 5.6新特性之一：内部操作符重载](https://segmentfault.com/a/1190000000432002 "PHP 5.6新特性之一：内部操作符重载")
* [VLD扩展使用指南](http://www.phppan.com/2011/05/vld-extension/ "VLD扩展使用指南")
* [PHP安装与使用VLD查看opcode代码【PHP安装第三方扩展的方法】](http://blog.csdn.net/21aspnet/article/details/7002644 "PHP安装与使用VLD查看opcode代码【PHP安装第三方扩展的方法】")
* [认识PHP 7虚拟机](http://yangxikun.github.io/php/2016/11/04/php-7-engine.html "认识PHP 7虚拟机")
