---
title: 图片资源base64化
date: 2017-05-22 10:32:28
categories: coding
tags:
  - Base64
---


除ie外，大部分现代浏览器都已经支持原生的基于 Base64 的 encode 和 decode，例如 `window.btoa` 和 `window.atob` 函数。

## data URI

URI 是 (uniform resource identifier) 的缩写，它定义了接受内容的协议以及附带的相关内容，如果附带的相关内容是一个地址，那么此时的 URI 也是一个 URL (uniform resource locator)

data URI 的格式：

```
data:[<mediatype>][;base64],<data>
```

在这种格式中，`data:` 就是 URI 的协议，表明这是一个 data URI。

以下是一个HTML代码片段：

```
<img src="data:image/gif;base64,R0lGODdhMAAwAPAAAAAAAP///ywAAAAAMAAwAAAC8IyPqcvt3wCcDkiLc7C0qwyGHhSWpjQu5yqmCYsapyuvUUlvONmOZtfzgFzByTB10QgxOR0TqBQejhRNzOfkVJ+5YiUqrXF5Y5lKh/DeuNcP5yLWGsEbtLiOSpa/TPg7JpJHxyendzWTBfX0cxOnKPjgBzi4diinWGdkF8kjdfnycQZXZeYGejmJlZeGl9i2icVqaNVailT6F5iJ90m6mvuTS4OK05M0vDk0Q4XUtwvKOzrcd3iq9uisF81M1OIcR7lEewwcLp7tuNNkM3uNna3F2JQFo97Vriy/Xl4/f1cf5VWzXyym7PHhhx4dbgYKAAA7" alt="Larry">
```

### Base64 使用的优缺点

原本页面由 HTML、CSS、和若干图片组成，如果将图片通过 data URI 的形式合并到CSS文件中，页面就可以完全由 HTML、CSS 来组成。

对于前端来说，显而易见好处是能够减少一个图片的 HTTP 请求，而缺点可能就不够显而易见。

样式表会变得很大，从而阻塞关键下载和渲染。通俗地讲，图片文件或字体文件的体积转移到了 HTML 或 CSS中，而后者的体积直接影响渲染，导致用户会长时间注视空白屏幕。HTML 和 CSS 阻塞渲染，图片不会。

#### 使用 Gzip 压缩文本

Gzip把原文本中多次出现的相同字符串记为一个“标记”，所以文本中重复出现的字符串越多，压缩率越高。

HTML 中重复出现大量的 HTML 标签以及类名等，CSS中重复出现大量的属性，JavaScript 中重复的函数调用等（即使经过混淆）。因此 HTML、CSS、JavaScript 的 Gzip 压缩率都是很高的，最高可达到90%。

而图片经过Base64转化后变成的文本是无规律的，所以在Gzip中不能达到较高的压缩率。

#### Base64 缓存

Base64跟CSS混在一起，难以分别进行缓存设置和更新。

平常的项目中，CSS文件的修改频率是较高的，图片其次，而字体文件，几乎是几个月甚至一年以上才修改一次。我们一般会为不同类型的文件设置不同的缓存失效时间，以及在更新某个文件之后单独更新这个文件的时间戳。

混在一起之后，即使我们只是想更新CSS规则里面一个字号，整个几百K的文件就会重新生成。用户不得不在每次小型更新后重新下载整个大文件，这违背了基本的缓存原则。

#### CSSOM 渲染

**Base64跟CSS混在一起，大大增加了浏览器需要解析CSS树的耗时**。其实解析CSS树的过程是很快的，一般在几十微妙到几毫秒之间。如果CSS文件中混入了Base64，那么（因为文件体积的大幅增长）解析时间会增长到十倍以上。

### 适合使用 Base64 的场景

对于仅有一两个小图标的网页，开发者也许没有必要专门生成一个雪碧图。

如果网页还能保证几个月不会更新，那么缓存不可控的问题也不会凸显。

对于一个使用了背景平铺图片的网页，平铺图片无法合并到页面资源雪碧图中，这时使用 data URI 也许是一个合理的选择。

## Base64 的实现原理

下面的步骤为 Base64 编码过程：

1. Base64的编码都是按字符串长度，以每3个8bit的字符为一组，
2. 针对每组，首先获取每个字符的ASCII编码，
3. 将ASCII编码转换成8bit的二进制，得到一组3*8=24bit的字节
4. 然后再将这24bit划分为4个6bit的字节，并在每个6bit的字节前面都填两个高位0，得到4个8bit的字节（**因为 0 - 63 需要6位字节表示**）
5. 将这4个8bit的字节转换成10进制，对照Base64编码表得到对应编码后的字符。

```

Base64 编码表
0	A	16	Q	32	g	48	w
1	B	17	R	33	h	49	x
2	C	18	S	34	i	50	y
3	D	19	T	35	j	51	z
4	E	20	U	36	k	52	0
5	F	21	V	37	l	53	1
6	G	22	W	38	m	54	2
7	H	23	X	39	n	55	3
8	I	24	Y	40	o	56	4
9	J	25	Z	41	p	57	5
10	K	26	a	42	q	58	6
11	L	27	b	43	r	59	7
12	M	28	c	44	s	60	8
13	N	29	d	45	t	61	9
14	O	30	e	46	u	62	+
15	P	31	f	47	v	63	/
```

（注：1. 要求被编码字符是8bit的，所以须在ASCII编码范围内，`\u0000`-`\u00ff`，中文就不行。如果被编码字符长度不是3的倍数的时候，则都用0代替，对应的输出字符为=）

### 编码过程

#### 字符长度为能被3整除时

字符长度为能被3整除时：比如“Tom” ：

```
            T           o           m
ASCII:      84          111         109
8bit字节:   01010100    01101111    01101101
6bit字节:     010101      000110      111101      101101
十进制:     21          6           61          45
对应编码:   V           G           9           t
```

```
btoa(‘Tom’) = VG9t
```

#### 字符串长度不能被3整除时

字符串长度不能被3整除时，比如“Lucy”：

```
            L           u           c           y
ASCII:      76          117         99          121
8bit字节:   01001100    01110101    01100011    01111001      00000000    00000000
6bit字节:     010011      000111      010101      100011      011110  010000  000000  000000
十进制:     19          7           21          35             30      16      (异常) (异常)
对应编码:   T           H           V           j               e       Q       =       =
```

由于 `Lucy` 只有4个字母，所以按3个一组的话，第二组还有两个空位，所以需要用 `0` 来补齐。这里就需要注意，因为是需要补齐而出现的 `0` ，所以转化成十进制的时候就不能按常规用 Base64 编码表来对应，所以不是a， 可以理解成为一种特殊的“异常”，编码应该对应 `=`。

### 实现 Base64

```
/**
 * base64 encoding & decoding
 * for fixing browsers which don't support Base64 | btoa |atob
 */
 
(function (win, undefined) {
 
     var Base64 = function () {
        var base64hash = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/';
 
        // btoa method
        function _btoa (s) {
            if (/([^\u0000-\u00ff])/.test(s)) {
                throw new Error('INVALID_CHARACTER_ERR');
            }
            var i = 0,
                prev,
                ascii,
                mod,
                result = [];
 
            while (i < s.length) {
                ascii = s.charCodeAt(i);
                mod = i % 3;
 
                switch(mod) {
                    // 第一个6位只需要让8位二进制右移两位
                    case 0:
                        result.push(base64hash.charAt(ascii >> 2));
                        break;
                    //第二个6位 = 第一个8位的后两位 + 第二个8位的前4位
                    case 1:
                        result.push(base64hash.charAt((prev & 3) << 4 | (ascii >> 4)));
                        break;
                    //第三个6位 = 第二个8位的后4位 + 第三个8位的前2位
                    //第4个6位 = 第三个8位的后6位
                    case 2:
                        result.push(base64hash.charAt((prev & 0x0f) << 2 | (ascii >> 6)));
                        result.push(base64hash.charAt(ascii & 0x3f));
                        break;
                }
 
                prev = ascii;
                i ++;
            }
 
            // 循环结束后看mod, 为0 证明需补3个6位，第一个为最后一个8位的最后两位后面补4个0。另外两个6位对应的是异常的“=”；
            // mod为1，证明还需补两个6位，一个是最后一个8位的后4位补两个0，另一个对应异常的“=”
            if(mod == 0) {
                result.push(base64hash.charAt((prev & 3) << 4));
                result.push('==');
            } else if (mod == 1) {
                result.push(base64hash.charAt((prev & 0x0f) << 2));
                result.push('=');
            }
 
            return result.join('');
        }
 
        // atob method
        // 逆转encode的思路即可
        function _atob (s) {
            s = s.replace(/\s|=/g, '');
            var cur,
                prev,
                mod,
                i = 0,
                result = [];
 
            while (i < s.length) {
                cur = base64hash.indexOf(s.charAt(i));
                mod = i % 4;
 
                switch (mod) {
                    case 0:
                        //TODO
                        break;
                    case 1:
                        result.push(String.fromCharCode(prev << 2 | cur >> 4));
                        break;
                    case 2:
                        result.push(String.fromCharCode((prev & 0x0f) << 4 | cur >> 2));
                        break;
                    case 3:
                        result.push(String.fromCharCode((prev & 3) << 6 | cur));
                        break;
 
                }
 
                prev = cur;
                i ++;
            }
 
            return result.join('');
        }
 
        return {
            btoa: _btoa,
            atob: _atob,
            encode: _btoa,
            decode: _atob
        };
    }();
 
    if (!win.Base64) { win.Base64 = Base64 }
    if (!win.btoa) { win.btoa = Base64.btoa }
    if (!win.atob) { win.atob = Base64.atob }
 
 })(window)
```

## 参考

* [关于base64编码的原理及实现](http://www.alloyteam.com/2012/02/关于base64编码的原理及实现/)
* [深挖data URI性能瓶颈](https://isux.tencent.com/understand-data-uri-performance.html)

