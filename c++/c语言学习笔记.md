##C语言学习笔记

###C代码运行流程

1. 编码
2. 编译: gcc hello.c -o hello
3. 运行:./hello

###输入输出函数

* puts("my name is jiong");
* scanf("%10s"， name);
* print("you name is %s", name);

###字符串定义

* char name[10];

###单引号和双引号

* 单引号表示字符
* 双引号表示字符串

###引用库

* 标准输入输出： stdio.h

###tips

* 检查程序退出状态: linux--echo $?
* C语言中任何不为0的值都为真
* & 和 | 除了进行位运算之外，在做布尔运算时总能计算两个条件
* 链式赋值： x = y = 4;

###一些函数

* atoi(strNum) - 将文本转化为数字
* sizeof(val) - 获取变量占用的内存空间，是一个运算符
* fgets(val, sizeof(val), stdin) --标准输入中读入数据
* fprintf(stdout/stderr, "data"); -- 和printf一样，不过可以决定数据输出到哪里

###指针（重点）

####使用指针的原因

* 函数调用时，指传递一个指针，不用传递整份数据
* 使两段代码处理同一条数据，而不是备份

####定义

* int *address_of_x = &x;

####符号

* &-引用
* *-解引用

####字符串定义

* char msg[] = "Hello World!";
* msg代表首字符的地址
* char *t = msg
  sizeof(t) != sizeof(msg)

####数组

	int drinks[] = {2, 3, 4};
	printf("%i ", drinks[1]) === printf("%i", *(drinks+1));

####PS

* 局部变量存储在栈中，全局变量存储在全局量段中
* print("x save in menory address is %p\n", &x);


###字符串类库

####引入：include <string.h>

####定义字符串数组

	* char names[][] = {};
	* char *names[] = {};

####相关函数

* strstr(）- 返回字符串中b在字符串a中的位置 
* strcmp() - 比较字符串
* strchr() - 在字符串中找到某个字符的位置
* strcat() - 连接字符串
* strcpy() - 复制字符串
* strlen() - 获取字符串长度

###文件操作

####打开文件

* FILE *in = fopen("flie_path", "mode");
* 文件模式：r(read),w(write),a(append)

####关闭文件

* fclose(in);

###命令行参数读取

* 方法一

	* int main(int argc, char *argv[]) -- argv[0] 问当前文件名

* 方法二

	* 使用getopt
	* 需要引入：#include <unistd.h>
	* 用法：ch = getopt(argc, argv, "ae:") -- 代表接受a和e选项，其中e选项要接受一个参数
	* 变量：
		* optarg -- 保存选项提供的参数
		* optind -- 保存了接受的参数的个数
