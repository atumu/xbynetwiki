title: python学习笔记109 

#  Python学习笔记之PyQt4之多线程修改GUI界面 
参考http://www.cnblogs.com/ribavnu/p/4721256.html
- GUI所在的线程是主线程，线程之间可以通过信号槽来通信，这个方法适用于重写PyQt部件类后对外开发接口。
**基本思路，定义一个QtCore.QThread子类，实现其run方法，在该类中定义信号（这个信号连接GUI线程的一个槽），然后在特定情况下发送信号即可。**
```

from PyQt4 import QtGui, QtCore, Qt
from Ui_113 import  Ui_container
import sys, os
import requests
from bs4 import BeautifulSoup
import random
from datetime import date
import threading, math
import re
 
headers={
        "Referer":"http://www.mm131.com/",
        "User-Agent":"Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/42.0.2311.90 Safari/537."
    }
class MyThread(QtCore.QThread):
    progressSignal=QtCore.pyqtSignal(int,int, str)
    def __init__(self,parent=None):
        super(MyThread,self).__init__(parent)
    def initArgs(self, siteUrl, dir):
        self.siteUrl=siteUrl
        self.dir=dir
    def run(self):
        self.getMM131Img(self.siteUrl, self.dir)
    def getMM131Img(self, siteUrl, dir):
        resp=requests.get(siteUrl)
        resp.encoding='gb2312'
        soup = BeautifulSoup(resp.text,'html5lib')
        dirname=soup.find('div',class_='content').h5.string #获取title作为保存目录名
        pagecount=soup.find('div',class_='content-page').span.string
        tmpm = re.findall(r'\d+',pagecount) #正则从字符串提取数字
        piccount=int(tmpm[0])
        imgStr=soup.find('div',class_='content-pic').img['src'] #获取图片url
        prefix=imgStr[:imgStr.rfind("/")+1] #获取图片url前缀
        picext="."+imgStr.split(".")[-1] #获取图片后缀名
        resp.close()
        session=requests.Session()
        index=-1 #保存循环索引
        if not dirname:
            dirname=date.today().strftime("%Y%m%d")
        saveDir=os.path.join(dir,dirname) #得到最终保存目录
        if not os.path.exists(saveDir) :
            os.mkdir(saveDir)  
        for img in [ prefix+str(i+1)+picext for i in range(piccount)]:
            name=img.split('/')[-1] #图片名称
            index+=1
            filename=os.path.join(saveDir,str(random.randrange(1000))+name)#图片保存路径
           
            with open(filename,'wb') as f:
                print(img)
                try:
                    resp1=session.get(img,headers=headers,timeout=30,stream=True)
                    for chunk in resp1.iter_content(chunk_size=512):
                        f.write(chunk)
                    resp1.close()
                 
                    self.progressSignal.emit(index, math.ceil((index+1)/piccount*100), img)
                except:
                    self.progressSignal.emit(index, math.ceil((index+1)/piccount*100), img+"下载发生异常")
                    print(sys.exc_info())
        session.close()      
        print("图片下载完成")
     
class MainWindow(QtGui.QWidget):
    def __init__(self, parent=None):
        super(MainWindow, self).__init__()
        self.ui=Ui_container()
        self.ui.setupUi(self)
        QtCore.QObject.connect(self.ui.startBtn, QtCore.SIGNAL("clicked()"), self.loadImg)
        QtCore.QObject.connect(self.ui.selectDirW, QtCore.SIGNAL("clicked()"), self.selectDir)
        #QtCore.QObject.connect(self.selectDirW, QtCore.SIGNAL(_fromUtf8("clicked()")), self.dirEdit.copy)
        self.events=MyThread()
        self.events.progressSignal.connect(self.progressHandle)
    def loadImg(self):
        self.ui.progressBar.setValue(0)
        url=self.ui.urlEdit.text()
        dir=self.ui.dirEdit.text()
        #t=threading.Thread(target=self.getMM131Img,args=(url,dir))
        #t.start()
        self.events.initArgs(url, dir)
        self.ui.imgListShow.clear()
        self.events.start()
    def selectDir(self):
        fname = QtGui.QFileDialog.getExistingDirectory(self, '打开',
                os.getcwd())
        self.ui.dirEdit.setText(fname)
    def progressHandle(self, index, percent, url):
        self.ui.progressBar.setValue(percent)
        self.ui.imgListShow.addItem(url)
        if(percent==100):
            self.ui.imgListShow.addItem("下载完成!")
     
 
 
if __name__=="__main__":
    app=QtGui.QApplication(sys.argv)
    form=MainWindow()
    form.show()
    sys.exit(app.exec_())


```