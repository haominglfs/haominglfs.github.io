---
title: cec邮件调用示例
date: 2019-09-30 15:26:07
tags: 工作
---

1. 调用邮件的服务为一个定时任务，定时扫描指定文件，若存在待发送的邮件，则以javaMail的方式调用邮件服务，调用示例如下(发件人邮箱SMTP服务器地址：mail.cec.com.cn)：

   <!--more-->
   
   ```java
   /**
    * JavaMail 版本: 1.6.0
    * JDK 版本: JDK 1.7 以上（必须）
    */
   public class sendMailService {
   	/**
   	 * 
   	 * @param myEmailAccount      发件人邮箱账号
   	 * @param myEmailPassword	     发件人邮箱密码
   	 * @param myEmailSMTPHost     发件人邮箱SMTP服务器地址
   	 * @param mainSendNameAccounts  收件人主送名称及账号
   	 * @param copySendNameAccounts   收件人抄送名称及账号
   	 * @param secretSendNameAccounts 收件人暗送名称及账号	
   	 * @param sendMsg      发送邮件主体信息
   	 * @return
   	 */
       public static String sendMail(String myEmailAccount, String myEmailPassword,String senderName,String myEmailSMTPHost,Map<String, String> mainSendNameAccounts,Map<String, String> copySendNameAccounts,Map<String, String> secretSendNameAccounts,Map sendMsg)throws Exception {
       	 Properties props = new Properties();                    // 参数配置
            props.setProperty("mail.transport.protocol", "smtp");   // 使用的协议（JavaMail规范要求）
            props.setProperty("mail.smtp.host", myEmailSMTPHost);   // 发件人的邮箱的 SMTP 服务器地址
            props.setProperty("mail.smtp.auth", "false");            // 需要请求认证
            // 2. 根据配置创建会话对象, 用于和邮件服务器交互
            Session session = Session.getInstance(props);
            session.setDebug(true); //开启日志
   		 // 3. 创建一封邮件
            MimeMessage message;
   		 message = createMimeMessage(session, myEmailAccount, senderName,mainSendNameAccounts,copySendNameAccounts,secretSendNameAccounts,sendMsg);
   		 // 4. 根据 Session 获取邮件传输对象
            Transport transport = session.getTransport();
            // 5. 使用 邮箱账号 和 密码 连接邮件服务器, 这里认证的邮箱必须与 message 中的发件人邮箱一致, 否则报错
            transport.connect(myEmailAccount, myEmailPassword);
            // 6. 发送邮件, 发到所有的收件地址, message.getAllRecipients() 获取到的是在创建邮件对象时添加的所有收件人, 抄送人, 密送人
            transport.sendMessage(message, message.getAllRecipients());
            transport.close();
            return Action.SUCCESS;
   	}
      
       /**
        * 创建邮件(带附件)
        *
        * @param session
        * @param myEmailAccount     发件人账号
        * @param mainSendAccount    主送人员账号
        * @param copySendAccount    抄送人员账号
        * @param secretSendAccount	  暗送人员账号
        * @param sendMsg            发送信息主体 
        * @return
        * @throws Exception
        */
       private static MimeMessage createMimeMessage(Session session, String myEmailAccount,String senderName, Map<String, String> mainSendNameAccounts ,Map<String, String> copySendNameAccounts,Map<String, String> secretSendNameAccounts,Map sendMsg) throws Exception {
       	IUser sUser = WebBaseUtil.getCurrentUser();
       	String subject = (String)sendMsg.get("subject");
       	String content = (String)sendMsg.get("content");
       	String attachPath = (String)sendMsg.get("attachPath");
      
           // 1. 创建一封邮件
           MimeMessage message = new MimeMessage(session);
           // 2. From: 发件人（昵称有广告嫌疑，避免被邮件服务器误认为是滥发广告以至返回失败，请修改昵称）
           message.setFrom(new InternetAddress(myEmailAccount,senderName, "UTF-8"));
           // 3. To: 收件人（可以增加多个收件人、抄送、密送）
           //设置主送人员
           Set mainSendNameSet = mainSendNameAccounts.keySet();//主送人员姓名
           List mailSendList = new ArrayList();
           for (Object mainSendName : mainSendNameSet) {
           	mailSendList.add(new InternetAddress(mainSendNameAccounts.get(mainSendName)));
   		}
           InternetAddress[] mainAddress =(InternetAddress[])mailSendList.toArray(new InternetAddress[mailSendList.size()]);
           message.setRecipients(MimeMessage.RecipientType.TO,mainAddress);//当邮件有多个收件人时，用逗号隔开
           //设置抄送人员
           if(!copySendNameAccounts.isEmpty()){
   	        Set copySendNameSet = copySendNameAccounts.keySet();//主送人员姓名
   	        List copylist = new ArrayList();
   	        for (Object copySendName : copySendNameSet) {
   	        	copylist.add(new InternetAddress(copySendNameAccounts.get(copySendName)));
   			}
   	        InternetAddress[] copyAddress =(InternetAddress[])copylist.toArray(new InternetAddress[copylist.size()]);
   	        message.setRecipients(MimeMessage.RecipientType.CC,copyAddress);//当邮件有多个收件人时，用逗号隔开
           }
           //设置暗送人员
           if(!secretSendNameAccounts.isEmpty()){
           	 Set secretSendNameSet = secretSendNameAccounts.keySet();//主送人员姓名
                List secretlist = new ArrayList();
                for (Object secretSendName : secretSendNameSet) {
                	secretlist.add(new InternetAddress(secretSendNameAccounts.get(secretSendName)));
        		}
                InternetAddress[] secretAddress =(InternetAddress[])secretlist.toArray(new InternetAddress[secretlist.size()]);
                message.setRecipients(MimeMessage.RecipientType.CC,secretAddress);//当邮件有多个收件人时，用逗号隔开
           }
           // 4. Subject: 邮件主题（标题有广告嫌疑，避免被邮件服务器误认为是滥发广告以至返回失败，请修改标题）
           message.setSubject(subject, "UTF-8");
           //5. 创建正文文本"节点"
           MimeBodyPart text = new MimeBodyPart();
           text.setContent(content,"text/html;charset=UTF-8");
           // 6. 创建附件"节点"
           MimeMultipart mm = new MimeMultipart();
           mm.addBodyPart(text);
           if(StringHelper.isNotEmpty(attachPath)){
           	MimeBodyPart attachment = new MimeBodyPart();
       	    DataHandler dh = new DataHandler(new FileDataSource(attachPath));
       	    attachment.setDataHandler(dh);
       	    attachment.setFileName(MimeUtility.encodeText(dh.getName())); 
       	    mm.addBodyPart(attachment);     // 如果有多个附件，可以创建多个多次添加
       	    mm.setSubType("mixed");         // 混合关系
           }
           message.setContent(mm);
           // 6. 设置发件时间
           message.setSentDate(new Date());
           // 7. 保存设置
           message.saveChanges();
           return message;
       }
       
}
   ```
   
   