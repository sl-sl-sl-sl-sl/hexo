---
title: Unicode学习笔记
date: 2016-08-05 01:30:00

categories:
- 字符串

tags: 
- Unicode
- NSString
- unichar
- UTF-32
- UTF-16
- UTF-8

---



# Unicode
	
**Unicode** 是21位编码, 不是16位, 被分为17个平面 [Plane](https://en.wikipedia.org/wiki/Plane_&#41;Unicode&#41;), 每个平面有65536个字符. 0号平面叫做基本多文种平面 (Basic Multilingual Plane, BMP), 涵盖了几乎所有字符, 除了emoji.

### 码点
单一码点: 一般字符都是单一码点

组合码点: 由两个码点组合而成
	
`é(U+00E9) 可是看成 e(U+0065) 和 ´(U+0301) 组成`

### 重复的定义
	
标准等价(canonically equivalent): 不同码点或不同形式的码点, 有相同的外观与意义

`A(U+0041),	A(U+0410)`

相容等价(compatibility equivalence): 相同的字符, 但在不同使用范围有不同的意义	
	
`ff(U+FB00),  f(U+0066) f(U+0066) `

### 正规形式 

在Unicode里, 判断字符串的等价性并不是一个简单概念, 为了鉴定标准等价与相容等价, Unicode定义了正规化(normalization)算法. 正规化一个字符串的意思是：为了能使它与另一个正规化了的字符串进行二进制比较(binary-compare)

` | 合成形式(é): | 分解形式 (e + ´ )
:------------: | :-------------: | :------------:
标准等价 | C | D
` | precomposedStringWithCanonicalMapping | decomposedStringWithCanonicalMapping
相容等价 | KC | KD
` | precomposedStringWithCompatibilityMapping | decomposedStringWithCompatibilityMapping

仅仅是为了比较的话, 先把字符串正规化成分解形式(D)还是合成形式(C)并不重要, 但是C包含两个步骤,先分解再合成, 所有D更快, Unicode联盟推荐用C的方式存储, 这是为了兼容旧的编码系统. 

### 字形变体

Unicode提供一个叫做变体序列(variation sequences)的机制. 一个基准字符加上256个变体选择符
	emoji的样式就是变体序列, 例: 彩色的伞与黑白的伞  ☔️ (U+2614 U+FE0F), ☔︎ (U+2614 U+FE0E)

### UTF-32

UTF-32每个码点上使用整32位, 每一个UTF-32都可以直接表示对应的码点, 但是从未在实际中使用, 因为每个字符占用4字节太浪费控件

##UTF-16 与 代理对 (Surrogate Pairs)
UTF-16是一种长度可变的编码, 基本多文中平面(BMP)中每一个码点都直接与一个码元相映射. 其他平面(plane)都是两个16位码元来编码的, 这合起来表示一个码点的码元就叫代理对.

为了避免使用UTF-16编码的字符串字节序列产生歧义, Unicode限制了U+0800到U+DFFF范围内的编码用于UTF-16, 这个范围外的序列为代理对的一部分. UTF-16下, U+0FFFF是编码最高值

### 字节顺序

在内存里存储的字符串, 大多数实现方式采用自己平台的CPU字节序(endianness), 而在硬盘存储和网络传输中, UTF8 语序在字符串的开头插入一个字节顺序标记(Byte Order Mask, BOM). 字节顺序标记是一个值为U+FEff的码元, 通过检查文件的头两个字节, 解码器就可以识别出其字节顺序. Unicode把高字节顺序(big-endian byte order)定为默认情况.

##UTF-8 (Ken Thompson 和 Rob Pike)
	
UTF-8 使用1~4个字节来编码一个码点. 从0到127的这些码点直接映射成1个字节(与ASCII完全相同), 接下来的1920歌码点映射成2个字节, BMP里剩下的码点需要3个字节. Unicode的其他平面里的码点则需要4个字节. UTF-8是基于8位的码元的, 因此不需要关心字节顺序.
	
UTF-8 成为存储和交流 Unicode 文本方面的最佳编码, 由于其有效率的空间使用(仅就西方语言来讲)，以及不需要操心字节顺序, 它现在也已经是文件格式、网络协议以及 Web API 领域里事实上的标准了.

### 长度
	
基本多稳重平面外的字符:

随着emoji被引入Unicode, 经常会遇到代理对

`[🌍 length] == 2  true`

直接计算字符串在UTF-32编码下所需要的字节数, 再除以4
		
`[🌍 lengthOfBytesUsingEncoding:UTF32StringEncoding]/4 == 1 true`

组合字符序列:
	
`如果字母 é 是以分解形式(e + ´)编码的，算作两个码元`

变体序列:

`他们和分解形式的组合字符序列一样`


### NSString + Unicode
	
NSString 完全建立在Unicode之上, CFString也包含了NSString的底层实现.
 
`typedef unsigned short unichar`

`unichar`为16位无符号整形, 不够用来表示21Unicode字符.

		- (unichar)characterAtIndex: 	
	
	    NSString *string = @"12🌍45";	// 🌍2个码元Unicode
	    NSLog(@"%c", [string characterAtIndex:1]);
    	NSLog(@"%c", [string characterAtIndex:2]);
    	NSLog(@"%c", [string characterAtIndex:3]);
    	NSLog(@"%c", [string characterAtIndex:4]);

    	Log:
		 2
		 <
		 空
		 4

   		NSLog(@"%@", [string substringWithRange:NSMakeRange(1, 1)]);
   		NSLog(@"%@", [string substringWithRange:NSMakeRange(2, 1)]);
   		NSLog(@"%@", [string substringWithRange:NSMakeRange(2, 2)]);
    	NSLog(@"%@", [string substringWithRange:NSMakeRange(4, 1)]);

    	Log:
    	 2
    	 空
    	 🌍
    	 5

   string中, 🌍的21位被截为2个16位处理, 第一个为<, 第二个未识别, 所以在使用BMP以外的Unicode时, 要特别注意


    	NSLog(@"%@", NSStringFromRange([string rangeOfComposedCharacterSequenceAtIndex:1]));
    	NSLog(@"%@", NSStringFromRange([string rangeOfComposedCharacterSequenceAtIndex:2]));
    	NSLog(@"%@", NSStringFromRange([string rangeOfComposedCharacterSequenceAtIndex:3]));
    	NSLog(@"%@", NSStringFromRange([string rangeOfComposedCharacterSequenceAtIndex:4]));
    	
    	Log:
    		{1, 1}
			{2, 2}
			{2, 2}
			{4, 1}

只有在`length == 1`的时候, 才能确认`unichar`是代表单个码元, 使用 `-characterAtIndex:` 方法不会导致问题

遍历String每一次都使用 `rangeOfComposedCharacterSequenceAtIndex:` 来判断字符是否是单个码元很不方便, 在 String的 `enumerateSubstringsInRange:options:usingBlock:` 方法里, 将参数指定为 `NSStringEnumerationByComposedCharacterSequences`, 内部实现了字符是单码元还是双码元的判断,

加上 `NSStringEnumerationLocalized` 参数, 在定义词语间的和句子间的边界可以将用户所在的区域也考虑进去.

苹果把字符串看作是子字符串的集合, 而不是字符的集合.

### 比较
	
字符串不会自己正规化, 所以比较字符串是否相同会得出错误的结果. `isEqual:` 和 `isEqualToString:` 都是一个字节一个字节地比较, 如果希望字符的合成和分解形式相吻合, 需要手动正规化.

	
		NSString *s = @"\u00E9";
		NSString *t = @"e\u0301"; // e + ´
		BOOL isEqual = [s isEqualToString:t];
		NSLog(@"%@ is %@ to %@", s, isEqual ? @"equal" : @"not equal", t);  
		
		Log:
			é is not equal to é

		NSString *sNorm = [s precomposedStringWithCanonicalMapping];  
		NSString *tNorm = [t precomposedStringWithCanonicalMapping];  
		BOOL isEqualNorm = [sNorm isEqualToString:tNorm];  
		NSLog(@"%@ is %@ to %@", sNorm, isEqualNorm ? @"equal" : @"not equal", tNorm); 

		Log:
			é is equal to é


另一个选择是使用 `compare:` 方法 (或 `localizedCompare:`), 这个方法返回一个和它相容等价的字符串

		NSString *s = @"ff"; // ff  
		NSString *t = @"\uFB00"; // ﬀ ligature  
		NSComparisonResult result = [s localizedCompare:t];  
		NSLog(@"%@ is %@ to %@", s, result == NSOrderedSame ? @"equal" : @"not equal", t);  

		Log:
 			ff is equal to ﬀ


 不考虑等价关系, 指示比较字符串 compare:options 指定 NSLiteralSearch 会有更快的速度



### 从文件和网络读取文本
	
当知道字符编码类型时, `-[NSString initWithData:encoding:]` 使用这个方法实例化字符串, 但是这个方法不提供错误信息.
	
虽然文本文件本身不包含编码信息, 但 NSString 会通过查看扩展文件属性 (extented file attributes)或者规律试探(UTF-8文件里不会出现某些特定的二进制序列)猜测来确定文件的编码, 可以使用 `-[NSString initWithContentsOfURL:encoding:error:]` 这个方法，来从编码已知的文件里读取文本。
	
如果不得不猜测文件的编码:

1. 试试这两个方法：`stringWithContentsOfFile:usedEncoding:error:` 或者 `initWithContentsOfFile:usedEncoding:error:` （或者这两个方法参数为 URL 的等价方法）。 这些方法会尝试猜测资源的编码，如果猜测成功，会以引用的形式带回所用的编码。
2. 如果 1 失败了，试着用 UTF-8 读取资源。
3. 如果 2 失败了，试试合适的老的编码。 这里「合适的」取决于具体情况。它可以是默认的 C 语言字符串编码，也可以是 ISO 或者 Windows Latin 1 编码，亦或者是其它的，取决于你的数据来源。
4. 最终，还可以试试 `Application Kit` 里 `NSAttributedString` 类的载入方法（比如：`initWithFileURL:options:documentAttributes:error:`）。这些方法会尝试纯文本文件，然后返回使用的编码。可以用这些方法打开任意的文档。如果你的程序并不是专业处理文本的程序，这些方法也值得考虑。对于 Foundation 级别的工具，或者不是自然语言的文本来说，这些方法可能不太合适。

网络传输文本应该UTF-8编码, 除非有特别的需要只能用其他的编码需要向文件中写入文本, 需要使用 `writeToURL:atomically:encoding:error:`方法, 这个方法会在UTF-16或UTF-32编码文件上自动加上字节顺序标记. 它还会把文件的编码存储在名为 `com.apple.TextEncoding` 的扩展文件属性里


本文主要参考: https://objccn.io/issue-9-1/

其他链接:

http://www.cl.cam.ac.uk/~mgk25/ucs/utf-8-history.txt
	
https://developer.apple.com/library/ios/documentation/Cocoa/Conceptual/Strings/Articles/stringsClusters.html
	
http://www.joelonsoftware.com/articles/Unicode.html
	
https://vimeo.com/86030221
	
http://nsconference.com
	
https://en.wikipedia.org/wiki/Unicode	





