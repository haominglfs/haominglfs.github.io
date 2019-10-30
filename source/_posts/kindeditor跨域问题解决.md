---
title: kindeditor跨域问题解决
date: 2019-09-05 17:38:25
tags:
---

### 问题背景

因为在A系统中需要嵌入B系统的页面，两个系统有相同的父域名，所以使用了iframe+domain的方式解决跨域问题，在A系统的页面中加入了`document.domain = 'exame.com.cn';`;但是在嵌入kindeditor富文本编辑器后，上传图片出现跨域问题。

<!--more-->

### 解决思路

通过分析kindeditor的图片上传代码，如下

```html
<div class="tab2" style=""><iframe name="kindeditor_upload_iframe_1567676701755" style="display:none;"></iframe>
        <form class="ke-upload-area ke-form" method="post" enctype="multipart/form-data"
            target="kindeditor_upload_iframe_1567676701755" action="/portal/kindeditor/uploadImg?dir=image">
            <div class="ke-dialog-row"><label style="width:60px;">上传文件</label><input type="text" name="localUrl"
                    class="ke-input-text" tabindex="-1" style="width:200px;" readonly="true"> &nbsp;<div
                    class="ke-inline-block ke-upload-button">
                    <div class="ke-upload-area"><span class="ke-button-common"><input type="button"
                                class="ke-button-common ke-button" value="浏览..."></span><input type="file"
                            class="ke-upload-file" name="uploadFile" style="width: 60px;"></div>
                </div><input type="button" class="ke-upload-button" value="浏览..." style="display: none;"></div>
        </form>
    </div>
```

上传后的返回值会放入`iframe`中，所以只要在iframe中加入`document.domain = 'exame.com.cn';`就能解决跨域问题。

### 解决方法

参照kindeditor包中的uplod_json.jsp，重新写了一个UploadImgServlet，最终代码如下

* 前端代码(正常写，不需要特殊处理)

  ```javascript
  editor = KindEditor.create('textarea[rel="ehContentOnlyOne"]',{
  			resizeType : 1,
  			allowPreviewEmoticons : false,
  			allowImageUpload : true,//上传图片框本地上传的功能，false为隐藏，默认为true
        allowImageRemote : false,//上传图片框网络图片的功能，false为隐藏，默认为true
  			cssPath:['cssui/main/editor.css'],
              filePostName: "uploadFile",
              uploadJson : '/portal/kindeditor/uploadImg',// 上传图片接口
  			items : [
  				'source','preview','code','|','fontname', 'fontsize', '|', 'forecolor', 'hilitecolor', 'bold', 'italic', 'underline',
  				'removeformat', '|', 'justifyleft', 'justifycenter', 'justifyright', 'insertorderedlist',
  				'insertunorderedlist', '|', 'emoticons','image', 'link']
  		});
  ```

  

* 后端代码

  ```java
  package com.css.bbs.bbs.action;
  
  import net.sf.json.JSONObject;
  import org.apache.commons.fileupload.FileItem;
  import org.apache.commons.fileupload.FileItemFactory;
  import org.apache.commons.fileupload.FileUploadException;
  import org.apache.commons.fileupload.disk.DiskFileItemFactory;
  import org.apache.commons.fileupload.servlet.ServletFileUpload;
  
  import javax.servlet.ServletException;
  import javax.servlet.http.HttpServlet;
  import javax.servlet.http.HttpServletRequest;
  import javax.servlet.http.HttpServletResponse;
  import java.io.File;
  import java.io.IOException;
  import java.io.PrintWriter;
  import java.text.SimpleDateFormat;
  import java.util.*;
  
  /**
   * @Author: haoming
   * @Date: 2019/9/3 10:36 AM
   * @Version 1.0
   */
  public class UploadImgServlet extends HttpServlet {
      protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
          //设置Response响应的编码
          response.setContentType("text/html; charset=UTF-8");
          PrintWriter out = response.getWriter();
          //文件保存目录路径
          String savePath = request.getServletContext().getRealPath("/") + "attached/";
          //文件保存目录URL
          String saveUrl  = request.getContextPath() + "/attached/";
          //定义允许上传的文件扩展名
          HashMap<String, String> extMap = new HashMap<String, String>();
          extMap.put("image", "gif,jpg,jpeg,png,bmp");
          extMap.put("flash", "swf,flv");
          extMap.put("media", "swf,flv,mp3,wav,wma,wmv,mid,avi,mpg,asf,rm,rmvb");
          extMap.put("file", "doc,docx,xls,xlsx,ppt,htm,html,txt,zip,rar,gz,bz2");
          //最大文件大小
          long maxSize = 1000000;
          if(!ServletFileUpload.isMultipartContent(request)){
              out.println(getError("请选择文件。"));
              return;
          }
          //检查目录
          File uploadDir = new File(savePath);
          if(!uploadDir.isDirectory()){
              uploadDir.mkdirs();
          }
          //检查目录写权限
          if(!uploadDir.canWrite()){
              out.println(getError("上传目录没有写权限。"));
              return;
          }
  
          String dirName = request.getParameter("dir");
          if (dirName == null) {
              dirName = "image";
          }
          if(!extMap.containsKey(dirName)){
              out.println(getError("目录名不正确。"));
              return;
          }
          //创建文件夹
          savePath += dirName + "/";
          saveUrl += dirName + "/";
          File saveDirFile = new File(savePath);
          if (!saveDirFile.exists()) {
              saveDirFile.mkdirs();
          }
          SimpleDateFormat sdf = new SimpleDateFormat("yyyyMMdd");
          String ymd = sdf.format(new Date());
          savePath += ymd + "/";
          saveUrl += ymd + "/";
          File dirFile = new File(savePath);
          if (!dirFile.exists()) {
              dirFile.mkdirs();
          }
  
          FileItemFactory factory = new DiskFileItemFactory();
          ServletFileUpload upload = new ServletFileUpload(factory);
          upload.setHeaderEncoding("UTF-8");
          List items = null;
          try {
              items = upload.parseRequest(request);
          } catch (FileUploadException e) {
              out.println(getError("上传文件失败。"));
              return;
          }
          Iterator itr = items.iterator();
          while (itr.hasNext()) {
              FileItem item = (FileItem) itr.next();
              String fileName = item.getName();
              long fileSize = item.getSize();
              if (!item.isFormField()) {
                  //检查文件大小
                  if (item.getSize() > maxSize) {
                      out.println(getError("上传文件大小超过限制。"));
                      return;
                  }
                  //检查扩展名
                  String fileExt = fileName.substring(fileName.lastIndexOf(".") + 1).toLowerCase();
                  if (!Arrays.<String>asList(extMap.get(dirName).split(",")).contains(fileExt)) {
                      out.println(getError("上传文件扩展名是不允许的扩展名。\n只允许" + extMap.get(dirName) + "格式。"));
                      return;
                  }
  
                  SimpleDateFormat df = new SimpleDateFormat("yyyyMMddHHmmss");
                  String newFileName = df.format(new Date()) + "_" + new Random().nextInt(1000) + "." + fileExt;
                  try {
                      File uploadedFile = new File(savePath, newFileName);
                      item.write(uploadedFile);
                  } catch (Exception e) {
                      out.println(getError("上传文件失败。"));
                      return;
                  }
  
                  JSONObject obj = new JSONObject();
                  obj.put("error", 0);
                  obj.put("url", saveUrl + newFileName);
                	//解决跨域问题
                  out.println(obj.toString()+"<script>document.domain='cec.com.cn';</script>");
              }
          }
          //将writer对象中的内容输出
          out.flush();
          //关闭writer对象
          out.close();
      }
  
      protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
          doPost(request,response);
      }
  
      private String getError(String message) {
          JSONObject obj = new JSONObject();
          obj.put("error", 1);
          obj.put("message", message);
          return obj.toString();
      }
  }
  
  ```

* web.xml配置

  ```xml
  <servlet>
    <servlet-name>uploadImg</servlet-name>
    <servlet-class>com.css.bbs.bbs.action.UploadImgServlet</servlet-class>
  </servlet>
  
  <servlet-mapping>
    <servlet-name>uploadImg</servlet-name>
    <url-pattern>/kindeditor/uploadImg</url-pattern>
  </servlet-mapping>
  ```

  