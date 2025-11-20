---
title: Scripts
pubDate: 2024-10-18
categories: ['Wp']
description: '作者分享比赛中使用的多种模板化脚本，包括异或、大小写转化、RSA+MD5处理、z3求解、pwn计算、随机数处理、高低位组合、爆破程序及二次剩余+中国剩余定理等。这些脚本可节省大量时间，部分是从其他wp获取，需修改后使用。作者提到多解是正常的，需排除验证。'
slug:
---

最近打比赛，发现有很多脚本能够实现模板化处理，能够节省大量时间

下面将展示笔者收藏的一些常用脚本

## 简单异或

```c
#include <stdio.h>

int main() {
    int string_length = ; // 字符串长度
    int key = ;           // 密钥(需要异或的内容)

    // 定义v5数组
    unsigned char v5[string_length] = {
        // 密文
    };

    // 定义解密后的flag数组
    char flag[string_length + 1]; // 长度 + '\0' 结束符

    // 解密过程
    for (int i = 0; i < string_length; ++i) {
        flag[i] = v5[i] ^ key; // xor，这里的 key 也可以根据需要进行修改
    }
    flag[string_length] = '\0'; // 结束符

    // 输出解密后的结果
    printf("Flag: %s\n", flag);
    return 0;
}
```

## 带密钥的异或

```python
# 密文
v6 = []

# 密钥
key = b""
key_length = len(key)

# 逆向解密，得到 flag
flag = []
for i in range(len(v6)):
    for j in range(4):  # 每个 v6[i] 是 4 个字节
        flag_byte = (v6[i] >> (j * 8)) & 0xFF  # 提取 v6[i] 的每个字节
        key_byte = key[(i * 4 + j) % key_length]  # 密钥循环使用
        decrypted_byte = flag_byte ^ key_byte  # 逆向异或得到原始字节
        flag.append(decrypted_byte)

# 输出解密得到的 flag
print(bytes(flag).decode())
```

## 大小写转化

```python
from base64 import b64encode
from os import urandom

def Decrypt(msg, key):
    Lenth = len(key)
    result = ''

    upper_base = ord('A')
    lower_base = ord('a')
    KEY = [ord(key.upper()[_]) - upper_base for _ in range(Lenth)]

    index = 0
    for m in msg:
        tmp_key = KEY[index % Lenth]
        if not m.isalpha():
            result += m
            continue

        if m.isupper():
            result += chr(upper_base + (ord(m) - upper_base - tmp_key) % 26)
        else:
            result += chr(lower_base + (ord(m) - lower_base - tmp_key) % 26)
        index += 1
    return result

# 用你知道的key
key = "oWccl"  # 你需要知道加密时生成的密钥
encrypted_msg = "0lCcop{oyd94092-g8mq-4963-88b6-4helrxdhm6q7}"

# 解密
decrypted_flag = Decrypt(encrypted_msg, key)
print(decrypted_flag)
```

## 大小写转化+简单移位

```python
from base64 import b64encode
from os import urandom

def Decrypt(msg, key):
    Lenth = len(key)
    result = ''

    upper_base = ord('A')
    lower_base = ord('a')
    KEY = [ord(key.upper()[_]) - upper_base for _ in range(Lenth)]

    index = 0
    for m in msg:
        tmp_key = KEY[index % Lenth]
        if not m.isalpha():
            result += m
            continue

        if m.isupper():
            result += chr(upper_base + (ord(m) - upper_base - tmp_key) % 26)
        else:
            result += chr(lower_base + (ord(m) - lower_base - tmp_key) % 26)
        index += 1
    return result

# 用你知道的key
key = ""  # 加密时生成的密钥（如果你知道部分信息，你可以逆推出来）
encrypted_msg = ""  # 移位的密文

# 解密
decrypted_flag = Decrypt(encrypted_msg, key)
print(decrypted_flag)
```

## 简单的RSA+MD5处理flag

```python
from Crypto.Util.number import getPrime
from gmpy2 import invert, gcd
import hashlib

# 已知的公钥
N =
e =
# 使用 N 来尝试分解成 p 和 q
p=
q=

# 计算 phi——逆元
phi = (p - 1) * (q - 1)

# 计算私钥 d
d = invert(e, phi)

# 已知的密文 c
c =

# 解密
m_decrypted = pow(c, d, N)

md5 = hashlib.md5()
md5.update(str(m_decrypted).encode('utf-8'))

# 生成 flag
flag =md5.hexdigest()
print(flag)
```

## 使用z3求解

```python
from z3 import *

# 创建 Z3 Solver
solver = Solver()

# 创建变量，如果进行位操作的话，需要使用 BitVec 类型
v13 = BitVec('v13', 64)  # 64 位
v14 = BitVec('v14', 64)
v15 = BitVec('v15', 64)

# 添加条件
solver.add()
solver.add()
solver.add()

# 求解
if solver.check() == sat:
    model = solver.model()
    v13_val = model[v13].as_long()
    v14_val = model[v14].as_long()
    v15_val = model[v15].as_long()

    # 打印结果
    print(f"Found values: v13 = {v13_val:#x}, v14 = {v14_val:#x}, v15 = {v15_val:#x}")
```

## pwn计算的小脚本

不知道怎么修改，之间照搬过来吧，跑完就可以直接读取 flag 了

```python
from pwn import *
import string

context.log_level = 'debug'

p = remote('47.97.58.52', 40010) #环境地址，端口

def send_after_clean(content: bytes = b"", until: bytes = None,
                     timeout: float = 0.05, no_show: bool = True):
    if until is not None:
        p.recvuntil(flat(until))
    else:
        received = p.clean(timeout)
        if not no_show:
            print(f"[$]received:\n{received.decode('UTF-8')}")
    p.send(flat(content))

def sendline_after_clean(content: bytes = b"", until: bytes = None,
                         timeout: float = 0.05, no_show: bool = True):
    send_after_clean([content, p.newline], until, timeout, no_show)

def interactive_after_clean(timeout: int = 0.05, no_show: bool = True):
    received = p.clean(timeout)
    if not no_show:
        print(f"[$]received:\n{received}")
    p.interactive()

def formula_compute(formula: bytes, precise: bool = False):
    if isinstance(formula, bytes):
        formula = formula.decode("UTF-8")
    formula = formula.strip()
    formula = formula.strip("\n")
    formula = formula.replace("x", "*")
    formula = formula.replace("^", "**")
    formula = formula.replace("÷", "/")
    if not precise:
        formula = formula.replace("//", "/")
        formula = formula.replace("/", "//")
    return bytes(str(eval(formula)), encoding="UTF-8")

p.recvuntil(b'Welcome to the calc game!\n')  #p.sendlineafter(some_string, payload) 接收到 some_string 后, 发送你的 payload，加个换行
for i in range(100):
	p.recvline(keepends=True)
	#p.recvuntil(b' ',drop=True) #p.recvuntil(some_string) 接收到 some_string 为止
	s=p.recvuntil(b' =',drop=True)
	#p.recvall()
	print(s)
	res=str(eval(s)).encode()
	p.sendlineafter(b" ", res)
	p.recvline() #p.recvline() 接收一行输出

p.interactive() #交互模式
```

忘记哪个是真的能用的了，等下次跑出来再改吧

```python
# 自动生成头部
from pwn import *
from pwn import p64, p32, u32, u64, p8, p16

if_32: bool = False
if_debug: bool = False
pg = p32 if if_32 else p64
ug = u32 if if_32 else u64
context(log_level="debug", arch="i386" if if_32 else "amd64", os="linux")
p=remote("47.97.58.52",40010)

# 下面是自定义的一些工具函数

def send_after_clean(content: bytes = b"", until: bytes = None,
                     timeout: float = 0.05, no_show: bool = True):
    if until is not None:
        p.recvuntil(flat(until))
    else:
        received = p.clean(timeout)
        if not no_show:
            print(f"[$]received:\n{received.decode('UTF-8')}")
    p.send(flat(content))

def sendline_after_clean(content: bytes = b"", until: bytes = None,
                         timeout: float = 0.05, no_show: bool = True):
    send_after_clean([content, p.newline], until, timeout, no_show)

def interactive_after_clean(timeout: int = 0.05, no_show: bool = True):
    received = p.clean(timeout)
    if not no_show:
        print(f"[$]received:\n{received}")
    p.interactive()

def formula_compute(formula: bytes, precise: bool = False):
    if isinstance(formula, bytes):
        formula = formula.decode("UTF-8")
    formula = formula.strip()
    formula = formula.strip("\n")
    formula = formula.replace("x", "*")
    formula = formula.replace("^", "**")
    formula = formula.replace("÷", "/")
    if not precise:
        formula = formula.replace("//", "/")
        formula = formula.replace("/", "//")
    return bytes(str(eval(formula)), encoding="UTF-8")

for i in range(100):
    # 从标准输出流获取题目
    # 会一直读到b"."为止
    question = p.recvuntil(b".")
    # 工具函数自动计算结果
    # 注意，输入的题目必须是有意义的算式，不能有等号
    # 比如可以是1+2, 1÷2
    # 如果需要小数，定义参数precise=True
    answer = formula_compute(question)
    # 在结尾加上回车发送answer
    # 直到读到"answer:"才开始发送
    # 也可以不定义until字符串，脚本会在程序没有输出的时候发送（但如果网络不好，脚本可能判断出错）
    sendline_after_clean(answer, "answer:")

# 进入人机交互模式
interactive_after_clean()
```

## 随机数+奇偶分离

```cpp
#include <iostream>
#include <array>
#include <string>

int main() {
    // 加密后的数组，用逗号隔开
    std::array<uint64_t, 4> v13 = {
    };

    // 密钥
    uint8_t initialRandom = ;

    // 解密结果字符串
    std::string decrypted;

    // 解密过程
    for (size_t i = 0; i < 30; ++i) {
        uint8_t xorValue;

        // 奇偶选择XOR值
        if (i % 2 == 0) {
            xorValue = initialRandom + 3; // 偶数位置
        } else {
            xorValue = initialRandom; // 奇数位置
        }

        // 对应的加密值
        uint8_t encryptedChar = static_cast<uint8_t>(v13[i / 8] >> ((i % 8) * 8)); // 取得v13的当前字符

        // 解密字符
        uint8_t originalChar = encryptedChar ^ xorValue;

        // 将字符添加到解密字符串
        decrypted += static_cast<char>(originalChar);
    }

    // 输出解密结果
    std::cout << decrypted << std::endl;

    return 0;
}
```

## 高低位组合

```cpp
#include <stdio.h>

void decode(unsigned char *encoded_str, int length) {
    for (int i = 0; i < length; i++) {
        unsigned char high = (encoded_str[i] & 0xF0) >> 4;  // 提取高4位
        unsigned char low = (encoded_str[i] & 0x0F) << 4;   // 提取低4位
        encoded_str[i] = high | low;                        // 组合高位和低位
    }
}

int main() {
    // Encoded data
    unsigned char encdata[] = {

    };

    int length = sizeof(encdata) / sizeof(encdata[0]);

    // Decode the data
    decode(encdata, length);

    // Output the decoded string
    for (int i = 0; i < length; i++) {
        printf("%c", encdata[i]);
    }
    printf("\n");

    return 0;
}
```

## 爆破程序

```python
import itertools
import string
import subprocess

# 定义可执行文件的路径
exe_path = ""  # 替换为你程序的路径

# 固定前缀和后缀
prefix = ""
suffix = ""

# 字符集为数字和小写字母
charset = string.ascii_lowercase + string.digits

# 生成自定义输入
def generate_inputs():
    for combo1 in itertools.product(charset, repeat=4):  # 第一个 4 字节部分
        for combo2 in itertools.product(charset, repeat=4):  # 第二个 4 字节部分
            for combo3 in itertools.product(charset, repeat=4):  # 第三个 4 字节部分
                middle = ''.join(combo1) + '-' + ''.join(combo2) + '-' + ''.join(combo3)  # 连接三个部分
                yield prefix + middle + suffix

# 运行程序并传递生成的输入
def run_program_with_input(input_str):
    try:
        process = subprocess.Popen([exe_path], stdin=subprocess.PIPE, stdout=subprocess.PIPE, stderr=subprocess.PIPE)
        output, error = process.communicate(input=input_str.encode())
        if "Congratulations" in output.decode() or "Correct" in output.decode():  # 替换为程序的正确输出标志
            return True, output.decode()
        return False, output.decode()
    except Exception as e:
        return False, str(e)

# 使用生成器进行爆破
for generated_input in generate_inputs():
    print(f"Trying input: {generated_input}")
    success, result = run_program_with_input(generated_input)

    if success:
        print(f"Success! The correct input is: {generated_input}")
        print(f"Program output: {result}")
        break
    else:
        continue
        #print(f"Failed attempt with input: {generated_input}")
        #print(f"Program output: {result}")
```

## 二次剩余+中国剩余定理

2024-0xGame-w1 的 wp 里偷的，以后有需要直接扒过来改改就能用~

多解是正常的，跑久一点也能理解，四个解带进去排除一下就好了

```python
from gmpy2 import invert, gcd, gcdext
Pub_Key = (1022053332886345327, 294200073186305890)
Encrypt_msg = 107033510346108389
m = 759871216848924391
N = Pub_Key[0]
e = Pub_Key[1]
c = Encrypt_msg

#import q、p form factordb
q = 970868179
p = 1052721013
phi = (q-1)*(p-1)

def crt(b,m):
    #判断是否互素
    for i in range(len(m)):
        for j in range(i+1,len(m)):
            if gcd(m[i],m[j]) != 1:
                print("m中含有不是互余的数")
                return -1

    #乘积
    M = 1
    for i in range(len(m)):
        M *= m[i]
    #求M/mi
    Mm = []
    for i in range(len(m)):
        Mm.append(M // m[i])
    #求Mm[i]的乘法逆元
    Mm_ = []
    for i in range(len(m)):
        _,a,_ = gcdext(Mm[i],m[i])
        Mm_.append(int(a % m[i]))
    #求MiM'ibi的累加
    y = 0
    for i in range(len(m)):
        #print(Mm[i] * Mm_[i] * b[i])
        y += (Mm[i] * Mm_[i] * b[i])
    y = y % M
    return y

#直接在Zmod p、q下求m^2 ，也可以在Zmod N下求解m^2后，再⽤因⼦取余:
e0 = e//2
d1 = invert(e0,p-1)
m1 = pow(c,d1,p)
d2 = invert(e0,q-1)
m2 = pow(c,d2,q)

#直接⽤遍历开根：
def Gao(x,p):
    result = []
    for i in range(p):
        if pow(i,2,p) == x:
            result.append(i)
    return result

m1_ = Gao(m1,p)
m2_ = Gao(m2,q)

print(m1_,m2_)

from hashlib import md5
def MD5(m):return md5(str(m).encode()).hexdigest()

m1_ = [215896886, 836824127]
m2_ = [215973055, 754895124]

for m1 in m1_:
    for m2 in m2_:
        print('0xGame{' + MD5(crt([m1,m2],[p,q])) + '}')

'''
 0xGame{15820afdb9a129e89e40e57f40ff8de9}
 0xGame{f4107420d94cc7037114376d8566d4ef}
 0xGame{3932f6728585abbf751a212f69276d3e}
 0xGame{127016d0be858ef48a99723710ad4d49}
'''
```
