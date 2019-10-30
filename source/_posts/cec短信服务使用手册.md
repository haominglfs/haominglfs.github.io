---
title: cec短信服务使用手册
date: 2019-09-30 14:10:32
tags: 工作
---

1. 短信服务以hessian的方式对外提供，地址为：

   `http://10.0.1.55:8485/SendSmsService`

2. 对外提供的接口如下：

   ```java
   public interface SendSmsServiceInterface {
   	int sendSMS(String phoneNum, String smsContent);
   	int sendBatchSMS(String[] phoneNums, String smsContent);
   	int sendScheduleSMS(String phoneNum, String smsContent, String sendTime);
   	int sendBatchScheduleSMS(String[] phoneNums, String smsContent, String sendTime);
   	double getBalance();
   }
   ```

3. 示例

   ```java
   HessianProxyFactory factory = new HessianProxyFactory();
   SendSmsServiceInterface SendSmsService = (SendSmsServiceInterface)factory.create(SendSmsServiceInterface.class,"http://10.0.1.55:8485/SendSmsService");
   String status = "1";
   try {
   		status = Integer.toString(SendSmsService.sendBatchSMS(phoneNum,"msg test"));
   } catch (Exception ex) {
   		ex.printStackTrace();
   }
   ```

   

