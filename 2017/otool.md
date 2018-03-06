## otool

[iOS应用所依赖的系统库检查](http://www.molotang.com/articles/1535.html)

`otool(object file displaying tool)` : 针对目标文件的展示工具，用来发现应用中使用到了哪些系统库，调用了其中哪些方法，使用了库中哪些对象及属性，它是Xcode自带的常用工具。下面是一些常用的命令：

```shell
-f print the fat headers
-a print the archive header
-h print the mach header
-l print the load commands
-L print shared libraries used
-D print shared library id name
-t print the text section (disassemble with -v)
-p <routine name>  start dissassemble from routine name
-s <segname> <sectname> print contents of section
-d print the data section
-o print the Objective-C segment
-r print the relocation entries
-S print the table of contents of a library
-T print the table of contents of a dynamic shared library
-M print the module table of a dynamic shared library
-R print the reference table of a dynamic shared library
-I print the indirect symbol table
-H print the two-level hints table
-G print the data in code table
-v print verbosely (symbolically) when possible
-V print disassembled operands symbolically
-c print argument strings of a core file
-X print no leading addresses or headers
-m don't use archive(member) syntax
-B force Thumb disassembly (ARM objects only)
-q use llvm's disassembler (the default)
-Q use otool(1)'s disassembler
-mcpu=arg use `arg' as the cpu for disassembly
-j print opcode bytes
-P print the info plist section as strings
-C print linker optimization hints
--version print the version of /Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/otool
```



**1、依赖的库查询：**

在iOS开发中，通常我们Xcode工程的一个最终的Product是app格式，而如果你在Finder中点击“显示包内容”看看app里都有什么，你会看到里面的好多东西，有nib、plist、mobileprovision和各种图片资源等，而其中最重要的一个文件就是被标为“Unix可执行文件”(用file查看是Mach-O executable arm)类型的文件。我们的代码逻辑都在这里面。

比如我们需要分析 **钉钉** 这个应用所用到的一些系统库、支持的架构信息及版本号：(部分)

```shell
otool -L DingTalk
DingTalk (architecture armv7):
	/usr/lib/libc++.1.dylib (compatibility version 1.0.0, current version 307.4.0)
	/usr/lib/libiconv.2.dylib (compatibility version 7.0.0, current version 7.0.0)
	/usr/lib/libicucore.A.dylib (compatibility version 1.0.0, current version 57.1.0)
	/usr/lib/libresolv.9.dylib (compatibility version 1.0.0, current version 1.0.0)
	/usr/lib/libstdc++.6.dylib (compatibility version 7.0.0, current version 104.2.0)
	/usr/lib/libxml2.2.dylib (compatibility version 10.0.0, current version 10.9.0)
	/usr/lib/libz.1.dylib (compatibility version 1.0.0, current version 1.2.8)
	/System/Library/Frameworks/AVFoundation.framework/AVFoundation (compatibility version 1.0.0, current version 2.0.0)
	/System/Library/Frameworks/Accelerate.framework/Accelerate (compatibility version 1.0.0, current version 4.0.0)
	...
	...
	...
	
DingTalk (architecture arm64):
	/usr/lib/libc++.1.dylib (compatibility version 1.0.0, current version 307.4.0)
	/usr/lib/libiconv.2.dylib (compatibility version 7.0.0, current version 7.0.0)
	/usr/lib/libicucore.A.dylib (compatibility version 1.0.0, current version 57.1.0)
	/usr/lib/libresolv.9.dylib (compatibility version 1.0.0, current version 1.0.0)
	/usr/lib/libstdc++.6.dylib (compatibility version 7.0.0, current version 104.2.0)
	/usr/lib/libxml2.2.dylib (compatibility version 10.0.0, current version 10.9.0)
	/usr/lib/libz.1.dylib (compatibility version 1.0.0, current version 1.2.8)
	/System/Library/Frameworks/AVFoundation.framework/AVFoundation (compatibility version 1.0.0, current version 2.0.0)
	/System/Library/Frameworks/Accelerate.framework/Accelerate (compatibility version 1.0.0, current version 4.0.0)
	/System/Library/Frameworks/AddressBook.framework/AddressBook (compatibility version 1.0.0, current version 30.0.0)
	...
	...
	...
```



**2、汇编码**

其实，查看依赖系统库仅仅是其中的一小功能。otool命令配上不同的参数可以发挥很强大的功用。如果使用 `otool -tV ex` 则整个ARM的汇编码就都显示出来了。能看到ARM的汇编码，那接下来怎么用就看大家的了，想象空间无限啊:

```shell
otool -tV DingTalk
DingTalk (architecture armv7):
(__TEXT,__text) section
__ZN12mobilesearch12SearcherImpl13sqlite_hookerEPviPKcS3_x:
011a85cc	f0 b5 	push	{r4, r5, r6, r7, lr}
011a85ce	03 af 	add	r7, sp, #0xc
011a85d0	2d e9 00 0d 	push.w	{r8, r10, r11}
011a85d4	ad f1 40 04 	sub.w	r4, sp, #0x40
011a85d8	6f f3 03 04 	bfc	r4, #0, #4
011a85dc	a5 46 	mov	sp, r4
011a85de	04 f9 ed 82 	vst1.64	{d8, d9, d10, d11}, [r4:128]!
011a85e2	04 f9 ef c2 	vst1.64	{d12, d13, d14, d15}, [r4:128]
011a85e6	ad f5 24 5d 	sub.w	sp, sp, #0x2900
011a85ea	88 b0 	sub	sp, #0x20
011a85ec	0a 93 	str	r3, [sp, #0x28]
011a85ee	04 46 	mov	r4, r0
011a85f0	4f f6 06 50 	movw	r0, #0xfd06
011a85f4	0b aa 	add	r2, sp, #0x2c
011a85f6	c0 f2 4b 10 	movt	r0, #0x14b
011a85fa	4f f6 00 23 	movw	r3, #0xfa00
011a85fe	c0 f2 4b 13 	movt	r3, #0x14b
011a8602	78 44 	add	r0, pc
011a8604	7b 44 	add	r3, pc
011a8606	0e 46 	mov	r6, r1
011a8608	00 68 	ldr	r0, [r0]
011a860a	1b 68 	ldr	r3, [r3]
011a860c	df f8 30 15 	ldr.w	r1, [pc, #0x530]
011a8610	1b 68 	ldr	r3, [r3]
011a8612	79 44 	add	r1, pc
011a8614	13 60 	str	r3, [r2]
...
...
...
```



**3、查看该应用是否砸壳**

```shell
otool -l DingTalk | grep -B 2 crypt
          cmd LC_ENCRYPTION_INFO
      cmdsize 20
     cryptoff 16384
    cryptsize 40239104
      cryptid 0
--
          cmd LC_ENCRYPTION_INFO_64
      cmdsize 24
     cryptoff 16384
    cryptsize 45400064
      cryptid 0
```

cryptid 0（砸壳） 1（未砸壳）



**4、 Mach-O头结构等**

其实otool文档本身的介绍就是：otool是对目标文件或者库文件的特定部分进行展示。这里举个例子，看一下其头部的内容是怎样的：

```shell
otool -h DingTalk
Mach header
      magic cputype cpusubtype  caps    filetype ncmds sizeofcmds      flags
 0xfeedface      12          9  0x00           2    71       7148 0x00218085
Mach header
      magic cputype cpusubtype  caps    filetype ncmds sizeofcmds      flags
 0xfeedfacf 16777228          0  0x00           2    71       7832 0x00218085
```

至于各字段的含义，可参看 [loader.h](https://github.com/gdbinit/MachOView/blob/master/mach-o/loader.h)

