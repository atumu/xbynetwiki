title: xlsxwriter 

#  XlsxWriter创建excel 
官网：https://xlsxwriter.readthedocs.io/
Demo:https://github.com/jmcnamara/XlsxWriter/tree/master/examples
当前版本:0.9.3

pip install XlsxWriter

```

import xlsxwriter
workbook = xlsxwriter.Workbook('hello.xlsx')
worksheet = workbook.add_worksheet()
#worksheet.set_column('A:A', len('hello world')+1)
worksheet.write('A1', 'Hello world')
workbook.close()

```
就是创建文件，然后增加sheet，然后写入数据，然后关闭。
其中那个set_column中的'A:A'是一个列的范围，这里就是A列，如果是A到E列就是'A:E'。
```

import xlsxwriter
# Create an new Excel file and add a worksheet.
workbook = xlsxwriter.Workbook('demo.xlsx')
worksheet = workbook.add_worksheet()
# Widen the first column to make the text clearer.
worksheet.set_column('A:A', 20)
# Add a bold format to use to highlight cells.
bold = workbook.add_format({'bold': True})
# Write some simple text.
worksheet.write('A1', 'Hello')
# Text with formatting.
worksheet.write('A2', 'World', bold)
# Write some numbers, with row/column notation.
worksheet.write(2, 0, 123)
worksheet.write(3, 0, 123.456)
# Insert an image.
worksheet.insert_image('B5', 'logo.png')
worksheet1.insert_image('B6', 'logo.png', {'x_scale':0.2, 'y_scale':0.2})
workbook.close()

```
##  创建带格式的Excel表格 
```

# 添加一些格式
import xlsxwriter
workbook = xlsxwriter.Workbook('test2.xlsx')
worksheet = workbook.add_worksheet('test2')

bold = workbook.add_format({'bold': True})  # 设置粗体，默认False
money = workbook.add_format({'num_format': '$#,##0'})   # 定义数字格式
#date_format = workbook.add_format({'num_format': 'mmmm d yyyy'}) 日期格式
worksheet.write('A1', 'Item', bold)     # 设置自定义表头加粗
worksheet.write('B1', 'Cost', bold)

expenses = (
     ['Rent', 1000],
     ['Gas',   100],
     ['Food',  300],
     ['Gym',    50],
 )

row = 1
col = 0

for item, cost in (expenses):
    worksheet.write(row, col, item)    # 默认格式写入
    worksheet.write(row, col + 1, cost, money)   # 设置带money格式写入
    row += 1

worksheet.write(row, 0, 'Total', bold)
worksheet.write(row, 1, '=SUM(B2:B5)', money)

workbook.close()

```
##  创建excel图表 
```

import xlsxwriter
import random
 
def get_num():
    return random.randrange(0, 201, 2)
 
workbook = xlsxwriter.Workbook('analyse_spider.xlsx')    #创建一个Excel文件
worksheet = workbook.add_worksheet()    #创建一个工作表对象
chart = workbook.add_chart({'type': 'column'})    #创建一个图表对象
#定义数据表头列表
title = [u'业务名称',u'星期一',u'星期二',u'星期三',u'星期四',u'星期五',u'星期六',u'星期日',u'平均流量']
buname= [u'时光网',u'汽车之家',u'weixin.com',u'163.com',u'baidu.com']    #定义频道名称
#定义5频道一周7天流量数据列表
data = []
for i in range(5):
    tmp = []
    for j in range(7):
        tmp.append(get_num())
    data.append(tmp)
    
format=workbook.add_format()    #定义format格式对象
format.set_border(1)    #定义format对象单元格边框加粗(1像素)的格式
 
format_title=workbook.add_format()    #定义format_title格式对象
format_title.set_border(1)   #定义format_title对象单元格边框加粗(1像素)的格式
format_title.set_bg_color('#cccccc')   #定义format_title对象单元格背景颜色为
                                       #'#cccccc'的格式
format_title.set_align('center')    #定义format_title对象单元格居中对齐的格式
format_title.set_bold()    #定义format_title对象单元格内容加粗的格式
 
format_ave=workbook.add_format()    #定义format_ave格式对象
format_ave.set_border(1)    #定义format_ave对象单元格边框加粗(1像素)的格式
format_ave.set_num_format('0.00')   #定义format_ave对象单元格数字类别显示格式
 
#下面分别以行或列写入方式将标题、业务名称、流量数据写入起初单元格，同时引用不同格式对象
worksheet.write_row('A1',title,format_title)  
worksheet.write_column('A2', buname,format)
worksheet.write_row('B2', data[0],format)
worksheet.write_row('B3', data[1],format)
worksheet.write_row('B4', data[2],format)
worksheet.write_row('B5', data[3],format)
worksheet.write_row('B6', data[4],format)
 
#定义图表数据系列函数
def chart_series(cur_row):
    worksheet.write_formula('I'+cur_row, \
     '=AVERAGE(B'+cur_row+':H'+cur_row+')',format_ave)    #计算（AVERAGE函数）频
                                                          #道周平均流量
    chart.add_series({
        'categories': '=Sheet1!$B$1:$H$1',    #将“星期一至星期日”作为图表数据标签(X轴)
        'values':     '=Sheet1!$B$'+cur_row+':$H$'+cur_row,    #频道一周所有数据作
                                                               #为数据区域
        'line':       {'color': 'black'},    #线条颜色定义为black(黑色)
        'name': '=Sheet1!$A$'+cur_row,    #引用业务名称为图例项
    })
 
for row in range(2, 7):    #数据域以第2～6行进行图表数据系列函数调用
    chart_series(str(row))
 
chart.set_size({'width': 577, 'height': 287})    #设置图表大小
chart.set_title ({'name': u'爬虫分析'})    #设置图表（上方）大标题
chart.set_y_axis({'name': 'count'})    #设置y轴（左侧）小标题
 
worksheet.insert_chart('A8', chart)    #在A8单元格插入图表
workbook.close()    #关闭Excel文档

```