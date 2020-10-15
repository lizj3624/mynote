## Base64, Base32 和 Base16，用通俗的语言深入到内部

以前只知道有 Base64，就用这东西练手，不过运行结果和别人的编码有点不一样，也不知道是为什么。后来水平长了点儿，知道看文档了，就去找了标准文档来看——这东西人家都给写得清清楚楚的，因为标准就是规定这东西是怎么玩儿的。

Base64 定义在 RFC 4648 文档。一看，原来还有 Base32 和 Base16（下文称作 Base 系列编码）。

[RFC 4648: The Base16, Base32, and Base64 Data Encodings](https://link.zhihu.com/?target=https%3A//www.rfc-editor.org/info/rfc4648)

## Base 系列的作用

先说下这些编码的作用。我们知道，有的字符在一些环境中是不能显示或使用的，比如 `&`, `=` 等字符在 URL 被保留为特殊作用的字符；比如描述一张图片，而图片中的二进制码如果转成对应的字符的话，会有很多不可见字符和控制符（如换行、回车之类），这时就需要对进行编码。 Base 系列的就是用来将字节编码为 ASCII 中的可见字符的。

![img](https://pic2.zhimg.com/80/v2-b2ddcb837ab540024181b2445f80c8d1_1440w.jpg)

上图是一张 .png 格式的图片，右边的乱字符是没法作为 src 的属性的，但将其转为 Base64 后，就没问题了：

<img src="data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAqAAAAE3CAIAAAD61WT0AACTL0lEQVR42ux9B4Ab1bX2qPfeVyttX3vX62.....>

所以说，Base 系列是用来编码的，不是用来加密的，加密怎么的也得有个密码吧。当然，你要将编码序列中的字符打乱那就另说了，但那你也得把打乱后的字符序列记住当密码的，你要忘记或破不开的话，也是没法解密的。

## Base 系列的原理

我们知道，计算机中，存储的的二进制码，是一个字节（8位）一组。Base 系列编码，本质上就是将字节切片组合，然后为重组后的数字找一个对应的、可见的 ASCII 字符，这就是 Base 系列编码了。

![img](https://pic2.zhimg.com/80/v2-3d04e987b76f15050acf3be576ce12b5_1440w.jpg)

**Base16** 编码会将字节切为 4 个一组，所以此编码后会用到 ![[公式]](https://www.zhihu.com/equation?tex=2%5E4+%3D+16) 个字符，数据会扩大二倍。这十六个字符分别是 `0-9A-F`：

![img](https://pic2.zhimg.com/80/v2-2ffb86346061b5166575ebcf17028d61_1440w.jpg)

就是说，`00001111` 重组为 `0000 1111`，编码后的字符取第 0 个和 15 个，也就是 `0F`。

类推，**Base32** 编码会将字节切为 5 个一组，每 5 个字节可以重组为 8 个字符。如果不够 5 个字节，那么就在切出的最后一组后边充 0，充够 5 位，然后右边充 `=`，充够 8组。

```text
00000001 00000011 00000111 00001111 00011111
=> 
00000 00100 00001 10000 01110 00011 11000 11111


00000001 00000011 00000111 00001111 00011111 11111111
=> 
00000 00100 00001 10000 01110 00011 11000 11111  // 先切一组
11111 11100 =     =     =     =     =     =
// 最后不足 5 字节的重组后先在右边充 0，编码完成后再右边充 =
```

就是说：编码一段 5 字节，需要切两下，重组出 8 个字符；编码一段 6 字节，第一下重组出 8 个字符，第二下重组后需充两个 0 和六个 =，为 `xxxxx xxx00 ======`，反正需要凑够 8 个字符。

Base32 编码会用到 ![[公式]](https://www.zhihu.com/equation?tex=2%5E5+%3D+32) 个字符，数据量扩大了 8/5（文本一长，最后填充的 0 和 = 的数据量差不多就可忽略不计了）。

Base32 编码字符序列有两种方案，一种是字母方案，另一种是扩展十六进制的字母方案。

The Base 32 Aplphabet

![img](https://pic3.zhimg.com/80/v2-5641da0653fbb518d506fe2f5a55c8fe_1440w.jpg)

The "Extended Hex" Base 32 Alphabet

![img](https://pic1.zhimg.com/80/v2-6a19192d53155329ca588737eca28eec_1440w.jpg)

**Base64** 编码和 Base32 在同小异。是将每 3 个字节重组为 4 组，每组 6 位，会用到 ![[公式]](https://www.zhihu.com/equation?tex=2%5E6+%3D+64) 个字符。填充 0 和 = 的方法和 Base 32 一样。Base64 编码后，数据量扩大了 4/3。

Base64 编码字符序列也有两种方案，一种是字母方案，一种是 URL 和文件名安全的字母方案。

The The Base 64 Alphabet

![img](https://pic1.zhimg.com/80/v2-4fcf8fc38f90e163285c10be42b5c1fc_1440w.jpg)

The "URL and Filename safe" Base 64 Alphabet

![img](https://pic1.zhimg.com/80/v2-caf4d4ab2a9058c812c49df9c6636ad8_1440w.jpg)

有了上边的知识，再加上 UTF-8 的知识，详见另一篇专栏：[Unicode 编码及 UTF-32, UTF-16 和 UTF-8](https://zhuanlan.zhihu.com/p/51202412)，于是我写出了 Base 系列编码字符的 JavaScript 代码：

```js
/**
 * @param str  {String}
 * @param type {String} "base64", "base64safe", "base32", "base32ex" or "base16"
 *                      default is "base64"
 * @author Liulinwj
 * @licence MIT
 */
let baseEncode = (function() {

  const CONFIG = {
    base64: {
      fromCharCount: 3,
      toCharCount: 4,
      chars: "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/",
      pad: "=",
      enc: base64,
    },
    base64safe: {
      fromCharCount: 3,
      toCharCount: 4,
      chars: "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789-_",
      pad: "=",
      enc: base64,
    },
    base32: {
      fromCharCount: 5,
      toCharCount: 6,
      chars: "ABCDEFGHIJKLMNOPQRSTUVWXYZ234567",
      pad: "=",
      enc: base32,
    },
    base32ex: {
      fromCharCount: 5,
      toCharCount: 6,
      chars: "0123456789ABCDEFGHIJKLMNOPQRSTUV",
      pad: "=",
      enc: base32,
    },
    base16: {
      fromCharCount: 1,
      toCharCount: 2,
      chars: "0123456789ABCDEF",
      enc: base16,
    },
  };


  function base64(c1, c2, c3) {
    let d1, d2, d3, d4;
    d1 = d2 = d3 = d4 = -1;
    if (Number.isFinite(c1)) {
      d1 = c1 >>> 2;
      d2 = (c1 & 0x03) << 4;
    }
    if (Number.isFinite(c2)) {
      d2 |= c2 >>> 4;
      d3 = (c2 & 0x0f) << 2;
    }
    if (Number.isFinite(c3)) {
      d3 |= c3 >>> 6;
      d4 = c3 & 0x3f;
    }
    return [d1, d2, d3, d4];
  }


  function base32(c1, c2, c3, c4, c5) {
    let d1, d2, d3, d4, d5, d6, d7, d8;
    d1 = d2 = d3 = d4 = d5 = d6 = d7 = d8 = -1;
    if (Number.isFinite(c1)) {
      d1 = c1 >>> 3;
      d2 = (c1 & 0x07) << 2;
    }
    if (Number.isFinite(c2)) {
      d2 |= c2 >>> 6;
      d3 = (c2 & 0x3f) >>> 1;
      d4 = (c2 & 0x01);
    }
    if (Number.isFinite(c3)) {
      d4 |= c3 >>> 4;
      d5 = (c3 & 0x0f) << 1;
    }
    if (Number.isFinite(c4)) {
      d5 |= c4 >>> 7;
      d6 = c4 & 0x7c;
      d7 = (c4 & 0x03) << 3;
    }
    if (Number.isFinite(c5)) {
      d7 |= (c5 & 0x03) << 3;
      d7 = c5 & 0x1f;
    }
    return [d1, d2, d3, d4, d5, d6, d7, d8];
  }


  function base16(c) {
    return [c >>> 4, c & 0x0f];
  }


  function getBytes(str) {
    let result = [];
    for (let v of str) {
      let cp = v.charCodeAt(0);
      if (cp < 0x80) {
        result.push(cp);
      } else if (cp < 0x800) {
        result.push((cp >>> 6) | 0xC0, cp & 0x3F | 0x80);
      } else if (cp < 0x10000) {
        result.push(
          (cp >>> 12) | 0xE0,
          ((cp & 0xFC0) >>> 6) | 0x80, cp & 0x3F | 0x80,
        );
      } else {
        result.push(
          (cp >>> 18) | 0xF0,
          ((cp & 0x3F000) >>> 12) | 0x80,
          ((cp & 0xFC0) >>> 6) | 0x80, cp & 0x3F | 0x80,
        );
      }
    }
    return result;
  }


  return function(str, type = "base64") {
    if (!str) {
      return "";
    }
    let config = CONFIG[type];
    if (!config) {
      return "";
    }
    let result = "";
    let bytes = getBytes(str);
    let combine = (r, v) => {
      return r += v === -1 ? config.pad : config.chars[v];
    };
    for (let i = 0, len = bytes.length; i < len; i += config.fromCharCount) {
      let sequence = bytes.slice(i, i + config.fromCharCount);
      result += config.enc(...sequence).reduce(combine, "");
    }
    return result;
  };

})();
```

此方法用于编码字符串，字符编码用的是 UTF-8。至用解码的，没写，因为会写编码的了，就会写解码的了。

## Base64 的其他实现

## atob(), btoa()

先说下浏览器内置的 `atob()` 和 `btoa()`。

`atob()` 是解码，`btoa()` 是编码。但这两个方法只支持 ASCII 字符 `0x00` - `0x99` 的编码，如果字符超过范围，就会报错。对于中文，这两个方法是不支持的，编码前需要用 `decodeURIComponent()` 或 `decodeURI()` 处理一下，解码这样编码的字符后也需要用 `encodeURIComponent()` 或 `encodeURI()` 还原一下。

这样设计的原因，我认为是这两个方法只负责编码、解码，而不负责解决字符集编码（Charset Encoding）的问题。在 Java 的实现中也是这样。

## Encoder & Decoder in Java

再说下 Java 中的实现。`java.util.Base64.Encoder` 和 `java.util.Base64.Decoder` 这两个类分别用于编码和解码。

先看 `Encoder`：

```java
public byte[] encode(byte[] src);

public int encode(byte[] src, byte[] dst);

public String encodeToString(byte[] src);

public ByteBuffer encode(ByteBuffer buffer) {}
```

所有用于编码的方法都是编码字节的，没有一个是直接编码字符串的。编码字符串时，需要先调用 `String.getBytes()` 方法将字符串转换为字节。至于何用什么编码，由 `getBytes()` 方法决定。

再看 `Decoder()`，同样没有直接解码成字符串的——解码后要转换成字符串使用什么字符编码也不由这个类决定。

```java
public byte[] decode(byte[] src);

public byte[] decode(String src);

public int decode(byte[] src, byte[] dst);

public ByteBuffer decode(ByteBuffer buffer);
```

有的实现编码和解码的字符不一样，大多都是因为使用的字符集编码或字节顺序编码（BOM）不一样而已。

## 引用

1. [base64-wiki](https://zh.wikipedia.org/zh-hans/Base64)
2. [Base64笔记](http://www.ruanyifeng.com/blog/2008/06/base64.html)

