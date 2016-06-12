# 继承与接口

继承与接口在语言层中是不可缺少的。但在扩展中也提供了相应的方法。我找到了一篇相当棒的一篇文章：[http://www.walu.cc/phpbook/10.4.md](http://www.walu.cc/phpbook/10.4.md)。文章中使用的php5来进行说明，但与php7其实也差不多。接下来如何在php7中实现继承与接口。

### 继承

扩展中继承使用`zend_register_internal_class_ex(zend_class_entry *class_entry, zend_class_entry *parent_ce);`函数，class_entry表示当前的class,parent_ce表示父类。

我们新建parent_test_class.h和parent_test_class.c文件。

代码清单：

parent_test_class.h

```c
#ifndef PARENT_TEST_CLASS_H
#define PARENT_TEST_CLASS_H


void create_parent_test_class(void);//创建父类函数

zend_class_entry *get_parent_test_class();//返回父类zend_class_entry

#endif /* PARENT_TEST_CLASS_H */
```

parent_test_class.c

```c
#ifdef HAVE_CONFIG_H
#include "config.h"
#endif

#include "php.h"
#include "php_ini.h"
#include "ext/standard/info.h"
#include "parent_test_class.h"

zend_class_entry *parent_test_class_ce;

zend_class_entry *get_parent_test_class() {//返回当前zend_class_entry
    return parent_test_class_ce;
}

ZEND_METHOD(parent_test_class,parent_hi) {
    zend_printf("我是父类hi函数！");
}

const static zend_function_entry parent_test_class_functions[] = {
    ZEND_ME(parent_test_class,parent_hi,NULL,ZEND_ACC_PUBLIC)
    ZEND_FE_END
};

void create_parent_test_class(void) {
    zend_class_entry ce;
    INIT_CLASS_ENTRY(ce,"test\\parent_test",parent_test_class_functions);
    parent_test_class_ce = zend_register_internal_class(&ce);
}
```

修改test_class.c文件。在顶部引入parent_test_class.h头文件。把`test_class_ce = zend_register_internal_class(&ce);`修改成`test_class_ce = zend_register_internal_class_ex(&ce,get_parent_test_class());`然后在test.c源文件的｀`PHP_MINIT_FUNCTION(test)`添加`create_parent_test_class()`。并在config.m4加入parent_test_class.c源文件。执行phpize等一系列操作。

运行`php -d extension='test.so' -r "var_dump((new test\\test_class())->parent_hi());"`命令是不是输出“我是父类hi函数！”。



### 接口

个人感觉接口没啥用，项目中基本用不到它。各位还是看上面那篇文章吧。

