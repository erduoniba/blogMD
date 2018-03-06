### ACK分享

![](http://www.geekfan.net/wp-content/uploads/499741e0670668777f7239bfa0b0e21e.jpg) 

相关资料：

[ack官网](http://beyondgrep.com/) 

[the5fire的技术博客  linux下的高效代码搜索工具-ack](https://www.the5fire.com/about-ack-grep-in-linux.html)

[Linux下比grep更快速的检索工具ack-grep安装](http://blog.csdn.net/xtx1990/article/details/9047203)

[如何在Linux上提高文本的搜索效率](http://www.geekfan.net/6881)



#### 1、简单介绍：

i. 它是高效代码搜索工具;

ii.它和 [`grep(强大的文本搜索工具)`](https://www.gnu.org/savannah-checkouts/gnu/grep/manual/grep.html) 很像, 但是对于开发者来说做了进一步的优化, 目的就是要取代 `grep` ;

iii.它纯粹的写在 [`Perl 5(脚本语言)`](http://www.perlchina.org/) 中,利用Perl的正则表达式的力量。

#### 2、特点：

```
速度非常快,因为它只搜索有意义的东西。
更友好的搜索，忽略那些不是你源码的东西。
为源代码搜索而设计，用更少的击键完成任务。
非常轻便，移植性好。
免费且开源
```

#### 3、安装：

if ( `brew` ) {

```shell
$ brew install ack
```

} else {

```shell
$ sudo apt-get install ack-grep   		 #安装(ack已经被其它软件占用了，所以使用ack-grep)
$ sudo mv /usr/bin/ack-grep /usr/bin/ack #更换名称
```

}

#### 4、参数说明：

```shell
-i          忽略大小写
-v        	显示不匹配行
-w        	强制匹配整个单词
-l          只显文件名
-h          不显示名称
-m          在每个文件中最多匹配多少行就停止搜索
-c          显示匹配的总行数
```

#### 5、使用：

**i.** 在当前目录递归搜索单词 **nh_photo_choice** , 不匹配类似于 **nh_photo_choiced** 或 **nh_photo_choices** 的字符串:

```shell
$ ack -w -c -l nh_photo_choice #搜索强制匹配nh_photo_choice,并且只显示文件名和匹配的总行数
```

在项目中搜索结果：

```shell
denglibingdeMacBook-Pro:NewHouseSDK denglibing$ ack -w -l -c nh_photo_choice
NewHouseSDK/NewHouseSDK/image/pdf.xcassets/6.6/nh_photo_choice.imageset/Contents.json:1
NewHouseSDK/ViewController/NHHouseDetailViewController1.m:1
NewHouseSDK/ViewController/PhotoVC/NhPhotoViewController.m:1
```

**ii.** 当前目录下搜索 **image** 目录外的 **nh_photo_choice** ：

```shell
denglibingdeMacBook-Pro:NewHouseSDK denglibing$ ack nh_photo_choice --ignore-dir=image
ViewController/NHHouseDetailViewController1.m
532:            [_choicePhotoBtn setImage:[ResourceManager imageNamed:@"nh_photo_choice"] forState:UIControlStateNormal];

ViewController/PhotoVC/NhPhotoViewController.m
107:        [_choicePhotoBtn setImage:[ResourceManager imageNamed:@"nh_photo_choice"] forState:UIControlStateNormal];
```

**iii.** 对制定类型的文件才进行搜索 **nh_photo_choice**：

```shell
denglibingdeMacBook-Pro:NewHouseSDK denglibing$ ack --objc nh_photo_choice
ViewController/NHHouseDetailViewController1.m
532:            [_choicePhotoBtn setImage:[ResourceManager imageNamed:@"nh_photo_choice"] forState:UIControlStateNormal];

ViewController/PhotoVC/NhPhotoViewController.m
107:        [_choicePhotoBtn setImage:[ResourceManager imageNamed:@"nh_photo_choice"] forState:UIControlStateNormal];

denglibingdeMacBook-Pro:NewHouseSDK denglibing$ ack --json nh_photo_choice
NewHouseSDK/image/pdf.xcassets/6.6/nh_photo_choice.imageset/Contents.json
5:      "filename" : "nh_photo_choice.pdf"
```

你可能会对 `objc` 和 `json` 所对应的文件类型感到困惑，使用如下命令可以得到对应关系：

```shell
denglibingdeMacBook-Pro:NewHouseSDK denglibing$ ack --help-type
Usage: ack [OPTION]... PATTERN [FILES OR DIRECTORIES]

The following is the list of filetypes supported by ack.  You can
specify a file type with the --type=TYPE format, or the --TYPE
format.  For example, both --type=perl and --perl work.

Note that some extensions may appear in multiple types.  For example,
.pod files are both Perl and Parrot.

    --[no]actionscript .as .mxml
    --[no]ada          .ada .adb .ads
    --[no]asm          .asm .s
    --[no]asp          .asp
    --[no]aspx         .master .ascx .asmx .aspx .svc
    --[no]batch        .bat .cmd
    --[no]cc           .c .h .xs
    --[no]cfmx         .cfc .cfm .cfml
    --[no]clojure      .clj
    --[no]cmake        CMakeLists.txt; .cmake
    --[no]coffeescript .coffee
    --[no]cpp          .cpp .cc .cxx .m .hpp .hh .h .hxx
    --[no]csharp       .cs
    --[no]css          .css
    --[no]dart         .dart
    --[no]delphi       .pas .int .dfm .nfm .dof .dpk .dproj .groupproj .bdsgroup .bdsproj
    --[no]elisp        .el
    --[no]elixir       .ex .exs
    --[no]erlang       .erl .hrl
    --[no]fortran      .f .f77 .f90 .f95 .f03 .for .ftn .fpp
    --[no]go           .go
    --[no]groovy       .groovy .gtmpl .gpp .grunit .gradle
    --[no]haskell      .hs .lhs
    --[no]hh           .h
    --[no]html         .htm .html
    --[no]jade         .jade
    --[no]java         .java .properties
    --[no]js           .js
    --[no]json         .json
    --[no]jsp          .jsp .jspx .jhtm .jhtml
    --[no]less         .less
    --[no]lisp         .lisp .lsp
    --[no]lua          .lua; first line matches /^#!.*\blua(jit)?/
    --[no]make         .mk; .mak; makefile; Makefile; Makefile.Debug; Makefile.Release
    --[no]matlab       .m
    --[no]objc         .m .h
    --[no]objcpp       .mm .h
    --[no]ocaml        .ml .mli
    --[no]parrot       .pir .pasm .pmc .ops .pod .pg .tg
    --[no]perl         .pl .pm .pod .t .psgi; first line matches /^#!.*\bperl/
    --[no]perltest     .t
    --[no]php          .php .phpt .php3 .php4 .php5 .phtml; first line matches /^#!.*\bphp/
    --[no]plone        .pt .cpt .metadata .cpy .py
    --[no]python       .py; first line matches /^#!.*\bpython/
    --[no]rake         Rakefile
    --[no]rr           .R
    --[no]rst          .rst
    --[no]ruby         .rb .rhtml .rjs .rxml .erb .rake .spec; Rakefile; first line matches /^#!.*\bruby/
    --[no]rust         .rs
    --[no]sass         .sass .scss
    --[no]scala        .scala
    --[no]scheme       .scm .ss
    --[no]shell        .sh .bash .csh .tcsh .ksh .zsh .fish; first line matches /^#!.*\b(?:ba|t?c|k|z|fi)?sh\b/
    --[no]smalltalk    .st
    --[no]smarty       .tpl
    --[no]sql          .sql .ctl
    --[no]stylus       .styl
    --[no]tcl          .tcl .itcl .itk
    --[no]tex          .tex .cls .sty
    --[no]tt           .tt .tt2 .ttml
    --[no]vb           .bas .cls .frm .ctl .vb .resx
    --[no]verilog      .v .vh .sv
    --[no]vhdl         .vhd .vhdl
    --[no]vim          .vim
    --[no]xml          .xml .dtd .xsl .xslt .ent; first line matches /<[?]xml/
    --[no]yaml         .yaml .yml
```
