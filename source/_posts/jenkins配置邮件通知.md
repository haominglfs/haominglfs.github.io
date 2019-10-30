---
title: jenkins配置邮件通知
date: 2018-11-01 14:31:18
tags: jenkins
---
### jenkins系统配置

1. **设置jenkins地址和管理员邮箱地址**

   系统管理-->系统设置-->Jenkins Location

   ![](http://ow83fnk93.bkt.clouddn.com/20181101143431.png)

   <!--more-->

2. **设置发件人等信息**

   系统管理-->系统设置-->Extended E-mail Notification

   PS：这里的发件人邮箱地址切记要和系统管理员邮件地址保持一致（当然，也可以设置专门的发件人邮箱，不过不影响使用，根据具体情况设置即可）

   ![](http://ow83fnk93.bkt.clouddn.com/20181101143854.png)

3. 配置邮件模板default Content

   ```html
   <!DOCTYPE html>
   <html>
   <head>
   <meta charset="UTF-8">
   <title>${ENV, var="JOB_NAME"}-第${BUILD_NUMBER}次构建日志</title>
   </head>
   
   <body leftmargin="8" marginwidth="0" topmargin="8" marginheight="4"
       offset="0">
       <table width="95%" cellpadding="0" cellspacing="0"
           style="font-size: 11pt; font-family: Tahoma, Arial, Helvetica, sans-serif">
           <tr>
               <td>(本邮件是程序自动下发的，请勿回复！)</td>
           </tr>
           <tr>
               <td><h2>
                       <font color="#0000FF">构建结果 - ${BUILD_STATUS}</font>
                   </h2></td>
           </tr>
           <tr>
               <td><br />
               <b><font color="#0B610B">构建信息</font></b>
               <hr size="2" width="100%" align="center" /></td>
           </tr>
           <tr>
               <td>
                   <ul>
                       <li>项目名称&nbsp;：&nbsp;${PROJECT_NAME}</li>
                       <li>构建编号&nbsp;：&nbsp;第${BUILD_NUMBER}次构建</li>
                       <li>SVN&nbsp;版本：&nbsp;${SVN_REVISION}</li>
                       <li>触发原因：&nbsp;${CAUSE}</li>
                       <li>构建日志：&nbsp;<a href="${BUILD_URL}console">${BUILD_URL}console</a></li>
                       <li>构建&nbsp;&nbsp;Url&nbsp;：&nbsp;<a href="${BUILD_URL}">${BUILD_URL}</a></li>
                       <li>工作目录&nbsp;：&nbsp;<a href="${PROJECT_URL}ws">${PROJECT_URL}ws</a></li>
                       <li>项目&nbsp;&nbsp;Url&nbsp;：&nbsp;<a href="${PROJECT_URL}">${PROJECT_URL}</a></li>
                   </ul>
               </td>
           </tr>
           <tr>
               <td><b><font color="#0B610B">Changes Since Last
                           Successful Build:</font></b>
               <hr size="2" width="100%" align="center" /></td>
           </tr>
           <tr>
               <td>
                   <ul>
                       <li>历史变更记录 : <a href="${PROJECT_URL}changes">${PROJECT_URL}changes</a></li>
                   </ul> ${CHANGES_SINCE_LAST_SUCCESS,reverse=true, format="Changes for Build #%n:<br />%c<br />",showPaths=true,changesFormat="<pre>[%a]<br />%m</pre>",pathFormat="&nbsp;&nbsp;&nbsp;&nbsp;%p"}
               </td>
           </tr>
           <tr>
               <td><b>Failed Test Results</b>
               <hr size="2" width="100%" align="center" /></td>
           </tr>
           <tr>
               <td><pre
                       style="font-size: 11pt; font-family: Tahoma, Arial, Helvetica, sans-serif">$FAILED_TESTS</pre>
                   <br /></td>
           </tr>
           <tr>
               <td><b><font color="#0B610B">构建日志 (最后 100行):</font></b>
               <hr size="2" width="100%" align="center" /></td>
           </tr>
           <!-- <tr>
               <td>Test Logs (if test has ran): <a
                   href="${PROJECT_URL}ws/TestResult/archive_logs/Log-Build-${BUILD_NUMBER}.zip">${PROJECT_URL}/ws/TestResult/archive_logs/Log-Build-${BUILD_NUMBER}.zip</a>
                   <br />
               <br />
               </td>
           </tr> -->
           <tr>
               <td><textarea cols="80" rows="30" readonly="readonly"
                       style="font-family: Courier New">${BUILD_LOG, maxLines=100}</textarea>
               </td>
           </tr>
       </table>
   </body>
   </html>
   ```

4. 设置邮件触发机制

   ![](http://ow83fnk93.bkt.clouddn.com/20181101144430.png)

5. 上面的几步完成后，点击应用，保存即可。

### 项目配置

1. 进入项目配置页面

   ![](http://ow83fnk93.bkt.clouddn.com/20181101144658.png)

2. 进入系统配置页面后，点击上方的**构建后操作**选项，配置内容如下：

   ![](http://ow83fnk93.bkt.clouddn.com/20181101144826.png)

   ![](http://ow83fnk93.bkt.clouddn.com/20181101145324.png)

   ![](http://ow83fnk93.bkt.clouddn.com/20181101145623.png)

3. 构建项目测试

   ![](http://ow83fnk93.bkt.clouddn.com/20181101145815.png)