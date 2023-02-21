---
title: phpGD库图像处理
tags: []
id: '169'
categories:
  - - 网站建设
date: 2020-03-27 03:41:58
---

# phpGD库操作

## 简介及安装配置

> PHP并不仅限于创建HTML输出，它也可以创建和处理包括GIF，PNG， JPEG， WBMP 以及XPM在内的多种格式的图像。更加方便的是，PHP可以直接将图像数据流输出到浏览器。要想在PHP中使用图像处理功能，这时候就需要使用PHP提供的GD函数库。

### GD库的安装及配置

*   配置PHP配置文件，开启extension=php\_gd2.dll
*   设置extension\_dir="ext目录所在的位置"

重启服务器检测gd扩展是否开启`var_dump(extension_loaded('gd));`

检测函数是否可使用

```
var_dump(function_exists('gd_info'));
```

检查gd库信息

```
var_dump(gd_info());
```

查看所有包含的函数

```
print_r(get_defined_functions());
```

## 创建流程

### 创建画布

使用`imagecreatetruecolor($width,$height)`来创建一个画布  

> 创建一个画布，成功返回图像资源标识符，错误返回FALSE。
> 
> ```php
> $image = imagecreatetruecolor(500,300);
> ```
> 
> 注：`imagecreate`和`imagecreatetruecolor`都会创建一个画布，但不同的是前者创建一个空白画布，而后者则会创建一个黑色画布

使用imagecolorallocate ( resource $image , int $red , int $green , int $blue )为一画布分配颜色  

> ```php
> //创建颜色 $red = imagecolorallocate($image,255,0,0)
> ```

开始绘画，这里尝试使用`imagechar ( resource $image , int $font , int $x , int $y , string $c , int $color )`水平的画出一个字符

> imagechar bool型
> 
> ```php
> imagechar($image,5,50,10,'K',$red);
> ```
> 
> 注：若要画出一个字符串请使用imagestring()

将绘画的结果告诉浏览器以图片的形式显示

> 用imagejpeg输出一个图像到浏览器或文件
> 
> ```php
> header('content-type:image/jpeg');//告诉浏览器以图片的形式显示 imagejpeg($image);//输出图像
> ```

销毁资源

> ```php
> imagedestroy($image);
> ```
> 
> ![图片.png](https://i.loli.net/2020/03/27/kpE5tI9FNRbGTm6.png)
> 
> 输出结果：

## 使用系统字体绘制内容

如果使用以上方法绘制文字的话就会有两个问题

1.  1.  画布黑色
    2.  字体太小

\[toc\]