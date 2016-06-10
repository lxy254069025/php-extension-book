# 类成员属性

定义成员属性使用的是zend_declare_property_*系列函数。前三个参数是分别是：zend_class_entry类指针，名字，名字的长度。最后一个参数则是访问权限。其他参数在下面代码注释

```c
zend_declare_property(zend_class_entry *ce, const char *name, size_t name_length, zval *property, int access_type);	/* 定义任意类型，zval 指针类型 */

zend_declare_property_null(zend_class_entry *ce, const char *name, size_t name_length, int access_type);	/* 定义为空，只有上面说明的四个参数 */

zend_declare_property_bool(zend_class_entry *ce, const char *name, size_t name_length, zend_long value, int access_type);	/* 初始化为布尔类型，0:false，1true */

zend_declare_property_long(zend_class_entry *ce, const char *name, size_t name_length, zend_long value, int access_type);	/* 整型 */

zend_declare_property_double(zend_class_entry *ce, const char *name, size_t name_length, double value, int access_type);	/* 浮点型 */

zend_declare_property_string(zend_class_entry *ce, const char *name, size_t name_length, const char *value, int access_type);	/* 字符串char * */

zend_declare_property_stringl(zend_class_entry *ce, const char *name, size_t name_length, const char *value, size_t value_len, int access_type);	/* 字符串，需要指定长度 */

```

举个栗子

```C
void create_test_class(void) {
    zend_class_entry ce;
    INIT_CLASS_ENTRY(ce,"test\\test_class",test_functions);
    test_class_ce = zend_register_internal_class(&ce);
    
    zend_declare_property_null(test_class_ce, ZEND_STRL("ISNULL"), ZEND_ACC_PUBLIC);	/* 定义为空，只有上面说明的四个参数 */

    zend_declare_property_bool(test_class_ce, ZEND_STRL("ISBOOL"), 1,ZEND_ACC_PUBLIC);	/* 初始化为布尔类型，0:false，1true */

    zend_declare_property_long(test_class_ce, ZEND_STRL("ISLONG"),100, ZEND_ACC_PUBLIC);	/* 整型 */

    zend_declare_property_double(test_class_ce, ZEND_STRL("ISDOUBLE"),1.33, ZEND_ACC_PUBLIC);	/* 浮点型 */

    zend_declare_property_string(test_class_ce, ZEND_STRL("ISSTRING"),"LUXIXI", ZEND_ACC_PUBLIC);	/* 字符串char * */

    zend_declare_property_stringl(test_class_ce, ZEND_STRL("ISSTRINGL"),"LUXIXI",strlen("LUXIXI"), ZEND_ACC_PUBLIC);	/* 字符串，需要指定长度 */
}
```

