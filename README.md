Dzzoffice-2.01后台SQL注入

Dzzoffice介绍:

DzzOffice是一套开源办公套件，适用于企业、团队搭建自己的 类似“Google企业应用套件”、“微软Office365”的企业协同办公平台.

项目地址：https://github.com/zyx0814/dzzoffice/

漏洞描述:

在Dzzoffice-2.01版本后台“网盘”模块，存在SQL注入漏洞，可通过该漏洞获取数据库中的信息。

漏洞分析:

filelist方法中先对一些参数进行判断赋值操作，这些参数虽然有从数据包中获取得到的，但是都被重新赋值，不能被我们的输入控制。

<img width="416" alt="image" src="https://github.com/EternalGemini/dzz/assets/98577439/de7a5c95-1366-4fa1-b532-cb0a3c98fb3c">

接下来判断“doobj”和“do_obj”是否为空，不为空则通过“trim()”函数去除首尾的空格、制表符等后再存入数组“$condition”，这是两个参数是我们可以控制输入的地方，并且没有其他过滤限制。关于获取到的“uids”参数，目前看来是可以利用。
“startdate”和“enddate”参数经过“strtotime()”函数转化为时间戳再存入数组“$condition”；不能被利用。
然后调用fetch_all_event($start, $limit, $condition, $ordersql, true)函数，经过上面的分析，只有“$condition”数组中的“doobj”、“do_obj”和“uids”是可以控制的。

<img width="416" alt="image" src="https://github.com/EternalGemini/dzz/assets/98577439/c8a001ae-b34b-43c1-a58d-881795a9e9f3">

跟进fetch_all_event()函数

<img width="416" alt="image" src="https://github.com/EternalGemini/dzz/assets/98577439/46a94c51-252e-4974-885f-1b223fa8b2c6">

能够控制的参数放在“$condition”数组中，直接找到相关的地方进行进一步分析。
关于处理“$condition”的代码中，只是根据取出来的值不同，采用不同的连接参数，然后赋值给“$wheresql”，最后拼接到SQL语句中执行，在这个过程中没有对“$condition”中取出值做任何过滤。

<img width="415" alt="image" src="https://github.com/EternalGemini/dzz/assets/98577439/b8051b41-8a2a-4dde-9792-253b6db5e14d">

漏洞复现:

由于漏洞需要用户登录，所以我们可能需要一个可用账户和密码。
先进行登录。

![image](https://github.com/EternalGemini/dzz/assets/98577439/1b9fdede-32c2-4be2-a0f5-b205f2fbaf91)

然后进入漏洞发生的地方。

![image](https://github.com/EternalGemini/dzz/assets/98577439/dcac45ca-af4d-417a-a162-2f1217e5c4f4)

抓取数据包，修改“doobj”和“do_obj”为验证代码。

<img width="416" alt="image" src="https://github.com/EternalGemini/dzz/assets/98577439/247413a7-3687-48af-9338-ce77117c4407">
</br>
<img width="415" alt="image" src="https://github.com/EternalGemini/dzz/assets/98577439/d7bb60e2-c35e-4bf8-b7cd-c4984739cd22">

At this point, the repetition is complete.

The affected version

DzzOffice2.01
