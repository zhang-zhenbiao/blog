---
title: Python发送带附件电子邮件
date: 2017-11-28 13:02:59
tags: Python

---

可采用email模块发送电子邮件附件。发送一个未知MIME类型的文件附件其基本思路如下：



- 构造MIMEMultipart对象做为根容器

- 构造MIMEText对象做为邮件显示内容并附加到根容器

- 构造MIMEBase对象做为文件附件内容并附加到根容器
　　a. 读入文件内容并格式化
　　b. 设置附件头

- 设置根容器属性

- 得到格式化后的完整文本

- 用smtp发送邮件
<!-- more -->


    # -*- coding: utf-8 -*-  
    import smtplib  
    import email.MIMEMultipart# import MIMEMultipart  
    import email.MIMEText# import MIMEText  
    import email.MIMEBase# import MIMEBase  
    import os.path  
    import mimetypes 
    import datetime
    now_time = datetime.datetime.now()
    yes_time = now_time + datetime.timedelta(days = -1)
    yes_times = yes_time.strftime('%Y-%m-%d')
    n = str(yes_times) + ".xls"
    From = "xxxxx@163.com"  
    To = ["xxxxx@163.com","xxxxx@163.com","xxxxx@163.com"]
    file_name = "/data/meizu/" + n#附件名  
    #file_name = "/data/meizu/2017-06-30.xls" #附件名  
      
    server = smtplib.SMTP("smtp.163.com")  
    server.login("xxxxx@163.com","xxxxxx") #仅smtp服务器需要验证时  
      
    # 构造MIMEMultipart对象做为根容器  
    main_msg = email.MIMEMultipart.MIMEMultipart()  
      
    # 构造MIMEText对象做为邮件显示内容并附加到根容器  
    text_msg = email.MIMEText.MIMEText("您好，这是昨日数据，请查收",_charset="utf-8")  
    main_msg.attach(text_msg)  
      
    # 构造MIMEBase对象做为文件附件内容并附加到根容器  
      
    ## 读入文件内容并格式化 [方式1]－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－  
    data = open(file_name, 'rb')  
    ctype,encoding = mimetypes.guess_type(file_name)  
    if ctype is None or encoding is not None:  
    ctype = 'application/octet-stream'  
    maintype,subtype = ctype.split('/',1)  
    file_msg = email.MIMEBase.MIMEBase(maintype, subtype)  
    file_msg.set_payload(data.read())  
    data.close( )  
    email.Encoders.encode_base64(file_msg)#把附件编码  
    ''''' 
     测试识别文件类型：mimetypes.guess_type(file_name) 
     rar 文件 ctype,encoding值：None None（ini文件、csv文件、apk文件） 
     txt text/plain None 
     py  text/x-python None 
     gif image/gif None 
     png image/x-png None 
     jpg image/pjpeg None 
     pdf application/pdf None 
     doc application/msword None 
     zip application/x-zip-compressed None 
    encoding值在什么情况下不是None呢？以后有结果补充。 
    '''  
    #－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－  
    ## 设置附件头  
    basename = os.path.basename(file_name)  
    file_msg.add_header('Content-Disposition','attachment', filename = basename)#修改邮件头  
    main_msg.attach(file_msg)  
      
    # 设置根容器属性  
    main_msg['From'] = From  
    main_msg['To'] = ", ".join(To)  
    main_msg['Subject'] = "xxxx "  
    main_msg['Date'] = email.Utils.formatdate( )  
      
    # 得到格式化后的完整文本  
    fullText = main_msg.as_string( )  
      
    # 用smtp发送邮件  
    try:  
    server.sendmail(From, To, fullText)  
    finally:  
    server.quit()  

封装成类

	    from multiprocessing import Process
	    from email.header import Header
	    from email.mime.text import MIMEText
	    from email.mime.multipart import MIMEMultipart
	    import smtplib
	    class MailSender(object):
	    def __init__(self):
	    self.mail_server = 'smtp.exmail.qq.com'
	    self.mail_ssl_port = 465
	    self.mail_form_user = 'admin@shunzi.me'
	    self.mail_passwd = '123456'
	    
	    def _send(self, title, content, to_address, s_filename):
	    msg = MIMEMultipart()
	    msg['From'] = self.mail_form_user
	    msg['To'] = ','.join(to_address)
	    msg['Subject'] = Header(title, "utf-8").encode()
	    text_msg = MIMEText(content)
	    msg.attach(text_msg)
	    att1 = MIMEText(open('/home/Project/'+s_filename, 'rb').read(), 'base64', 'gb2312')
	    att1["Content-Type"] = 'application/octet-stream'
	    att1["Content-Disposition"] = 'attachment; filename='+s_filename
	    msg.attach(att1)
	    server = smtplib.SMTP_SSL(self.mail_server, self.mail_ssl_port)
	    server.login(self.mail_form_user, self.mail_passwd)
	    server.sendmail(self.mail_form_user, to_address, msg.as_string(), s_filename)
	    server.quit()
	    
	    def send_email(self, title, content, to_address, s_filename):
	    p = Process(target=self._send, args=(title, content, to_address, s_filename))
	    p.start()
	    
	    if __name__ == '__main__':
	    m = MailSender()
	    m.send_email('邮件内容, ['xxx@qq.com'],'附件名')
