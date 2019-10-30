---
title: websocket使用总结
date: 2019-09-05 18:12:10
tags:
---

### 背景

公司门户系统有一个显示待办消息的需求，要求其他系统产生的待办消息要及时的在门户系统中展示，网上查找了解到有ajax轮询和websocket两种主要方式，为了及时性，最终选择了websocket方式。

<!--more-->

### 整体思路

一图胜千言

![](https://raw.githubusercontent.com/haominglfs/images/master/20190905181723.png)

### 代码

* 前端

  ```js
  <!-- websocket-->
      <script type="text/javascript">
          var msgTypes = {
              1:'db',
              2:'dy',
              3:'yj',
              4:'gwdb'
          }
          userId = '<ww:property value="#session.sUser.userId" />';
          //定义websocket
          var ws = new WebSocket("ws://localhost:8888/msg/ws");
          //维持心跳
          var heartCheck = {
              timeout: 19000,//19s
              timeoutObj: null,
              reset: function(){
                  clearInterval(this.timeoutObj);
                  this.start();
              },
              start: function(){
                  this.timeoutObj = setInterval(function(){
                      if(ws.readyState==1){
                          ws.send("HeartBeat");
                      }
                  }, this.timeout)
              }
          };
          //连接websocket
          ws.onopen = function()
          {
              var msg = 'userid='+userId;
              console.log("open and send message："+msg);
              // 发送消息
              ws.send(msg);
              heartCheck.start();
          };
          //接收消息
          ws.onmessage = function(evt)
          {
              console.log("get message:"+evt.data);
              heartCheck.reset();
              if(evt.data.startsWith('您')){//提示消息
                  $("#msg_tip").text(evt.data);
                  console.log('显示tip')
                  $("#msg_tip").fadeIn();
              }else{
                  var typeId = 'span#'+msgTypes[evt.data.split(':')[0]];
                  $(typeId).html(evt.data.split(':')[1]);
                  if($navTab.getTab('dirMsg')!= ''){
                      debugger;
                      //$navTab.refreshCurrentTabForm('dirMsg');
                      $forms = $("#dirMsg").find('.table-form');
                      for(var i=0;i<$forms.length;i++){
                          $navTab.submitForm($forms[i]);
                      }
                  }
              }
          };
          //关闭连接
          ws.onclose = function(evt)
          {
              console.log("WebSocketClosed!");
          };
          //发生异常
          ws.onerror = function(evt)
          {
              console.log("WebSocketError!");
              ws = null;
          };
          window.onunload = function(event){
              if(ws){
                  ws=null;
              }
          };
  ```

* 后端（tomcat8)

  ```java
  package com.css.apps.msg.websocket;
  
  /**
   * @Author: haoming
   * @Date: 2019/5/8 11:22 PM
   * @Version 1.0
   */
  
  import com.css.apps.msg.constant.MsgType;
  import com.css.db.query.QueryCache;
  import org.apache.commons.logging.Log;
  import org.apache.commons.logging.LogFactory;
  
  import javax.websocket.*;
  import javax.websocket.server.ServerEndpoint;
  import java.io.IOException;
  import java.util.HashMap;
  import java.util.Map;
  
  /**
   * @Author: haoming
   * @Date: 2019/5/6 7:07 PM
   * @Version 1.0
   */
  @ServerEndpoint(value = "/ws")
  public class WsServlet {
      private static Log log = LogFactory.getLog(WsServlet.class);
      //设置Map,存放每个用户的连接
      public static Map<String,WsServlet> webSocketSet = new HashMap<String,WsServlet>();
      //浏览器与服务端的回话，浏览器每new一个WebSocket就创建一个session，关闭或刷新浏览器，session关闭
      private Session session;
      //代表浏览器
      private String userid;
      /**
       * 推送消息接口
       * 外部可以进行调用
       * @param sendMsg
       * @throws IOException
       */
      public void sendMsg(String sendMsg) throws IOException {
          log.info(this.userid+"发送消息:"+sendMsg);
          this.session.getBasicRemote().sendText(sendMsg);
      }
  
      @OnOpen
      public void onOpen(Session session) throws IOException {
          this.session = session;
          log.info(this+"有新连接,session="+session+";userid="+userid);
      }
  
      @OnClose
      public void onClose() {
          webSocketSet.remove(this.userid);
          log.info(this.userid+"；连接关闭");
      }
  
      @OnMessage
      public void onMessage(String info) throws IOException {
          log.info(this.userid+"；来自客户端的消息:" + info);
          String msg = "服务端接收到了来自客户端的消息："+info;
          if(info.contains("userid")){
              this.userid = info.split("userid=")[1];
              log.info("this.userid="+this.userid);
              webSocketSet.put(userid, this);
              //发送初始待办数量
              sendMsg("1:"+getMsgNum(userid, MsgType.DAIBAN));
              sendMsg("2:"+getMsgNum(userid, MsgType.DAIYUE));
              sendMsg("3:"+getMsgNum(userid, MsgType.YOUJIAN));
              sendMsg("4:"+getMsgNum(userid, MsgType.GWDAIBAN));
          }
      }
  
      @OnError
      public void onError(Throwable error) {
          log.error(this.userid+"；发生错误",error);
      }
  
      private String getMsgNum(String userId,Integer msgType){
          String sql = "";
          sql = "select count(a.uuid) from Msg a where a.receiver =:userId " +
                  "and a.msgType =:msgType and a.msgStatus = 1";
          QueryCache qc = new QueryCache(sql);
          qc.setParameter("userId",userId);
              qc.setParameter("msgType",msgType);
          return qc.uniqueResult().toString();
      }
  }
  ```

* 总结(问题)

  1. 前端添加心跳机制防止连接超时断开，但发送心跳的时间网络上未找到具体的应该根据什么来设置？。

  2. 利用如下代码测试最大连接数

     ```js
     for (var i = 0; i < 300; i++) {
             sleep(5)
             websocket();
         }
     
         function websocket() {
             var wsUri = "ws://localhost:8888/msg/ws";
             var ws = new WebSocket(wsUri);
             ws.onopen = function () {
                 ws.send("User connected");
             };
             ws.onmessage = function (e) {
                 console.log(e.data)
             };
             ws.onclose = function () {
                 console.log("User disconnected");
             };
         }
     
         function sleep(n) {
             var start = new Date().getTime();
             while (true)
                 if (new Date().getTime() - start > n) break;
         }
     ```

     发现浏览器最大连接数达到200时，其他连接无法连上，修改tomcat的连接数等配置，会有小幅度增加，达到了256，但不理想?，tomcat配置如下：

     ```xml
     <Executor maxThreads="500" minSpareThreads="20" name="tomcatThreadPool" namePrefix="catalina-exec-"/>
         <Connector URIEncoding="UTF-8" acceptCount="300" connectionTimeout="20000" enableLookups="false" executor="tomcatThreadPool" port="8080" protocol="org.apache.coyote.http11.Http11NioProtocol" redirectPort="8443"/>
     
     ```

     