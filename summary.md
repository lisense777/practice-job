# 大华实习总结

>  首先介绍一下我在大华实习的岗位是应用软件工程师，我们组负责的是一个smartpss客户端。
>
> smartpss是大华公司出品的一款远程监控软件，通过网络的连接将众多需要监控的摄像头进行连接，将街道、小区入口、或者停车厂等地区进行拍摄。停车场是另一个parking lot项目  主要监控车辆进入和离开停车场的时间 方便计算停车费用。



而我负责的是smartpss监控系统中的考勤人事管理系统，在这个项目代码中，主要使用qt开发。

考勤人事管理系统主要有创建组织结构、录入组织人员配置权限、配置人员认证方式



用户考勤打卡系统实现的主要功能

1.  实时显示当前时间

Qlabel对象  显示在考勤管理系统中的文本图像

通过qt中定义的信号和槽的方法实现界面切换（槽函数可以和信号连接起来，信号发生时，槽函数会被自动调用）

connect、slot、signal三个重要函数



通过定时器，每隔一秒获得当前时间，获得后显示在Qlabel中，将时间信息变成英文显示在相应的label中

 QString mouth[12] ={"Jan","Feb","Mar","Apr","May",
"June","July","Aug","Sept","Oct","Nov","Dec"};

connect(&timer,&QTimer::timeout,this,&ArmFace::run_time); //绑定定时器信号
timer.start(1000);

//将时间改变相应的格式显示在相应的label中
void ArmFace::show_time(int mm,QString dd, QString hhmm,QString dddd)
{
  //显示时间
  ui->label_4->setText(hhmm);
  QString week;
  if(dddd == "星期一") week = "Monday";
  else if(dddd == "星期二") week = "Tuesday";
  else if(dddd == "星期三") week = "Wednesday";
  else if(dddd == "星期四") week = "Thursday";
  else if(dddd == "星期五") week = "Friday";
  else if(dddd == "星期六") week = "Saturday";
  else if(dddd == "星期天") week = "Sunday";
  QString data = week+" "+mouth[mm-1]+" "+dd;
  ui->label_5->setText(data);

}
void ArmFace::run_time()
{
  //获取当前时间
  QString time = QDateTime::currentDateTime().toString("MM dd hh:mm dddd");
  int mm = time.mid(0,2).toInt();
  QString dd = time.mid(3,2);
  QString hhmm = time.mid(6,5);
  QString dddd = time.mid(12,4);
  show_time(mm,dd,hhmm,dddd);
  /*qDebug()<<mm;
  qDebug()<<hhmm;
  qDebug()<<dddd;
  qDebug()<<time;*/
} 





2.opencv实现 人脸识别打卡（个人并不会openc实现的人脸识别算法，因为在公司大家都是一起开发项目，所以直接用了别人写好的算法）

通过多次识别的结果和当天的日期去数据库的考勤打卡表找是否存在这个标签信息；如果存在就会弹出已打卡的信息；不存在的话就会弹出 您已打卡成功

//首先是拿到已经训练好的图片信息 和 数据库中的人脸信息比对五次

i f(detection == Predict(pImage_roi[i]))
{
   num++;
   qDebug()<<"num"<<num;
   if(num == 5 || flag ==1)
   {
     //判断是否打卡，已打则不进行打卡，在考勤打卡表中查询
     QString Time = time+"%";
     QString sql_select = QString("select * from Attend where Sno='%1'and Time like '%2'").arg(Predict(pImage_roi[i])).arg(Time);
     QSqlQuery query_select(sql_select);
     while(query_select.next())
     {
       //在用户信息表根据对应的标签(学号)拿出名字
       QString sql = QString("select * from Stu where Sno='%1'").arg(Predict(pImage_roi[i]));
       QSqlQuery query(sql);
       while(query.next())
       {
         num = 0;
         flag = 0;
         detection = 0;
       

         QString message = query.value(2).toString()+"，您已打卡";
         QMessageBox::about(this,"提示",message);
     
         mtimer.stop();
         capture.release();
         ui->openbt->setStyleSheet("border-image:url(:/close.png)");
         return ;
       }
       return ;
      }
     
     //根据编号查询数据库(判断用户信息表中是否存在该用户)
     QString sql = QString("select * from Stu where Sno='%1'").
arg(Predict(pImage_roi[i]));
     QSqlQuery query(sql);
     while(query.next())
     {
        num = 0;
        flag = 0;
        detection = 0;
       qDebug()<<query.value(0).toInt()<<query.value(1).toString()<<query.value(2).toString();
       
```c++
    QString message = query.value(2).toString()+": 打卡成功";
    QMessageBox::about(this,"提示",message);
      
    mtimer.stop();
    capture.release();
    ui->openbt->setStyleSheet("border-image:url(:/close.png)");
  
    //将打卡信息存在Attend(考勤打卡表)中
    QString time = QDateTime::currentDateTime().toString("MM-dd hh:mm");
    QString sql = QString("insert into Attend(Sno,time,State) values(%1,'%2',%3)").arg(Predict(pImage_roi[i])).arg(time).arg(1);
       
    SqlQuery query;
    if(!query.exec(sql))
    {
      qDebug()<<"add error";
    }
    else
    {
      qDebug()<<"添加成功";
    }
 
   }
 
    num =0;
   }
}
else {flag =0;qDebug()<<"不等";num = 0;}
 
detection = Predict(pImage_roi[i]);
```

管理员功能：

对用户信息录入

员工工号、姓名、权限、等录入到数据库中的员工信息表方便比对。

**查看考勤情况并统计**

 功能是显示当天已打卡和未打卡的用户，如果未打卡会用红色标记，并统计打卡和未打卡人数，并通过这两个数据绘制饼状图。今天已打卡的用户，只需要在考勤打卡表中查找，今天未打卡的用户需要通过用户表以及考勤打卡表进行查找。

未打卡的用户  就是数据库中总人数减去已经打卡的人数。

**管理员发布管理通知**

只需要对通知信息表进行操作，实现对通知的发布、显示、以及删除

发布就是把输入的标题、内容、日期存入通知信息表 

显示就是查看信息表内容

删除就是删除某条信息的标题内容日期等信息。

