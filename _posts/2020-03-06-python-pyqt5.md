---
layout: post
title: 'Python+PyCharm+PyQt5图形化开发'
date: '2020-03-06'
tags:
     - python
author: '王小胖'
---

# 环境准备

## 安装python、pycharm和pyqt5
----
我的本地环境为windows7 64位、python版本3.7。

安装好python后，通过pip工具安装好图形化开发所需的类库（pyqt5、pyqt5-sip、pyqt5-tools），可以在pycharm中查看是否已安装所需类库。
![image](img/2020-03-06-python-pyqt5/pyqt5_1.png)

## PyCharm基本配置
----
创建新project：csv2txt

设置默认PyCharm解析器，与1中展示图一致

选择File -> Settings -> Project: csv2txt -> Project Interpreter，设置 Project Interpreter为你使用的python版本

配置PyQt

PyCharm中选择File -> Settings -> Tools -> External Tools，点击+ 新建工具， 建立QTdesigner 和 PyUIC工具
              
QTdesigner 

配置两个关键参数：

Program：自己的designer.exe路径 例如C:\Program Files\Python37\Lib\site-packages\pyqt5_tools\Qt\bin\designer.exe

Working directory：$ProjectFileDir$
![image](img/2020-03-06-python-pyqt5/pyqt5_2.png)

PyUIC

配置三个关键参数：

Program：自己的python.exe路径 例如C:\Program Files\Python37\Lib\site-packages\pyqt5_tools\Qt\bin\designer.exe

Arguments：-m PyQt5.uic.pyuic $FileName$ -o $FileNameWithoutExtension$.py

Working directory：$ProjectFileDir$
![image](img/2020-03-06-python-pyqt5/pyqt5_3.png)

# GUI界面开发步骤

## 利用QT Designer代码生成一个GUI界面 ，并用PyUIC转换为.py文件
----
![image](img/2020-03-06-python-pyqt5/pyqt5_4.png)

![image](img/2020-03-06-python-pyqt5/pyqt5_5.png)

配置完成后保存，会在project目录下创建untitled.ui文件，然后右键选择External Tools->PyUIC 就会生成Utitled.ui对应的Utitled.py文件

![image](img/2020-03-06-python-pyqt5/pyqt5_6.png)

## 将PyUIC转换后的.py文件中的内容添加到已开发完成的python脚本中
----
```
# -*- coding: utf-8 -*-

# Form implementation generated from reading ui file 'untitled.ui'
#
# Created by: PyQt5 UI code generator 5.13.0
#
# WARNING! All changes made in this file will be lost!

import pandas  
import os
import sys
import csv
import cx_Oracle
from PyQt5 import QtCore, QtGui, QtWidgets



class Ui_MainWindow(object):
    def setupUi(self, MainWindow):
        MainWindow.setObjectName("MainWindow")
        MainWindow.resize(843, 607)
        self.centralwidget = QtWidgets.QWidget(MainWindow)
        self.centralwidget.setObjectName("centralwidget")
        self.lineEdit = QtWidgets.QLineEdit(self.centralwidget)
        self.lineEdit.setGeometry(QtCore.QRect(190, 230, 161, 31))
        self.lineEdit.setObjectName("lineEdit")
        self.label = QtWidgets.QLabel(self.centralwidget)
        self.label.setGeometry(QtCore.QRect(20, 190, 171, 31))
        font = QtGui.QFont()
        font.setFamily("Aharoni")
        font.setPointSize(16)
        font.setBold(True)
        font.setWeight(75)
        self.label.setFont(font)
        self.label.setObjectName("label")
        self.label_2 = QtWidgets.QLabel(self.centralwidget)
        self.label_2.setGeometry(QtCore.QRect(20, 230, 151, 31))
        font = QtGui.QFont()
        font.setFamily("Aharoni")
        font.setPointSize(14)
        font.setBold(True)
        font.setWeight(75)
        self.label_2.setFont(font)
        self.label_2.setObjectName("label_2")
        self.label_3 = QtWidgets.QLabel(self.centralwidget)
        self.label_3.setGeometry(QtCore.QRect(20, 290, 151, 31))
        font = QtGui.QFont()
        font.setFamily("Aharoni")
        font.setPointSize(14)
        font.setBold(True)
        font.setWeight(75)
        self.label_3.setFont(font)
        self.label_3.setObjectName("label_3")
        self.lineEdit_2 = QtWidgets.QLineEdit(self.centralwidget)
        self.lineEdit_2.setGeometry(QtCore.QRect(190, 290, 161, 31))
        self.lineEdit_2.setObjectName("lineEdit_2")
        self.label_4 = QtWidgets.QLabel(self.centralwidget)
        self.label_4.setGeometry(QtCore.QRect(20, 350, 151, 31))
        font = QtGui.QFont()
        font.setFamily("Aharoni")
        font.setPointSize(14)
        font.setBold(True)
        font.setWeight(75)
        self.label_4.setFont(font)
        self.label_4.setObjectName("label_4")
        self.lineEdit_3 = QtWidgets.QLineEdit(self.centralwidget)
        self.lineEdit_3.setGeometry(QtCore.QRect(190, 350, 161, 31))
        self.lineEdit_3.setObjectName("lineEdit_3")
        self.label_5 = QtWidgets.QLabel(self.centralwidget)
        self.label_5.setGeometry(QtCore.QRect(20, 410, 151, 31))
        font = QtGui.QFont()
        font.setFamily("Aharoni")
        font.setPointSize(14)
        font.setBold(True)
        font.setWeight(75)
        self.label_5.setFont(font)
        self.label_5.setObjectName("label_5")
        self.lineEdit_4 = QtWidgets.QLineEdit(self.centralwidget)
        self.lineEdit_4.setGeometry(QtCore.QRect(190, 410, 161, 31))
        self.lineEdit_4.setObjectName("lineEdit_4")
        self.pushButton = QtWidgets.QPushButton(self.centralwidget)
        self.pushButton.setGeometry(QtCore.QRect(20, 490, 101, 31))
        self.pushButton.setObjectName("pushButton")
        self.textBrowser = QtWidgets.QTextBrowser(self.centralwidget)
        self.textBrowser.setGeometry(QtCore.QRect(370, 40, 451, 471))
        self.textBrowser.setObjectName("textBrowser")
        self.pushButton_2 = QtWidgets.QPushButton(self.centralwidget)
        self.pushButton_2.setGeometry(QtCore.QRect(210, 490, 101, 31))
        self.pushButton_2.setObjectName("pushButton_2")
        self.label_6 = QtWidgets.QLabel(self.centralwidget)
        self.label_6.setGeometry(QtCore.QRect(20, 30, 181, 31))
        font = QtGui.QFont()
        font.setFamily("Aharoni")
        font.setPointSize(16)
        font.setBold(True)
        font.setWeight(75)
        self.label_6.setFont(font)
        self.label_6.setObjectName("label_6")
        self.lineEdit_5 = QtWidgets.QLineEdit(self.centralwidget)
        self.lineEdit_5.setGeometry(QtCore.QRect(20, 80, 301, 21))
        self.lineEdit_5.setObjectName("lineEdit_5")
        MainWindow.setCentralWidget(self.centralwidget)
        self.menubar = QtWidgets.QMenuBar(MainWindow)
        self.menubar.setGeometry(QtCore.QRect(0, 0, 843, 23))
        self.menubar.setObjectName("menubar")
        MainWindow.setMenuBar(self.menubar)
        self.statusbar = QtWidgets.QStatusBar(MainWindow)
        self.statusbar.setObjectName("statusbar")
        MainWindow.setStatusBar(self.statusbar)

        self.retranslateUi(MainWindow)
        QtCore.QMetaObject.connectSlotsByName(MainWindow)

    def retranslateUi(self, MainWindow):
        _translate = QtCore.QCoreApplication.translate
        MainWindow.setWindowTitle(_translate("MainWindow", "MainWindow"))
        self.label.setText(_translate("MainWindow", "请输入以下信息"))
        self.label_2.setText(_translate("MainWindow", "BXJGJRXKZNO"))
        self.label_3.setText(_translate("MainWindow", "BXJGTJDM"))
        self.label_4.setText(_translate("MainWindow", "BXJGMC"))
        self.label_5.setText(_translate("MainWindow", "CJRQ"))
        self.pushButton.setText(_translate("MainWindow", "生成CSV文件"))
        self.pushButton_2.setText(_translate("MainWindow", "CSV转TXT"))
        self.label_6.setText(_translate("MainWindow", "请添加数据库信息"))

    def setupFunction(self):
        self.pushButton.clicked.connect(self.createcsv)
        self.pushButton_2.clicked.connect(self.csv2txt)


    	  
    def createcsv(self):
        BXJGJRXKZNO = self.lineEdit.text()
        BXJGTJDM = self.lineEdit_2.text()
        BXJGMC = self.lineEdit_3.text()
        CJRQ = self.lineEdit_4.text()
        dbconnect = self.lineEdit_5.text()
        csvPath = os.path.abspath(os.path.join(os.getcwd(), "..", "csv"))
        orcl = cx_Oracle.connect(dbconnect)
        curs = orcl.cursor()
        printHeader = True  # include column headers in each table output
        sql = "select table_name from csv2txt"  # get a list of all tables
        curs.execute(sql)
        for row_data in curs:
            if not row_data[0].startswith('BIN$'):  # skip recycle bin tables
                tableName = row_data[0]
            # output each table content to a separate CSV file
                csv_file_dest = csvPath + '\\' + BXJGJRXKZNO + "-" + tableName + "-" + CJRQ + ".csv"
                outputFile = open(csv_file_dest, 'w')  # 'wb'
                output = csv.writer(outputFile, dialect='excel')
                sql = "select a.*,'" + BXJGJRXKZNO + "','" + BXJGTJDM + "','" + BXJGMC + "','" + CJRQ + "' from " + tableName + " a"
                curs2 = orcl.cursor()
                curs2.execute(sql)
                if printHeader:
                    cols = []
                    for col in curs2.description:
                        cols.append(col[0])
                    output.writerow(cols)
                for row_data in curs2:
                    output.writerow(row_data)
                outputFile.close()
                self.textBrowser.append("开始导出CSV文件......")
                self.textBrowser.append("已成功导出文件：" + csv_file_dest)
                self.textBrowser.append("导出CSV文件任务已完成！" )
                self.textBrowser.append("====================")
                self.textBrowser.append("====================")
                        
    def csv2txt(self):
        csvPath = os.path.abspath(os.path.join(os.getcwd(), "..", "csv"))
        outPath = os.path.abspath(os.path.join(os.getcwd(), "..", "txt"))
        csvLists = os.listdir(csvPath)
        for csvlist in csvLists:
            (filepath, fullname) = os.path.split(csvlist)
            (prename, suffix) = os.path.splitext(fullname)
            df = pandas.read_csv(csvPath + '\\' + csvlist, dtype=object)
            df.to_csv(outPath + '\\' + prename + '.txt', header=None, sep='', index=None, line_terminator='\n')
            self.textBrowser.append("开始转换为TXT文件......")
            self.textBrowser.append("已成功转换文件：" + outPath + '\\' + prename + '.txt')
            self.textBrowser.append("转换文件任务已完成！" )
            self.textBrowser.append("====================")
            self.textBrowser.append("====================")    
                  
if __name__ == "__main__":
    app = QtWidgets.QApplication(sys.argv)  # 创建一个QApplication，也就是你要开发的软件app
    MainWindow = QtWidgets.QMainWindow()    # 创建一个QMainWindow，用来装载你需要的各种组件、控件
    ui = Ui_MainWindow()                    # ui是Ui_MainWindow()类的实例化对象
    ui.setupUi(MainWindow)                  # 执行类中的setupUi方法，方法的参数是第二步中创建的QMainWindow
    ui.setupFunction()                            # 执行类中的setupFunction方法
    MainWindow.show()                       # 执行QMainWindow的show()方法，显示这个QMainWindow
    sys.exit(app.exec_())                   # 使用exit()或者点击关闭按钮退出QApplication
```
## 脚本保存为csv2txtmain.py后，使用pyinstaller进行打包
----
![image](img/2020-03-06-python-pyqt5/pyqt5_7.png)


> 如有任何知识产权、版权问题或理论错误，还请指正。
>
> 转载请注明原作者及以上信息。
