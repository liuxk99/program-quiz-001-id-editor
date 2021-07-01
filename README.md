# 背景知识
在Android应用或者库(aar)中，如果一个资源需要给其它应用使用，为了保证编译后资源(id)仍然独立，必须给这个资源设置一个public描述。
在该描述中把对应名字的资源的id给固定下来。在客户应用编译后，id为指定的值。这样运行时通过id查找的资源就不会是其它的资源了。

## public元素
### XML
```XML
<resources>
    <!--... -->
    <public type="attr" name="theme" id="0x01010000" />
    <public type="attr" name="label" id="0x01010001" />
    <!--... -->
    <public type="attr" name="advancedPrintOptionsActivity" id="0x10103f1"/>
    <!--... -->
</resources>
```
type代表类型，name代表同一类型中唯一的名字，id代表aapt编译出的32位整型值。在xml中引用该资源的写法如下：
`@attr/theme`，而编译后所对应的id则为`0x01010000`。

### id
资源的id，为一个32位的整型值。
其中第1字节为pkg-index,`0x01`，普通应用内部的资源pkg-index为`0x7f`，
`0x01`代表系统应用(pkg:android)，apk为/system/framework/framework-res.apk，
第2字节为type值，其中`01`对于attr类型，对于不同的type，这个字节是不同的，比如。
```XML
<resources>
    <!--... -->
    <public type="id" name="background" id="0x01020000" />
    <!--... -->
</resources>
```
第3、4字节代表同一个类型资源的顺序号，`0000`表示该类别的第一个资源。

## 改动
在管理系统资源上，通常有三个角色AOSP、方案商、厂商。
AOSP来自于Google，高通或者MTK在此基础上加上基带的软硬件，提供手机的Android方案。
然后厂商如小米、OPPO在方案商的基础上继续添加自己的特性，比如摄像头、UI等。
AOSP、方案商、厂商都会引入新的资源，而这些资源将被其它应用来使用，它们就必须有自己的id。

## 相关文件
笔者分别提炼出public.xml，overlay.xml，3rd.xml，分别代表了ASOP、方案商、厂商的改动。
在三个xml文件中，同一type，资源id连续。public.xml中最后一个attr的id值为`0x10103f1`，那么overlay.xml的第一个attr资源的id就为
`0x10103f2`，如下文所示。
```XML
<resources>
    <!--... -->
    <public type="attr" name="scrollbarThumb" id="0x10103f2" />
    <!--... -->
</resources>
```
作为厂商要适配不同的方案，不同的方案商对应的overlay.xml中资源数量、名称有所不同，
那么在管理自身引入public资源时，3rd.xml的public元素中id的值则有所不同。通常可能引入的内容不多，而且AOSP，方案商的内容几个月才会改变，所以手动编辑还凑活能应付。
如果同一个AOSP大版本，2-3个方案商，再加上好些产品的分支，在每个分支上都重新编辑一次，工作量就大了。
估算如下：
```
time(edit)*num(AOSP-version)*num(Solution)*num(branch)
```
可能是几十上百个编辑时间。那么能不能编写一个工具来计算出3rd.xml中public元素的id值呢？

# 问题
经过分析，问题如下：
## 输入
public.xml, overlay.xml从中分析各个type中id的最大值max_id。
## 输出
从3rd.xml中读入public元素，然后根据type设置新的id值，从max_id开始，并且逐个public元素递增。
## 注意
public.xml, overlay.xml中的type的public元素的id值只需要唯一，不需要严格升序，虽然默认这么编辑更易于阅读。笔者对于本题中的overlay.xml文件进行了顺序调整，所以同type的id值不符合升序，这一点请读者注意。

# 运行
程序需要解析xml文件，计算不同type的最大值，解析3rd.xml，并写入public元素新的id，另存xml文件。
## 控制台输出
```
After parse 'data/public.xml'
            anim: 0x010a000c
        animator: 0x010b0001
           array: 0x01070004
            attr: 0x010103f1
           color: 0x0106001b
           dimen: 0x01050006
        drawable: 0x010800b3
              id: 0x0102002d
         integer: 0x010e0003
    interpolator: 0x010c000c
          layout: 0x01090027
          mipmap: 0x010d0000
          string: 0x01040017
           style: 0x010301e4
length: 14
After parse 'data/overlay.xml'
            anim: 0x010a000c
        animator: 0x010b0005
           array: 0x01070004
            attr: 0x010103f5
           color: 0x01060026
           dimen: 0x01050023
        drawable: 0x010800be
              id: 0x01020036
         integer: 0x010e0003
    interpolator: 0x010c000c
          layout: 0x01090030
          mipmap: 0x010d0000
          string: 0x01040017
           style: 0x010301ed
length: 14

Edit 'data/3rd.xml' and save to 'data/out.xml'
        type,                                 name,                         id
        attr,                       windowXXXTitle, (0x010103db -> 0x010103f6)
        attr,                   windowXXXTitleSize, (0x010103dc -> 0x010103f7)
        attr,                  windowXXXTitleStyle, (0x010103dd -> 0x010103f8)
       style,                       Theme.Holo.XXX, (0x010301e1 -> 0x010301ee)
       style,             TextAppearance.Large.XXX, (0x010301e2 -> 0x010301ef)
       style,   Theme.Holo.XXX.Dialog.Alert.Simple, (0x010301ee -> 0x010301f0)
       dimen,              XXX_window_left_padding, (0x01050007 -> 0x01050024)
       dimen,             XXX_window_right_padding, (0x01050008 -> 0x01050025)
       dimen,                  XXX_text_size_large, (0x01050009 -> 0x01050026)
       color,              primary_holo_XXX_normal, (0x0106001c -> 0x01060027)
       color,             primary_holo_XXX_focused, (0x0106001d -> 0x01060028)
       color,             primary_holo_XXX_disable, (0x0106001e -> 0x01060029)
    drawable,                 btn_default_holo_XXX, (0x010800b4 -> 0x010800bf)
    drawable,                    setting_highlight, (0x010800b5 -> 0x010800c0)
    drawable,                       setting_normal, (0x010800b6 -> 0x010800c1)
      layout,                      XXX_list_item_1, (0x01090018 -> 0x01090031)
      layout,                      XXX_list_item_2, (0x01090019 -> 0x01090032)
      layout,               XXX_volume_adjust_item, ( 0x1090021 -> 0x01090033)
      layout,                    XXX_volume_adjust, ( 0x1090022 -> 0x01090034)
      layout,                      XXX_volume_mute, ( 0x1090023 -> 0x01090035)
        attr,                        layoutOffsetX, (0x010103fa -> 0x010103f9)
        attr,                        layoutOffsetY, (0x010103fb -> 0x010103fa)
        attr,                          offsetWidth, (0x010103fc -> 0x010103fb)
        attr,                                editW, (0x01010403 -> 0x010103fc)
        attr,                         pwdlabelText, (0x01010409 -> 0x010103fd)
        attr,                       itemNormalIcon, (0x01010412 -> 0x010103fe)
        attr,                           itemDetail, (0x01010415 -> 0x010103ff)
        attr,                               banner, (0x01010416 -> 0x01010400)
```
## 效果
处理完out.xml应该和result.xml的中id的值一致，不要求格式完全一致。效果如下图：
![out.xml](https://github.com/liuxk99/program-quiz-001-id-editor/blob/main/result.png)

# 校招
参加校招的同学，请联系HR或者面试官获取校招提示包。