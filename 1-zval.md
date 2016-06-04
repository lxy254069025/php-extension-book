# zval结构体

php中的任何变量都存放在zval结构体中。包含变量类型和值，php会根据不同的类型标记来取出对应的值。

```c
struct _zval_struct {
	zend_value        value;			/* 值 */
	union {
		struct {
			ZEND_ENDIAN_LOHI_4(
				zend_uchar    type,			/* 值类型 */
				zend_uchar    type_flags,
				zend_uchar    const_flags,
				zend_uchar    reserved)	    /* call info for EX(This) */
		} v;
		uint32_t type_info;
	} u1;
	union {
		uint32_t     var_flags;
		uint32_t     next;                 /* hash collision chain */
		uint32_t     cache_slot;           /* literal cache slot */
		uint32_t     lineno;               /* line number (for ast nodes) */
		uint32_t     num_args;             /* arguments number for EX(This) */
		uint32_t     fe_pos;               /* foreach position */
		uint32_t     fe_iter_idx;          /* foreach iterator index */
	} u2;
};
```

value值取放在zend_value结构中

```c
typedef union _zend_value {
	zend_long         lval;				/* long value */
	double            dval;				/* double value */
	zend_refcounted  *counted;
	zend_string      *str;
	zend_array       *arr;
	zend_object      *obj;
	zend_resource    *res;
	zend_reference   *ref;
	zend_ast_ref     *ast;
	zval             *zv;
	void             *ptr;
	zend_class_entry *ce;
	zend_function    *func;
	struct {
		uint32_t w1;
		uint32_t w2;
	} ww;
} zend_value;
```

这个结构体定义在zend_types.h头文件中。



### zval结构体的应用

首先创建一个php扩展函数，在test.c源文件中加入如下代码：

```c
PHP_FUNCTION(hello) {
    zval str;
    ZVAL_STRING(&str,"HELLO");
    RETURN_ZVAL(&str,0,1);
}
```

然后将函数名添加到test_functions数组中。完整代码如下：

```c
const zend_function_entry test_functions[] = {
	PHP_FE(confirm_test_compiled,	NULL)		/* For testing, remove later. */
    PHP_FE(hello,NULL)
	PHP_FE_END	/* Must be the last line in test_functions[] */
};
```

记得执行make &&  make install,以后的每一次代码的修改都记得执行这条命令。不然会报出找不到该函数的错误。`php -d extension='test.so' -r "echo hello();"`是不是输出HELLO字符串？

为了实现这个功能，我们创建一个名为str的zval变量，记得不要是指针，因为php7为提升性能，减少程序执行时所发出指令，将原来的二级指针改为一级指针，而原来一级指针则直接不需要。

ZVAL_STRING宏函数第一参数是zval的引用，第二个参数是字符串。

RETURN_ZVAL宏函数反回给php。第一个参数zval引用，第二个参数是否需要拷贝，第三个参数是否释放。



### 数组

php数组是通过HashTable实现的而具体的值则存在Bucket中

```c
typedef struct _Bucket {
	zval              val;				/* 值 */
	zend_ulong        h;                /* 数字key  */
	zend_string      *key;              /* 字符串key */
} Bucket;
```

我们可以通过:

```c
PHP_FUNCTION(hello) {
    zval arr;
    array_init(&arr);
    add_assoc_long(&arr,"key",1);
    add_index_long(&arr,1,2);
    
    RETURN_ZVAL(&arr,0,1);
}
```

array_init用于初始化数组，add_assoc_long，add_index_long则对该数组进行赋值，带assoc的函数为字符串作为key，index则是数字作为key。

详细数组操作函数如下：

```c
#define add_assoc_long(__arg, __key, __n)	/* 添加字符串为key的长整型值 */
#define add_assoc_null(__arg, __key) 	/* 添加字符串为key的为null值 */
#define add_assoc_bool(__arg, __key, __b)	/* 添加字符串为key的布尔值 */
#define add_assoc_resource(__arg, __key, __r) 	/* 添加字符串为key的资源类型值 */
#define add_assoc_double(__arg, __key, __d) 	/* 添加字符串为key的双精度值 */
#define add_assoc_str(__arg, __key, __str) 	/* 添加字符串为key的zend_string值 */
#define add_assoc_string(__arg, __key, __str) 	/* 添加字符串为key的char*值 */
#define add_assoc_stringl(__arg, __key, __str, __length)	/* 添加字符串为key的char* 带长度，你的字符串长度为10,但你只取前5个字符 */
#define add_assoc_zval(__arg, __key, __value) 	/* 添加字符串为key的zval值 */

ZEND_API int add_index_long(zval *arg, zend_ulong idx, zend_long n);	/* 添加数字为key的长整型值 */
ZEND_API int add_index_null(zval *arg, zend_ulong idx);	/*  */
ZEND_API int add_index_bool(zval *arg, zend_ulong idx, int b);	/*  */
ZEND_API int add_index_resource(zval *arg, zend_ulong idx, zend_resource *r);	/*  */
ZEND_API int add_index_double(zval *arg, zend_ulong idx, double d);	/*  */
ZEND_API int add_index_str(zval *arg, zend_ulong idx, zend_string *str);	/*  */
ZEND_API int add_index_string(zval *arg, zend_ulong idx, const char *str);	/*  */
ZEND_API int add_index_stringl(zval *arg, zend_ulong idx, const char *str, size_t length);	/*  */
ZEND_API int add_index_zval(zval *arg, zend_ulong index, zval *value);	/*  */

ZEND_API int add_next_index_long(zval *arg, zend_long n);	/* 自动添加数字为key的值 */
ZEND_API int add_next_index_null(zval *arg);	/*  */
ZEND_API int add_next_index_bool(zval *arg, int b);	/*  */
ZEND_API int add_next_index_resource(zval *arg, zend_resource *r);	/*  */
ZEND_API int add_next_index_double(zval *arg, double d);	/*  */
ZEND_API int add_next_index_str(zval *arg, zend_string *str);	/*  */
ZEND_API int add_next_index_string(zval *arg, const char *str);	/*  */
ZEND_API int add_next_index_stringl(zval *arg, const char *str, size_t length);	/*  */
ZEND_API int add_next_index_zval(zval *arg, zval *value);	/*  */
```



