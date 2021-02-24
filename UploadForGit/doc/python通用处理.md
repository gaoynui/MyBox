### python多线程和表格输出

多线程：

```
import _thread

// 线程main函数
def parallel():
	// 处理前置逻辑
	queries = getQueries(params)
	try:
		// 多线程
		_thread.start_new_thread(fun1, (params))
		_thread.start_new_thread(fun2, (params))
	except:
		print("error")
	while 1:
		pass
parallel()

注：多线程处理公共变量，如一个队列，生产者消费者参与生产消费。队列不能用list处理，只能用queue。不同线程不会共享一个list。
```

写入csv:

```
import csv

csvFile = open(path, "w", newline='')
wr = csv.writer(csvFile)
wr.writerow(headers字段)
for s : store:
	wr.writerow(行字段)
	
注:在linux系统上生成的csv在windows的excel打开时出现乱码，可通过记事本或者wps打开。
```

写入excel:

```
import os
import xlrd
import xlwt
from xlutils.copy import copy

// 生成表格对象
excel = "表名"
sheetName = "sheetName"
if os.path.exists(excel):
	mExcelFile = xlrd.open_workbook(excel)
	if sheetName in mExcelFile.sheet_names():
		print("error")
	else:
		mExcelFile = copy(mExcelFile)
		mSheet = mExcelFile.add_sheet(sheetName, cell_overwrite_ok=True)
else:
	mExcelFile = xlwt.Workbook()
	mSheet = mExcelFile.add_sheet(sheetName)
	
// 插入header
header = []
mColumnIndex = 0
for mCellValue in header:
    mSheet.write(0, mColumnIndex, mCellValue)
    mColumnIndex = mColumnIndex + 1
    
// 插入数据
rowIndex = 1
for s in store:
	mSheet.write(rowIndex, 0, s[""])
	mSheet.write(rowIndex, 1, s[""])
	mSheet.write(rowIndex, 2, s[""])
	rowIndex += 1
	mExcelFile.save(excel)
```

