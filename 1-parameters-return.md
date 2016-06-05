

# 接收参数、返回参数

php扩展中通过zend_parse_parameters函数来接收参数，就像c语言中printf，把接收到参数格式化。接收多个参数由写成`zend_parse_parameters(ZEND_NUM_ARGS(),"sbldzrao",.....)`。

```c
b   Boolean
l   Integer 整型
d   Floating point 浮点型
s   String 字符串
S	zend_string 结构体
r   Resource 资源
a   Array 数组
o   Object instance 对象
O   Object instance of a specified type 特定类型的对象
z   Non-specific zval 任意类型～
Z   zval**类型
f   表示函数、方法名称
|	后台的参数为可选
/   表明参数解析函数将会对剩下的参数以 SEPARATE_ZVAL_IF_NOT_REF()的方式来提供这个参数的一份拷贝， 除非这些参数是一个引用。
!   表明剩下的参数允许被设定为 NULL（仅用在a、o、O、r和z身上）
```

**字符串**

zend_string 结构使用大写S来表示，不需要再定义一整数来表示字符串长度。

```c
zend_string * str;
if (zend_parse_parameters(ZEND_NUM_ARGS(),"S",&str) == FAILURE) {
  return;
}
zend_printf("%s",str->val);
RETURN_STR(str);
```

小写s表示接收char指针，需要定义一个整型来表示字符串的长度。使用RETURN_STRING(str)来返回字符串，RETURN_STRINGL(str,1)返回时会截取指定的长度。

```c

char * str;
long len;
if (zend_parse_parameters(ZEND_NUM_ARGS(),"s",&str,&len) == FAILURE) {
  return;
}
zend_printf("str:%s,len:%d",str,len);
RETURN_STRING(str);
```



**数字**

*整型*

php中整型使用long，当然可以用int，只要保证长度不超过int的范围即可。使用RETURN_LONG(l)返回。

```c
long l;
if (zend_parse_parameters(ZEND_NUM_ARGS(),"l",&l) == FAILURE) {
  return;
}
zend_printf("%d",l);
RETURN_LONG(l)
```

*浮点型*

php使用double作为符点型，当然只要你遵循c语言类型隐式转换规则，任何浮点类型都可以。使用RETURN_DOUBLE(x)做为返回函数。

```c
double d;
if (zend_parse_parameters(ZEND_NUM_ARGS(),"d",&d) == FAILURE) {
	return;
}
zend_printf("%.3f",d);
RETURN_DOUBLE(d)
```



**数组**

接收数组类型与以上几种类型不同，需要定义zval。使用RETURN_ARR(x)返回。如果定义zval来接收的，推荐使用RETURN_ZVAL做为返回，因为这个宏函数可以更方便的帮助管理内存。

```c
zval *arr;
if (zend_parse_parameters(ZEND_NUM_ARGS(),"a",&arr) == FAILURE) {
	return;
}
php_var_dump(arr,1);
RETURN_ZVAL(arr,0,1);
```



**任意类型**

任意类型与数组方式差不多。区别就在于使用z为作为接收格式。

```c
zval *zzz;
if (zend_parse_parameters(ZEND_NUM_ARGS(),"z",&zzz) == FAILURE) {
	return;
}
php_var_dump(zzz,1);
RETURN_ZVAL(zzz,0,1);
```



##### 类型判断

php内核中使用Z_TYPE(x)来取出zval的类型，如果是指针类型则使用Z_TYPE_P(x)。在Zend/zend_types.h中找到如下代码定义。

```c
#define IS_UNDEF					0	/* 是否未定义，zval arr = {{0}}此时为未定义 */
#define IS_NULL						1	/* 是否为空 */
#define IS_FALSE					2	/* 是否为false */
#define IS_TRUE						3	/* 是否为true */
#define IS_LONG						4	/* 是否为整型 */
#define IS_DOUBLE					5	/* 是否为浮点类型 */
#define IS_STRING					6	/* 是否为字符串 */
#define IS_ARRAY					7	/* 是否为数组 */
#define IS_OBJECT					8	/* 是否为对象 */
#define IS_RESOURCE					9	/* 是否为资源类型 */
#define IS_REFERENCE				10	/* 是否为引用 */
```

使用方法：

```c
switch (Z_TYPE_P(offset)) {
	case IS_STRING:
		......
	case IS_DOUBLE:
		......
	case IS_LONG:
		......
	case IS_FALSE:
		......
	case IS_TRUE:
		......
	case IS_REFERENCE:
		......
	case IS_RESOURCE:
		......
}
```



##### 参数校验

如果你使用zend_parse_parameters来接收参数，那么你是不需要ZEND_BEGIN_ARG_INFO_EX宏来进行入参数校验的。但如果使用的是zend_get_parameters那么就需要zend帮你自动校验了。

ZEND_BEGIN_ARG_INFO_EX需要四个参数，第一个为名字，第二个在php7中已经没用了，完全是为了向下兼容，第三个是返回参数是否是引用，第四则是参数的个数。ZEND_ARG_INFO(pass_by_ref, name) 第一个参数为是否引用，0否，1是，第二个参数为名称。

```c
ZEND_BEGIN_ARG_INFO_EX(a,0,0,3) //接收三个参数
	ZEND_ARG_ARRAY_INFO(0,config,0) //强制接收数组
	ZEND_ARG_INFO(0,name) //
	ZEND_ARG_OBJ_INFO(0,obj,0) // 接收对象
ZEND_END_ARG_INFO() //结束
```

记得在PHP_FE中加入校验名。

```c
const zend_function_entry test_functions[] = {
	PHP_FE(confirm_test_compiled,	NULL)		/* For testing, remove later. */
        PHP_FE(hello,a)
	PHP_FE_END	/* Must be the last line in test_functions[] */
};
```

