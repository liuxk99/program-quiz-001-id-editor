# 背景知识
在Android应用或者库(aar)中，如果一个资源需要给其它应用使用，为了保证编译后资源(id)仍然独立，必须给这个资源设置一个public描述。
在该描述中把对应名字的资源的id给固定下来。在客户应用编译后，id为指定的值。这样运行时通过id查找的资源就不会是其它的资源了。

## public元素
### XML
```XML
<public type="attr" name="theme" id="0x01010000" />
```
type代表类型，name代表同一类型中唯一的名字，id代表aapt编译出的32位整型值。在xml中引用该资源的写法如下：
`@attr/theme`，而编译后所对应的id则为`0x01010000`。

### id
资源的id，为一个32位的整型值。
其中第1字节为pkg-index,`0x01`，普通应用内部的资源pkg-index为`0x7f`，
`0x01`代表系统应用(pkg:android)，apk为/system/framework/framework-res.apk，
第2字节为type值，其中`01`对于attr类型，
第3、4字节代表同一个类型资源的顺序号，`0000`表示该类别的第一个资源。

# 相关文件
+ public.xml
+ overlay.xml
+ 3rd.xml
