# 创建扩展

php源码文件夹的ext目录有一个ext_skel的sh脚本，我们执行`./ext_skel --extname=test`命令。该命令会在当前文件夹下生一个名为test目录。`--extname=test`表示你的扩展的名称为test。有兴趣的朋友可阅读一下该文件源码。

打开目录后你会发现如下几个文件：

├── CREDITS //扩展描述文件，包含扩展名，开发者信息。默认生成时只带有扩展名
├── EXPERIMENTAL
├── config.m4 //UNIX 构建系统配置
├── config.w32 //Windows 构建系统配置 
├── php_test.h //包含附加的宏、原型和全局量
├── test.c //扩展源文件
├── test.php //测试文件
└── tests  //测试脚本目录

​     └── 001.phpt //测试脚本。测试方法：php ../../run-tests.php ./tests/001.phpt。run-tests.php文件存在php源码根目录

首先打开一个名为config.m4文件。找到如下几行代码。且将期dnl删除。

` PHP_ARG_ENABLE(test, whether to enable test support,`
`Make sure that the comment is aligned:`
`[  --enable-test           Enable test support]) `

PHP_ARG_ENABLE函数第一参数表示扩展名，第二个和第三个在生成configure时的一些提示信息。`--enable-test`表示不依赖第三方库，而`--with-extname`表示需要第三方库，但有没有基本不影响编译。

`PHP_NEW_EXTENSION(test, test.c, $ext_shared,, -DZEND_ENABLE_STATIC_TSRMLS_CACHE=1)`函数声明了这个扩展的名称、需要的源文件名、此扩展的编译形式。

如果是多个文件的话，就在第二个参数后加空格再填写你的源文件地址，需要换行的话记得反斜杠＼。



### 编译

第一步执行phpize，如果没有配置phpize的话，php安装目录下的bin目录加入到系统环境变量中。

```shell
$ phpize
Configuring for:
PHP Api Version:         20151012
Zend Module Api No:      20151012
Zend Extension Api No:   320151012
```

phpize会根据config.m4生一个configure文件，执行./configure后会生makefiles文件，再执行make && sudo make install命令。然后切换目录至cd modules下，你会发现一个test.so文件,并且该动态库会拷贝到你的php扩展目录下。

接下来执行：`php -d extension='test.so' -r "echo confirm_test_compiled('123');"`是不是会输出`Congratulations! You have successfully modified ext/test/config.m4. Module 123 is now compiled into PHP.`信息？那么confirm_test_compiled函数是被定义到哪了呢。接下来我们一探究竟。

还记得我们通过ext_skel生成test.c文件吗？我们的函数就定义在这中间。

```c
#ifdef HAVE_CONFIG_H
#include "config.h"
#endif

#include "php.h"
#include "php_ini.h"
#include "ext/standard/info.h"
#include "php_test.h"

static int le_test;

PHP_FUNCTION(confirm_test_compiled)
{
	char *arg = NULL;
	size_t arg_len, len;
	zend_string *strg;

	if (zend_parse_parameters(ZEND_NUM_ARGS(), "s", &arg, &arg_len) == FAILURE) {
		return;
	}

	strg = strpprintf(0, "Congratulations! You have successfully modified ext/%.78s/config.m4. Module %.78s is now compiled into PHP.", "test", arg);

	RETURN_STR(strg);
}

PHP_MINIT_FUNCTION(test)
{
	return SUCCESS;
}

PHP_MSHUTDOWN_FUNCTION(test)
{
	return SUCCESS;
}

PHP_RINIT_FUNCTION(test)
{
#if defined(COMPILE_DL_TEST) && defined(ZTS)
	ZEND_TSRMLS_CACHE_UPDATE();
#endif
	return SUCCESS;
}

PHP_RSHUTDOWN_FUNCTION(test)
{
	return SUCCESS;
}

PHP_MINFO_FUNCTION(test)
{
	php_info_print_table_start();
	php_info_print_table_header(2, "test support", "enabled");
	php_info_print_table_end();
}

const zend_function_entry test_functions[] = {
	PHP_FE(confirm_test_compiled,	NULL)		/* For testing, remove later. */
	PHP_FE_END	/* Must be the last line in test_functions[] */
};

zend_module_entry test_module_entry = {
	STANDARD_MODULE_HEADER,
	"test",
	test_functions,
	PHP_MINIT(test),
	PHP_MSHUTDOWN(test),
	PHP_RINIT(test),		/* Replace with NULL if there's nothing to do at request start */
	PHP_RSHUTDOWN(test),	/* Replace with NULL if there's nothing to do at request end */
	PHP_MINFO(test),
	PHP_TEST_VERSION,
	STANDARD_MODULE_PROPERTIES
};
/* }}} */

#ifdef COMPILE_DL_TEST
#ifdef ZTS
ZEND_TSRMLS_CACHE_DEFINE()
#endif
ZEND_GET_MODULE(test)
#endif
```

原来php内核是通过PHP_FUNCTION宏来定义php函数的。相当于php代码：

```php
function confirm_test_compiled(string $str) :string {
  return "Congratulations! You have successfully modified ext/test/config.m4. Module {$str} is now compiled into PHP.";
}
```

通过const zend_function_entry test_functions[]={}来生名函数。PHP_FE宏函数需要两个参数，第一个表示函数名，第二则参数校验。在下面的章节中我会仔细介绍如何使用他。PHP_FE_END则表示生名结束。

最后我们看看这段代码：

```c
zend_module_entry test_module_entry = {
	STANDARD_MODULE_HEADER,
	"test",
	test_functions,//在这里，在这里，在这里，在这里，在这里。
	PHP_MINIT(test),
	PHP_MSHUTDOWN(test),
	PHP_RINIT(test),		/* Replace with NULL if there's nothing to do at request start */
	PHP_RSHUTDOWN(test),	/* Replace with NULL if there's nothing to do at request end */
	PHP_MINFO(test),
	PHP_TEST_VERSION,
	STANDARD_MODULE_PROPERTIES
};
```

看到我的中文注释了吗？

扩展的启动和终止建议看看这篇文章[http://www.walu.cc/phpbook/1.2.md](http://www.walu.cc/phpbook/1.2.md)。