title: C++获取系统信息与拷贝文件到windows目录下
id: 1220
permalink: 1220
categories:
  - C++
date: 2014-12-06 22:28:33
tags:
---

```  cpp
#include <iostream>
#include <windows.h>
using namespace std;

void main(){
char szSelfName[MAX_PATH] = {0};//定义存储文件的名字的变量
char szWindowsPath[MAX_PATH] = {0};//定义winsows路径的变量
char szSystemPath[MAX_PATH] = {0};//定义存储system的变量
char szTmpPath[MAX_PATH] = {0};

GetModuleFileName(NULL,szSelfName,MAX_PATH);//得到文件的名字，NULL的时候是文件自身
GetWindowsDirectory(szWindowsPath,MAX_PATH);//得到windows目录
GetSystemDirectory(szSystemPath,MAX_PATH);//得到system目录

cout<<szSelfName<<endl<<szWindowsPath<<endl<<szSystemPath<<endl;
strcat(szWindowsPath,"\123.exe");//定义拷贝后的名字
cout<<szWindowsPath<<endl;
cout<<CopyFile(szSelfName,szWindowsPath,FALSE);

char szComputerName[MAXBYTE] = {0};//定义存储computer的变量
char szUserName[MAXBYTE] = {0};//定义userName存储的变量
unsigned long nSize = MAXBYTE;

OSVERSIONINFO OsVer;
//Before calling the GetVersionEx function, set the dwOSVersionInfoSize member of the OSVERSIONINFO data structure to sizeof(OSVERSIONINFO).
OsVer.dwOSVersionInfoSize = sizeof(OSVERSIONINFO);
GetVersionEx(&OsVer);//将得到的系统信息存储在OsVer中
cout<<OsVer.dwMajorVersion<<"."<<OsVer.dwMinorVersion<<" "<<OsVer.dwPlatformId<<endl;
if(OsVer.dwMajorVersion == 6 && OsVer.dwMinorVersion == 1){
cout<<"你的系统是win7"<<endl;
}else{
cout<<"你的系统不是win7"<<endl;
}
GetComputerName(szComputerName,&nSize);
GetUserName(szUserName,&nSize);
cout<<szComputerName<<endl;
cout<<szUserName<<endl;

```
<!-- more -->