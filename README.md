# 数据库实验四

## 实验目的

设计一个写字楼停车位管理系统

存储的数据：车主姓名，车牌，车辆大小，车身颜色

## PORT表的建表语句

```sql
//登记车辆信息，不限于车牌、车辆大小，可以查询和删除
CREATE TABLE PORT
(
	NAME VARCHAR(10) NOT NULL,
	lICENSE CHAR(7),
	SIZE INT,
	COLOR CHAR(1),
	PRIMARY KEY (LICENSE)
);
```

## 完整代码

```python
# -*- coding: utf-8 -*-
from PyQt5.QtWidgets import QListWidget,QApplication,QLineEdit,QWidget,QFormLayout,QPushButton
import sys
import psycopg2

class lineEditDemo(QWidget):
    
    
    def ButtonSearchClicked(self,PList1,PEdit1):
        PList1.clear()
        gConn = psycopg2.connect(host="127.0.0.1", port="54321", user="system", password = "123456", database ="test")

        if gConn:
            print("链接成功")
        
        cur = gConn.cursor()
        cur.execute("select * from PORT")
        rows = cur.fetchall()
        for row in rows:
            PList1.addItem(str(row[0]) + ", " + str(row[1]) + ", " + str(row[2]) + ", " + str(row[3]))
        cur.close()
        gConn.close()
                
    def PButtonInsertClicked(self,PList1,PEdit1):
       # Get the input text and split it by commas
        values = PEdit1.text().split(',')
        # Use with statement to automatically close connection and cursor
        with psycopg2.connect(host="127.0.0.1", port="54321", user="system", password = "123456", database ="test") as gConn:
            with gConn.cursor() as cur:
            # Use parameterized query to insert data
                cur.execute("insert into PORT(NAME,LICENSE,SIZE,COLOR) values(%s, %s, %s, %s)", (values[0], values[1], values[2], values[3]))
                # Commit the transaction
                gConn.commit()
        #Update the list widget to show the inserted data
        self.ButtonSearchClicked(PList1, PEdit1)
        cur.close()
        gConn.close()
  
        
    def ButtonDeleteHaveClicked(self,PList1,PEdit1):
        # Get the input text and split it by commas
        row = PList1.currentRow()
        item = PList1.item(row)
        values = item.text().split(',')
        License=values[1][1::]
        print(License)
        # Use with statement to automatically close connection and cursor
        with psycopg2.connect(host="127.0.0.1", port="54321", user="system", password = "123456", database ="test") as gConn:
            with gConn.cursor() as cur:
                try:
                    # Use parameterized query to delete data
                    cur.execute("DELETE FROM PORT WHERE(LICENSE =%s);", (License,))            
                except (Exception, psycopg2.DatabaseError) as error:
                    print(error)
        # Commit the transaction
        gConn.commit()
        #Update the list widget to show the inserted data
        self.ButtonSearchClicked(PList1, PEdit1)
        cur.close()
        gConn.close()

    def ButtonQuitClicked(self,PList1,PEdit1):

        app.quit()

        
    def __init__(self,parent=None):
        super(lineEditDemo, self).__init__(parent)
        self.setWindowTitle('芝士数据库')
        self.resize(1840, 950)  

        #实例化表单布局
        flo=QFormLayout()

        #创建4个文本输入框
        PNormalLineEdit=QLineEdit()
        PButtonSearch=QPushButton('查询已有')


        PButtonInsert=QPushButton('插入上述')

        PButtonDeleteHave=QPushButton('删除选择')
        PButtonQuit=QPushButton('退出程序')
        
        PList=QListWidget()
        PList.setResizeMode(1);
        PButtonSearch.clicked.connect(lambda: self.ButtonSearchClicked(PList,PNormalLineEdit)) 
        PButtonInsert.clicked.connect(lambda: self.PButtonInsertClicked(PList,PNormalLineEdit)) 

        PButtonDeleteHave.clicked.connect(lambda: self.ButtonDeleteHaveClicked(PList,PNormalLineEdit)) 
        PButtonQuit.clicked.connect(lambda: self.ButtonQuitClicked(PList,PNormalLineEdit)) 

        self.PList = PList
        self.PEdit1 = PNormalLineEdit


        #添加到表单布局中
        #flo.addRow(文本名称（可以自定义），文本框)
        flo.addRow('按顺序输入待插入记录（以英文逗号间隔）',PNormalLineEdit)
        flo.addRow('请点击插入上述记录',PButtonInsert)
        flo.addRow('请点击查询已有记录',PButtonSearch)
        flo.addRow('消息列表',PList)
        flo.addRow('请选中一行，点击删除该记录',PButtonDeleteHave)


        #设置setPlaceholderText()文本框浮现的文字
        PNormalLineEdit.setPlaceholderText('')

        #setEchoMode()：设置显示效果

        #QLineEdit.Normal：正常显示所输入的字符，此为默认选项
        PNormalLineEdit.setEchoMode(QLineEdit.Normal)

        #设置窗口的布局
        self.setLayout(flo)
        

        
        def resizeEvent(self, e):
            self.PList.height = self.flo.height -20
            
        
            

if __name__ == '__main__':
    if not QApplication.instance():
        app = QApplication(sys.argv)
    else:
        app = QApplication.instance()
    win=lineEditDemo()
    win.show()

    sys.exit(app.exec_())


```

## 代码说明

查询函数

```python
		PList1.clear()
        gConn = psycopg2.connect(host="127.0.0.1", port="54321", user="system", password = "123456", database ="test")

        if gConn:
            print("链接成功")
        
        cur = gConn.cursor()
        cur.execute("select * from PORT")
        rows = cur.fetchall()
        #打印PORT表
        for row in rows:
            PList1.addItem(str(row[0]) + ", " + str(row[1]) + ", " + str(row[2]) + ", " + str(row[3]))  
        cur.close()
        gConn.close()
```

插入函数

```python
		# 获取输入的文本并以逗号分割
        values = PEdit1.text().split(',')
        # 与数据库连接
        with psycopg2.connect(host="127.0.0.1", port="54321", user="system", password = "123456", database ="test") as gConn:
            with gConn.cursor() as cur:
            # 执行插入语句
                cur.execute("insert into PORT(NAME,LICENSE,SIZE,COLOR) values(%s, %s, %s, %s)", (values[0], values[1], values[2], values[3]))
                # 提交事务
                gConn.commit()
        #更新PORT表
        self.ButtonSearchClicked(PList1, PEdit1)
        cur.close()
        gConn.close()
```

删除函数

```python
		# 获取鼠标点击的文本并以逗号分割
        row = PList1.currentRow()
        item = PList1.item(row)
        values = item.text().split(',')
        # 由于处理后的文本在首端多出一个空格，这里将字符串切割后赋值给License
        License=values[1][1::]
        print(License)
        # 与数据库连接
        with psycopg2.connect(host="127.0.0.1", port="54321", user="system", password = "123456", database ="test") as gConn:
            with gConn.cursor() as cur:
                try:
                    # 执行删除语句
                    cur.execute("DELETE FROM PORT WHERE(LICENSE =%s);", (License,))   
                    # 异常处理
                except (Exception, psycopg2.DatabaseError) as error:
                    print(error)
        # 提交事务
        gConn.commit()
        #更新PORT表
        self.ButtonSearchClicked(PList1, PEdit1)
        cur.close()
        gConn.close()
```

