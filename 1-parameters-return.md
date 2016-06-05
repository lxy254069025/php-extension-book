

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

*浮点开*