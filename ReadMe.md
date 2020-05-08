# 基于Hi_PO框架的页面对象模型设计分析

微信公众号地址：[知识的阿尔法](https://mp.weixin.qq.com/s/FXX0J7iueLTu5YZQkQ9NJw)

   * [基于Hi_PO框架的页面对象模型设计分析](#基于hi_po框架的页面对象模型设计分析)
      * [1. 页面对象模型与框架主体](#1-页面对象模型与框架主体)
      * [2. Hi_PO框架设计](#2-hi_po框架设计)
         * [2.1 Page_Objects](#21-page_objects)
         * [2.2 hi_po_unit](#22-hi_po_unit)
         * [2.3 param](#23-param)
      * [3. 系统页面部分](#3-系统页面部分)
      * [4. 测试主体部分与测试报告](#4-测试主体部分与测试报告)
      * [5. 调用过程](#5-调用过程)
      * [6. 总结](#6-总结)

## 1. 页面对象模型与框架主体
在读《京东质量团队转型实践》中在讲UI自动化测试框架时，讲到了以PageObject设计模式设计的UI自动化测试框架示例Hi_PO（从page-objects项目衍生）。从图书所给的代码地址，在Github上面下载下来Hi_PO框架示例。框架示例剥离的比较简单，今天通过框架示例，来看页面PageObject的自动化框架的设计及相关必要环节。

以下是页面书中对PageObject设计模式的介绍：
> 页面对象模型将测试代码和被测试的页面的页面元素及其操作方法进行分离，以降低页面元素的变化对测试代码的影响。每一个被测试的页面都会被定义为一个类，类中会定位所有需进行测试操作的页面元素对象，并且定义操作每一个页面元素对象的方法。

![](https://github.com/DingTobest/Hi_PO/blob/master/pic/1.png?raw=true)

从项目结构上面，我们可以看到包含几部分，后面我们针对每一部分进行拆解：
- 浏览器Driver，针对Selenium进行Web自动化必须的Driver存放
- Hi_PO框架主体，封装了自动化测试过程中的基础功能实现
- 系统页面，对被测系统页面及元素的定义
- 测试主体部分，配置文件、案例、测试数据的定义部分
- 测试报告，测试结束后，生成的测试报告存储的位置
- 入口函数，main函数所处位置，包含测试整体过程

## 2. Hi_PO框架设计
![](https://github.com/DingTobest/Hi_PO/blob/master/pic/2.png?raw=true)

在框架的主体部分，包含以下5部分功能：
- page_objects模块包含页面对象框架设计的主体部分
- hi_po_unit继承于Python单元测试框架的unittest.TestCase类，实现测试调度部分
- param模块为测试案例提供所使用的测试数据
- report与HTMLTestRunner_cn模块提供测试报告生成
- mail模块提供发送测试报告的功能
- MySQLHelper模块封装了MySQL的基础操作（未使用）


### 2.1 Page_Objects

Page_Objects模块中，包含了页面对象模型设计中的几个相关重要的类。
![](https://github.com/DingTobest/Hi_PO/blob/master/pic/3.png?raw=true)

第一部分是PageObject类，这个类是包含页面的主体定义。包含页面的基本信息，如URL等。同时定义了基础的针对页面的操作，如获取页面标题（getTitle）、切换frame（switchTo）、对话框处理（acceptAlert）。后续的页面都从此页面对象类进行继承。

页面对象类进行定义后，对页面元素类进行定义。

![](https://github.com/DingTobest/Hi_PO/blob/master/pic/4.png?raw=true)

首先，定义的是webdriver中，8中元素定位信息。
其次，是三种不同的页面对象元素类：
- PageElement类，基础的页面元素
- MultiPageElement类，存在多个的页面元素
- GroupPageElement类，类似于下拉框、复选框等以组为单位的页面对象

![](https://github.com/DingTobest/Hi_PO/blob/master/pic/5.png?raw=true)

以PageElement类为例，PageObject定义了基本的页面元素信息，构造函数init可以通过不同的页面元素定位方式进行初始化。并且定义了元素的基础操作。MultiPageElement与roupPageElement类的基础操作也相同。

PageObject类与PageElement类内容也比较简单，主要思想也是对页面与页面元素的操作进行第一层的封装，这层的封装主要是针对技术层面的封装。

### 2.2 hi_po_unit

![](https://github.com/DingTobest/Hi_PO/blob/master/pic/6.png?raw=true)

hi_po_unit继承自Python单元测试框架的的unittest.TestCase。此部分主要定义了自动化测试运行过程控制。

第一部分包含setUp（测试前初始化）与tearDown（测试后处理）两部分。我们看到，在此类中定义了错误收集浏览器的dirver初始化等信息，与在结束后dirver推出，抛出异常等信息。

![](https://github.com/DingTobest/Hi_PO/blob/master/pic/7.png?raw=true)

第二部分是测试案例与数据驱动的管理，通过测试案例类与测试数据，生成测试的suite。在生成时提供了通过测试案例类与测试案例中具体的方法进行管理的方式。

### 2.3 param

![](https://github.com/DingTobest/Hi_PO/blob/master/pic/8.png?raw=true)

参数类当中首先是参数的Param基类，定义了数据模块的基础功能，XLS类继承于Param类，实现Param类中的相关操作，通过Excel提供测试数据。

![](https://github.com/DingTobest/Hi_PO/blob/master/pic/9.png?raw=true)

在最下面是数据工厂类，由于这个示例当中只有Excel获取数据这一种方式，所以工厂只能生产这一种数据。可以通过继承Param类，实现类似于数据库获取、接口获取等不同数据提供的形式。

## 3. 系统页面部分

![](https://github.com/DingTobest/Hi_PO/blob/master/pic/10.png?raw=true)

在系统页面部分，现在有一个SearchPage类，是对搜索页的定义。我们可以看到SearchPage类中继承于PageObject类，实现页面对象的定义。同时，类中包含4个成员，通过XPath定义的页面元素。

此类中使用PageObject类与PageElement类，完成了对页面对象与页面元素的定义。这一部分主要是对页面再次进行一次封装，将页面和元素的定位进行整合。这样在上层使用的时候，并不用关心页面元素定位的问题，只需要实现业务逻辑即可。

可以以SearchPage类的形式，增加所需的页面对象。

## 4. 测试主体部分与测试报告

![](https://github.com/DingTobest/Hi_PO/blob/master/pic/11.png?raw=true)

- config中包含的是与测试过程相关的所有配置信息。
- param中包含的是测试过程中所需要的测试数据。
- test_search.py就是此次使用的测试案例。
- test_report中包含的是测试结束后生成的测试报告。

![](https://github.com/DingTobest/Hi_PO/blob/master/pic/12.png?raw=true)

我们可以看到在数据文件中包含case、exp两个参数，在下面的测试案例中会使用到这两个参数。

![](https://github.com/DingTobest/Hi_PO/blob/master/pic/13.png?raw=true)

测试案例类中，我们可以看到从HiPoUnit类中继承，其中包含一条案例信息test_book。案例中使用测试。我们可以看到案例的执行过程，首先初始化SearchPage对象，打开相关页面的URL，上面从Excel文件中读取的数据，通过Key的形式读取出来，赋值给相关对象。经过一系列操作过程后，进行结果与预期值的比对，完成测试，最终，生成测试报告。

![](https://github.com/DingTobest/Hi_PO/blob/master/pic/14.png?raw=true)

## 5. 调用过程

![](https://github.com/DingTobest/Hi_PO/blob/master/pic/15.png?raw=true)

最后来到main函数，我们看整体的过程调用，代码很简单，只有5行，代表测试过程中的5个步骤。
- 第一步是进行数据的初始化，读取在Excel中存储的测试数据。
- 第二步是创建测试集。
- 第三步是添加测试案例到测试集，并且将相关的测试数据也一并添加进去
- 第四步是输出测试报告
- 第五步是是发送测试结果的邮件（此步被注释）

以上5个步骤也是比较典型的自动化测试过程的步骤。
## 6. 总结

通过Hi_PO这个UI自动化测试框架范例，比较清晰的可以看到页面对象模型这样的设计模式的结构。

页面对象模型整体把UI自动化测试分为三层：
- 最下层为自动化测试引擎的技术封装，集中在页面处理与对象识别和驱动中
- 在之上的为页面对象层，管理被测系统的功能页面与页面元素
- 在最上层是测试业务层，针对测试案例完成测试流程与校验

最后，将书中所描述的PageObject模式的先进性附在下面。

>PageObject模式的先进性：

>复用性：业务逻辑脚本和页面特征代码之间有了清楚的分割。页面特征代码是指页面定位器等和页面强依赖的测试脚本代码。这样任何涉及同一页面的业务逻辑测试脚本，都会通过同一个页面PageObject类来操作页面，有很高的复用性。

>可维护性：页面提供的服务或者操作都由同意特征代码类提供，有助于在业务测试脚本中进行维护。

> 可读性：通过测试脚本的分层处理，可以更好的将业务逻辑表现在测试脚本代码中测试脚本只有业务逻辑，而不与其他页面内容交互，有很高的可读性。


点击即可进入公众号：知识的阿尔法

[![](https://github.com/DingTobest/res/blob/master/zhishideaerfa.jpg?raw=true)](https://mp.weixin.qq.com/s/FXX0J7iueLTu5YZQkQ9NJw "点击进入公众号：知识的阿尔法")
