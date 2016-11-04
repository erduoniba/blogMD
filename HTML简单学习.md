### 一、标签

`p` : 段落

`b` : 粗体

`a` : 链接 

``` html
<!-- 当前页面打开 -->
<a href="http://www.baidu.com">百度</a>      
<!-- 另外在新页面打开 -->
<a href="http://www.baidu.com" target="_black">百度</a>
```

`i` : 斜体

`sup` : 上标 

`sub` : 下标

```html
<h3> h<sub>2</sub>o  harry<sup>TM</sup>  </h3>
```

`br` : 换行

`hr` : 插入水平线

`abbr` : 缩写词

```html
<abbr title="harry up">h</abbr>
```

效果：当光标定位在h上时，会展现出 harry up



`ins` : 显示插入到文档的数据

`del` : 显示删除的数据.  (推荐使用del) 

```html
<h3>hahaha <ins>insert</ins> <del>delete</del></h3>       
```

效果： hahaha <u>insert</u> ~~delete~~ 



`ol` 、`li` : 有序列表

```html
<ol>
	<li> one </li>
	<li> two </li>
	<li> three </li>
	<li> four </li>
</ol>
```

效果：

1. one
2. two
3. three
4. four



`ul` 、`li` : 有序列表

```html
<ul>
	<li> one </li>
	<ul>
		<li> two </li>
		<li> three </li>
	</ul>
	<li> four </li>
</ul>
```

效果：

- one
- - two
  - three
- four



本页面跳转到指定位置：

```html
<a href="#tag">goto tag</a>
...
...
...
<!-- 其他地方定义tag -->
<h3 id="tag">tag at bottom</h3>
```



`img` : 图片资源

```html
<!-- align:right/left/top/middle/bottom src:图片的url width:设置宽度 height:设置高度 alt:没有加载图片完全显示的文本 -->
<img align=right border=1 src="http://static.googleadsserving.cn/pagead/imgad?id=CICAgKDLttWnRxCgARjYBDIIiLnVv9lkb_M" width=100 height=200 alt="这个是京东的一个广告图片" title="这个是京东的一个广告图片" /> 
```



`input` ：文本输入框、单选按钮、复选框、文件上传区域、日期控件

```html
<!-- name：字段用于记录输入框的文本字段，type：输入框类型（hidden：隐藏） size:文本字体大小 maxlength：限制长度 -->
<input name="username" type="text" size=14 maxlength=30>
<input name="password" type="password">

<!-- type：radio(单选按钮) value：按钮对应的值 checked：是否选中 -->
<input name="user" type="radio" value="harry" />harry 
<input name="user" type="radio" value="anna" checked="checked" />anna 
<input name="user" type="radio" value="erduo" />erduo 

<!-- type：radio(单选按钮) value：按钮对应的值 checked：是否选中 -->
<input name="user" type="checkbox" value="harry1" />harry1
<input name="user" type="checkbox" value="anna1" checked="checked" />anna1
<input name="user" type="checkbox" value="harry1" />harry1

<!-- type：file(自动创建选择文件按钮) -->
<input type="file" name="图片上传"> 

<!-- type：submit(提交按钮用来将表单发送到服务器) -->
<input type="submit" value="上传">

<!-- type：date(选择日期) -->
<input type="date" name="selectDate" value="2016-11-04">

<!-- type：search(搜索控件) -->
<input type="search" name="search" placeholder="enter keyword" required>
```



`textarea` ：多行文本输入框

```html
<!-- rows:几行 cols：一行是多少个文字 -->
<textarea name="textView" rows=4 cols=40></textarea>
```



`select` ：下拉列表框

```html
<!-- 单选 -->
<select name="device">
	<option value="ipod">iPod</option>
	<option value="iPhone">iPhone</option>
	<option value="iMac">iMac</option>
</select>

<!-- 多选 multiple:多选标示 size：下拉列表框展示的个数 在不同操作系统中，选择多个选项的差异:对于 windows：按住 Ctrl 按钮来选择多个选项, 对于 Mac：按住 command 按钮来选择多个选项  -->
<select name="person"  multiple="multiple" size=3>
	<option value="Children" selected=selected>Children</option>
	<option value="Student" selected=selected>Student</option>
	<option value="Boss">Boss</option>
	<option value="Develop">Develop</option>
</select>
```



`form` ：表单，提交数据

`fieldset` ：组合表单元素

```html
<!-- 以post方式提交数据，action：url地址 required：字段必须填写 -->
<form action="http://www.example.com/info" method="post">
	<fieldset>
		<legend>个人资料</legend>
		<label>email:<br>
			<input type="text" name="email" required="required">
		</label> <br>
		<label>mobile:<br>
			<input type="text" name="mobile">
		</label> <br>
		<input type="submit" value="submit">
	</fieldset>	
</form>
```

效果如下：

![](http://7xqhx8.com1.z0.glb.clouddn.com/html_1.png)  
![](http://7xqhx8.com1.z0.glb.clouddn.com/html_1.png)






