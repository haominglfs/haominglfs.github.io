---
title: tomcat源码--启动脚本
date: 2019-02-03 11:43:22
tags: tomcat
---

#### startup.bat

```
@echo off
rem Licensed to the Apache Software Foundation (ASF) under one or more
rem contributor license agreements.  See the NOTICE file distributed with
rem this work for additional information regarding copyright ownership.
rem The ASF licenses this file to You under the Apache License, Version 2.0
rem (the "License"); you may not use this file except in compliance with
rem the License.  You may obtain a copy of the License at
rem
rem     http://www.apache.org/licenses/LICENSE-2.0
rem
rem Unless required by applicable law or agreed to in writing, software
rem distributed under the License is distributed on an "AS IS" BASIS,
rem WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
rem See the License for the specific language governing permissions and
rem limitations under the License.

rem ---------------------------------------------------------------------------
rem Start script for the CATALINA Server
rem ---------------------------------------------------------------------------
#临时修改系统变量
setlocal
#设置CATALINA_HOME环境变量
rem Guess CATALINA_HOME if not defined
set "CURRENT_DIR=%cd%"
if not "%CATALINA_HOME%" == "" goto gotHome
set "CATALINA_HOME=%CURRENT_DIR%"
if exist "%CATALINA_HOME%\bin\catalina.bat" goto okHome
cd ..
set "CATALINA_HOME=%cd%"
cd "%CURRENT_DIR%"
:gotHome
if exist "%CATALINA_HOME%\bin\catalina.bat" goto okHome
echo The CATALINA_HOME environment variable is not defined correctly
echo This environment variable is needed to run this program
goto end
:okHome
#验证CATALINA_HOME变量目录下的catalina.bat文件是否存在，不存在则批处理直接结束
set "EXECUTABLE=%CATALINA_HOME%\bin\catalina.bat"

rem Check that target executable exists
if exist "%EXECUTABLE%" goto okExec
echo Cannot find "%EXECUTABLE%"
echo This file is needed to run this program
goto end
:okExec

#如果设置了其他参数，将参数保存到 CMD_LINE_ARGS 变量中
rem Get remaining unshifted command line arguments and save them in the
set CMD_LINE_ARGS=
:setArgs
if ""%1""=="""" goto doneSetArgs
set CMD_LINE_ARGS=%CMD_LINE_ARGS% %1
shift
goto setArgs
:doneSetArgs
#执行 catalina.bat 批处理文件，注意，该行后面跟着两个参数，第1个参数是 start
call "%EXECUTABLE%" start %CMD_LINE_ARGS%

:end

```

#### catalina.bat

#### tomcat类加载设计

![](https://raw.githubusercontent.com/haominglfs/images/master/%E6%9C%AA%E5%91%BD%E5%90%8D%E6%96%87%E4%BB%B6.jpg)

整个Tomcat的classLoader分为了两条线，左边的一条线为catalinaLoader，这个是Tomcat服务器专用的，用于加载Tomcat服务器本身的class，右边的一条线则为web应用程序用的，每一个web应用程序都有自己专用的WebappClassLoader，用于加载属于自己应用程序的资源，例如/web-inf/lib下面的jar包，classes里面的class文件。