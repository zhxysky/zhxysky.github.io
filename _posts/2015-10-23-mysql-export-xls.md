---
layout: post
title: MySQL Workbeanch导出csv身份证显示错误问题
category: csv
tags: [mysql, MySQL Workbeanch, excel, xls, csv]
---

一般情况下使用mysql的界面管理工具MySQL Workbench可以直接导出导出xls或者csv格式的数据，但是如果数据中含有身份证号码，那么导出的csv打开的时候身份证会默认以科学计数法的方式显示，就算把单元格格式设置回文本，身份证显示还是错误的，后面几位显示为0。那么该如果处理这种情况呢？解决办法如下：


1.在MySQL Workbench导出csv的时候，文件格式不要直接选择csv，而是选择CSV(;separated)(*.csv)

![export csv](/images/csv/1.png "export csv")

2.打开excel，选择“数据”选项卡里面的“导入数据”，找到想要打开的csv文件

![import csv](/images/csv/2.jpg "import csv")

![select csv](/images/csv/3.png "select csv")

3.导入编码选择默认的UTF-8即可，当然你可以根据自己的实际情况进行更改，然后进行下一步

![encode](/images/csv/4.png "encode")

4.选择使用分隔符还是固定宽度，这里使用默认选项分隔符。然后下一步

![separator](/images/csv/5.png "separator")

5.选择要使用的分隔符。这里选择了分号和逗号。然后下一步

![separator](/images/csv/6.png "separator")

6.选择要把类型设置为文本格式的列。这里选择的身份证的列，然后修改的文本格式。当然其他的列也可以修改。然后点击完成

![set text](/images/csv/7.png "set text")

可以看到身份证一列的数据可以正确显示了。