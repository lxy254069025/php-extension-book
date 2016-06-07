# 定义类

为了不与之前的代码混淆而变得难以阅读，我们先创建test_class.h和test_class.c两个文件。

在test_class.h文件中加入如下代码：

```c
/*test_class.h*/
#ifndef TEST_CLASS_H
#define TEST_CLASS_H

void create_test_class(void);

#endif /* TEST_CLASS_H */
```

在test_class.c文件中加入如下代码：

```c
#ifdef HAVE_CONFIG_H
#include "config.h"
#endif

#include "php.h"
#include "php_ini.h"
#include "ext/standard/info.h"
#include "test_class.h"

zend_class_entry *test_class_ce;

ZEND_METHOD(test_class,__construct) {
    
}

ZEND_METHOD(test_class,hi) {
    char * hi = "你好！";
    RETURN_STRING(hi);
}

const static zend_function_entry test_functions[] = {
    ZEND_ME(test_class,__construct,NULL,ZEND_ACC_PUBLIC|ZEND_ACC_CTOR)
    ZEND_ME(test_class,hi,NULL,ZEND_ACC_PUBLIC)
    ZEND_FE_END
};


void create_test_class(void) {
    zend_class_entry ce;
    INIT_CLASS_ENTRY(ce,"test\\test_class",test_functions);
    test_class_ce = zend_register_internal_class(&ce);
}
```

先别急着下一步，我们先来了解一下我们干了什么。

test_class.h就不用多解释，就声名一个用于创建该类的函数。我们重在test_class.c文件中。

首先定义了一个zend_class_entry结构体，php中类经过解释后也使用该结构体表示。

ZEND_METHOD宏函数有两个参数，第一个是类名，第二个是该类的方法名。然后将定义好的类名和方法名放入到test_functions数组中，ZEND_ME宏函数参数分别为：类名（ZEND_METHOD中定义的），方法名（ZEND_METHOD中定义的），参数校验，访问权限｜特殊标志（ZEND_ACC_CTOR表示这个构造函数）。ZEND_FE_END作为结束标记。

最后再注册该类。INIT_CLASS_ENTRY初始化上面ce。有本个参数，分为是：zend_class_entry，类的名称（"test\\\test_class"表示test是命名空间，test_class表示类名），zend_function_entry结构体。

通过zend_register_internal_class注册，并把结果给test_class_ce，方便操作。当然不写也没问题。



光写这么几行代码还不能正常行动。还需要在自动生成的test.c文件（以后简称入口源文件）中引入test_class.h文件，并且修改成如下代码：

```c
PHP_MINIT_FUNCTION(test)
{
    create_test_class();
	return SUCCESS;
}
```

PHP_MINIT_FUNCTION函数会在php载入so这后执行，接着执行create_test_class函数注册自定义类。

别忘了在config.m4中加入test_class.c文件，并且重新phpize && ./configure 等操作。

`php -d extension='test.so' -r "var_dump((new test\\test_class())->hi());"`是不是会输出“你好！”。如果是，那恭喜你，第一个类创建成功了。