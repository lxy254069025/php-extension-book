# 类成员属性

### 普通变量

#### 定义

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

ZEND_STRL(str)宏函数编译后会替换成(str), (sizeof(str)-1)，一个字符串，一个字符串长度。

#### 更新

更新和定义变量基本差不多，使用的是zend_update_property*系列函数，第一个参数与定义时所使用的函数一致，第二个为当前对象的zval＊ ，第三个参数为名字，第四个为名字长度，第五个为值。

栗子

```c
zval str;
ZVAL_STRING(&str,"2b");
zval *self = getThis();//获取当前类对象

zend_string * abc = zend_string_init(ZEND_STRL("abbbb"),1);

zend_update_property(test_class_ce,self, ZEND_STRL("ISNULL"), &str);
zend_update_property_bool(test_class_ce,self, ZEND_STRL("ISBOOL"), 0);
zend_update_property_long(test_class_ce,self, ZEND_STRL("ISLONG"),-100);
zend_update_property_double(test_class_ce, self,ZEND_STRL("ISDOUBLE"),1.33);
zend_update_property_str(test_class_ce, self,ZEND_STRL("ISSTRING"),abc);
zend_update_property_string(test_class_ce, self,ZEND_STRL("ISSTRINGL"),"LUXIXI");
zend_update_property_stringl(test_class_ce, self,ZEND_STRL("ISSTRINGL"),"LUXIXI",strlen("LUXIXI"));
```

#### 读取值

`zend_read_property(zend_class_entry *scope, zval *object, const char *name, size_t name_length, zend_bool silent, zval *rv);`该函数用于读取值。silent为1是表示没有该变量时返回IS_NULL，为0时会报出经警告。

栗子

```c
zval *self = getThis();
zval *value = zend_read_property(test_class_ce,self,ZEND_STRL("ISNULL"),1,NULL);
```



### 静态变量

#### 定义

静态变量和普通变量的定义函数一致，只是最后一个参数加｜ZEND_ACC_STATIC。

```C
zend_declare_property_null(mxx_mxx_ce,ZEND_STRL("app"),ZEND_ACC_PUBLIC|ZEND_ACC_STATIC);
```

#### 更新

静态变量不需要getThis()来获取当前类的zval,类是直接使用zend_class_entry.

```c
zend_update_static_property(mxx_mxx_ce,ZEND_STRL("app"),getThis());//表示把当前的对象存放在app这个静态变量中。
```

#### 读取

```c
zval * app = zend_read_static_property(mxx_mxx_ce,ZEND_STRL("app"),1);
```



### 常量

内核中常量定义之后是不可变的，当然php语言层中也不可变。使用方法和普通变量一样。

```c
zend_declare_class_constant(zend_class_entry *ce, const char *name, size_t name_length, zval *value);

zend_declare_class_constant_null(zend_class_entry *ce, const char *name, size_t name_length);

zend_declare_class_constant_long(zend_class_entry *ce, const char *name, size_t name_length, zend_long value);

zend_declare_class_constant_bool(zend_class_entry *ce, const char *name, size_t name_length, zend_bool value);

zend_declare_class_constant_double(zend_class_entry *ce, const char *name, size_t name_length, double value);

zend_declare_class_constant_stringl(zend_class_entry *ce, const char *name, size_t name_length, const char *value, size_t value_length);

zend_declare_class_constant_string(zend_class_entry *ce, const char *name, size_t name_length, const char *value);
```

