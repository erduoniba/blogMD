### appium2-基于python调用unittest框架对iOS进行自动化测试

> 紧急上篇的 [appium1-macOS10.12下如何丝滑的使用appium?](http://blog.csdn.net/u012390519/article/details/54023336)  我相信环境问题已经解决完毕，虽然下载demo并且体验了一次完整的自动化流程，但是有太多的疑问在后面等着我们。这里我总结了一下自动化测试代码一些说明，比如关键字及输出结果等等，这是一条漫漫长路，需要耐心细心。



先来几篇优秀的文章开开胃：

[Python单元测试——深入理解unittest](http://blog.csdn.net/xiaosongbk/article/details/52884837)

[Unit testing framework](https://docs.python.org/2.7/library/unittest.html)

[Appium Python API 中文版 By-HZJ](https://testerhome.com/topics/3711)

[Appium 中文 Appium API 文档](https://testerhome.com/topics/3144) 

[XPath 教程](http://www.w3school.com.cn/xpath/) 



1、首先了解一下几个重要的概念：`test fixture` , `test case` , `test suite` ,  `test runner`

![](http://images.cnitblog.com/i/236038/201404/230040440455234.png) 

- 一个TestCase的实例就是一个测试用例。什么是测试用例呢？就是一个完整的测试流程，包括测试前准备环境的搭建(setUp)，执行测试代码(run)，以及测试后环境的还原(tearDown)。元测试(unit test)的本质也就在这里，一个测试用例是一个完整的测试单元，通过运行这个测试单元，可以对某一个问题进行验证。
- 而多个测试用例集合在一起，就是TestSuite，而且TestSuite也可以嵌套TestSuite。
- TestLoader是用来加载TestCase到TestSuite中的，其中有几个loadTestsFrom__()方法，就是从各个地方寻找TestCase，创建它们的实例，然后add到TestSuite中，再返回一个TestSuite实例。
- TextTestRunner是来执行测试用例的，其中的run(test)会执行TestSuite/TestCase中的run(result)方法。
- 测试的结果会保存到TextTestResult实例中，包括运行了多少测试用例，成功了多少，失败了多少等信息。



下面从代码进行说明，具体可以看代码注释：

```python
import unittest
import os
from appium import webdriver
from time import sleep
import time
import HTMLTestRunner #导入HTMLTestRunner.py进行输出结果,后文有介绍说明

# SimpleIOSTests 继承自 unittest.TestCase，该类中只包含了 test_scroll 一个测试用例
class SimpleIOSTests(unittest.TestCase):

    # 重写setUp()方法，设置自动化测试的一些基本参数
    def setUp(self):
        app = os.path.abspath('../../apps/HHH/build/Release-iphonesimulator/HHH.app')
        self.driver = webdriver.Remote(
                               command_executor='http://127.0.0.1:4723/wd/hub',
                               desired_capabilities={
                               'app': app,
                               'platformName': 'iOS',
                               'platformVersion': '10.2',
                               'deviceName': 'iPhone 7',
                               })
	
    # tearDown()中清除在数据库中产生的数据，然后关闭连接。注意tearDown的过程很重要，要为以后的TestCase留下一个干净的环境
    def tearDown(self):
        self.driver.quit()

    # 测试用例，以test开头
    def test_scroll(self):
        
        # 找到 accessibility_id 为 "button" 的控件执行 点击操作
        self.driver.find_element_by_accessibility_id('button').click()
        
        sleep(1)
        
        # 找到 值为 "HHH" 的控件
        textF1 = self.driver.find_element_by_name('HHH')
        
        print("HHHHHHHH1 %s" % (self.driver.contexts))
        print("HHHHHHHH2 %s" % (self.driver.page_source))

        sleep(1)
        
        # 为 textF1 输入框 赋值，只有 UITextFiled和UITextView可以改变值，如果发现能改变UIButton或者UILabel的值，请告知我 😁
        textF1.send_keys("HHHHHHH")

        sleep(1)
        # 消失键盘
        self.driver.hide_keyboard()
        
        try:
            sleep(1)
        except:
            pass


if __name__ == '__main__':
    
    # loadTestsFromTestCase : 从 SimpleIOSTests类中寻找 测试用例
    # TestLoader用来将 测试用例 加入到测试组中
    suite = unittest.TestLoader().loadTestsFromTestCase(SimpleIOSTests)
    
    # TextTestRunner是来执行测试用例的
    unittest.TextTestRunner(verbosity=2).run(suite)
```

除了看上面推荐的一些链接外，也可以在自己电脑中找到一些说明，比如在我自己电脑中 `/Library/Python/2.7/site-packages/selenium-3.0.2-py2.7.egg/selenium/webdriver/remote/webdriver.py` 可以看到一些说明：

```python
def find_element_by_tag_name(self, name):
     """
     Finds an element by tag name.

     :Args:
      - name: The tag name of the element to find.

     :Usage:
         driver.find_element_by_tag_name('foo')
     """
     return self.find_element(by=By.TAG_NAME, value=name)

 def find_elements_by_tag_name(self, name):
     """
     Finds elements by tag name.

     :Args:
      - name: The tag name the use when finding elements.

     :Usage:
         driver.find_elements_by_tag_name('foo')
     """
     return self.find_elements(by=By.TAG_NAME, value=name)

 def find_element_by_class_name(self, name):
     """
     Finds an element by class name.

     :Args:
      - name: The class name of the element to find.

     :Usage:
         driver.find_element_by_class_name('foo')
     """
     return self.find_element(by=By.CLASS_NAME, value=name)
```





2、输出测试结果：

先安装HTMLTestRunner，可以去 [官网下载](http://tungwaiyip.info/software/HTMLTestRunner_0_8_2/HTMLTestRunner.py) （不要直接复制代码然后拷贝生成 `HTMLTestRunner.py` , 会有格式问题，可以选择 `文件-存储为` 下载到本地），当然也可以直接从  [erduoniba/appium_ios_sample_code](https://github.com/erduoniba/appium_ios_sample_code) 下载获得。 然后将  `HTMLTestRunner.py` 拷贝到 `/Library/Python/2.7/site-packages/` 下即可。

将 `if __name__ == '__main__':` 改成如下代码，再次执行 自动化测试

```python
if __name__ == '__main__':
    suite = unittest.TestLoader().loadTestsFromTestCase(SimpleIOSTests)
    
    # 生成此刻的时间
	timestr = time.strftime('%Y%m%d%H%M%S',time.localtime(time.time()))
    
    # 输出文件
	filename = "/Users/denglibing/Desktop/appiumresult" + timestr + ".html"
	print(filename)
	fp = open(filename, 'wb')
	runner = HTMLTestRunner.HTMLTestRunner(
	                                   stream=fp,
	                                   title='testresult',
	                                   description='testreport'
	                                   )
	runner.run(suite)
	fp.close()	
```

成功之后会在 `/Users/denglibing/Desktop/` 生成对应的 `appiumresult.html` ，打开得到测试结果：

![](http://7xqhx8.com1.z0.glb.clouddn.com/appium2.png) 



一些使用过程的小细节：

```html
1:	如果app已经安装了，在不想安装app的情况下， 可通过dos窗口，通过启动appium带上 --no-reset 即可避免执行用例的时候再次安装app
2:	appium默认启动一个应用的session过期时间是60秒到时间会自动停了刚启动的应用
appium --command-timeout 600 时间变成600秒了
```

