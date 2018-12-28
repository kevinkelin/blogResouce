title: C++中windows编程基础
id: 1216
permalink: 1216
categories:
  - C++
date: 2014-12-06 22:20:08
tags:
---

```  cpp
一、宽字符与多字节字符
#include "tchar.h"
void main(){
TCHAR p[] = _T("IT学吧");
int l1 = sizeof(p);//I1 T1 学2 吧2 1
int l2 = _tcslen(p);
int l4 = wcslen(p);
//int l3 = strlen(p);
return;

}
```

在多字节字符集中，每个汉字占两个字节，英文字母占一个字节，sizeof(p) = 7 _tcslen(p)=6(字符串长度等于6，I1 T1 学2 吧2)

在宽字节（unicode）字符集中，所有有字符都是占两个字节 sizeof(p) = 10 _tcslen(p)=4(I1 T1 学1 吧1)
<!-- more -->
在_tcslen函数中 英文字母每一个都只占一个长度，多字节中，中文占两个，宽字节中，中文占一个

#define _tcslen&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; wcslen&#160; （宽字节定义 unicode）

#define _tcslen&#160;&#160;&#160;&#160; strlen （多字节定义）

总体来说 使用_tcslen可以兼容多字节与宽字节

一般由_t与_tcs开头的都是兼容的

```  cpp
// ca02windows.cpp : 定义应用程序的入口点。
//

#include "stdafx.h"
#include "ca02windows.h"
#include "tchar.h"
#include <Windows.h>

LRESULT CALLBACK WndProc(HWND hwnd,UINT uMsg,WPARAM wParam,LPARAM lParam){
switch (uMsg)
{
case WM_CLOSE:
::MessageBox(hwnd, _T("确认关闭吗？"), _T("确认关闭吗？"), MB_YESNO);

::DestroyWindow(hwnd);
break;
case WM_DESTROY:
::PostQuitMessage(0);
break;
default:
break;
}
return DefWindowProc(hwnd, uMsg, wParam, lParam);//不自已定义处理函数，使用系统自已的处理方法

}

int WINAPI _tWinMain(IN HINSTANCE hInstance,IN HINSTANCE hPrevInstance,IN LPTSTR lpCmdLine,IN int nShowCmd)
{
const TCHAR* pszClassName = _T("ITWnd");
WNDCLASSEX wcex;
wcex.cbSize = sizeof(WNDCLASSEX);
wcex.style = CS_HREDRAW | CS_VREDRAW;//水平与垂直变化重绘
wcex.lpfnWndProc = WndProc;//回调函数
wcex.cbClsExtra = 0;
wcex.cbWndExtra = 0;
wcex.hInstance = hInstance;//handle
wcex.hIcon = (HICON)::LoadIcon(NULL, IDI_HAND);//大图标
wcex.hIconSm = (HICON)::LoadIcon(NULL, IDI_APPLICATION);//小图标
wcex.hbrBackground = (HBRUSH)::GetStockObject(WHITE_BRUSH);//背景
wcex.hCursor = (HCURSOR)::LoadCursor(NULL, IDC_ARROW);//光标
wcex.lpszMenuName = NULL;//是否有菜单
wcex.lpszClassName = pszClassName;//注册的类名，不能重复的

BOOL bRet = ::RegisterClassEx(&wcex);//开始注册这个类
if (!bRet){
::MessageBox(NULL, _T("注册窗口失败"), _T("注册窗口"), MB_OK);
return FALSE;
}

HWND hwnd = ::CreateWindowEx(0, pszClassName, _T("杨彦星kevin"), WS_VISIBLE | WS_OVERLAPPEDWINDOW,
CW_USEDEFAULT, CW_USEDEFAULT, CW_USEDEFAULT, CW_USEDEFAULT,
NULL, NULL, hInstance, NULL); //开始创建窗口

if (!hwnd){
::MessageBox(NULL, _T("创建窗口失败"), _T("创建窗口"), MB_OK);
return FALSE;

}

::ShowWindow(hwnd, SW_NORMAL);//创建窗口 SW_NORMAL最大化
::UpdateWindow(hwnd);//更新窗口

//::DestroyWindow()

MSG msg;
while (::GetMessage(&msg,NULL,NULL,NULL))
{
::TranslateMessage(&msg);//翻译消息
::DispatchMessage(&msg);//调试消息，将消息传给回调函数进行处理

}

return TRUE;

}

```
