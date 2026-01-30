# Hardxxtea
*动态调试得到正确参数*

- 先看到的是题目的描述:
> 这个程序很挑剔,只有在**动态调试**时才会正确地检查flag

- 接着观察`main`函数:
``` C++
// 初始化
...

pbDebuggerPresent[0] = 11;
pbDebuggerPresent[1] = 22;
pbDebuggerPresent[2] = 33;
n44 = 44;
hProcess = GetCurrentProcess();
// 题目所谓的动态调试在这里
CheckRemoteDebuggerPresent(hProcess, pbDebuggerPresent);
n44 = IsDebuggerPresent();

Buf1 = 0;
v21 = 0;
sub_7FF608C9169C(std::cin, v4, (__int64)&Buf1);
v5 = HIDWORD(v21);
// sum
v6 = 0;
// rounds
n12 = 12;
// xxtea加密
do {
    v6 -= 1640531527;
    p_Buf1 = (unsigned int *)&Buf1;
    v9 = (unsigned int *)&Buf1 + 1;
    v10 = (v6 >> 2) & 3;
    v11 = 0;
    n7 = 7;
    do {
      v13 = *v9++;
      v14 = v10 ^ v11++ & 3;
      *p_Buf1 += ((v6 ^ v13) + (pbDebuggerPresent[v14] ^ v5)) ^ (((16 * v5) ^ (v13 >> 3)) + ((v5 >> 5) ^ (4 * v13)));
      v5 = *p_Buf1++;
      --n7;
    } while ( n7 );
    v5 = (((v6 ^ Buf1) + (pbDebuggerPresent[v10 ^ 3] ^ v5))
        ^ (((16 * v5) ^ ((unsigned int)Buf1 >> 3)) + ((v5 >> 5) ^ (4 * Buf1))))
       + HIDWORD(v21);
    HIDWORD(v21) = v5;
    --n12;
} while ( n12 );
Buf2[0] = _mm_load_si128((const __m128i *)&xmmword_7FF608C93430);
Buf2[1] = _mm_load_si128((const __m128i *)&xmmword_7FF608C93420);
v15 = memcmp(&Buf1, Buf2, 0x20u);
p_wrong = "right";
if ( v15 )
    p_wrong = "wrong";
sub_7FF608C91428(std::cout, p_wrong);
return 0;
```

- 然后是动调找key:
    - pbDebuggerPresent[0] = 1  
    - n44 = 1  
> 也就是key = [1, 0x16, 0x21, 1]

- 由于认不出来这里到底有没有魔改, 被迫求助题解
    - 发现 **rounds = 12**
    - 其他等同标准xxtea

- 向deepseek问来标准xxtea解密的python实现: 
``` python
to1 = [0x0C, 0x62, 0xB1, 0xE4, 0x6B, 0x23, 0x74, 0xE6, 0x2A, 0xC5, 0x1C, 0x91, 0xE7, 0xE8, 0x48, 0xA3]
to2 = [0x5B, 0x23, 0x79, 0xF4, 0x18, 0x15, 0x56, 0xF7, 0xD1, 0x8F, 0x7A, 0x99, 0x54, 0x7A, 0xA0, 0x24]

from struct import unpack
# 等同于 C 里 'XXTEA_decrypt((uint32_t *)cipher, 8, key);'
to = list(unpack('<8I', bytes(to1 + to2)))

key = [0x01, 0x16, 0x21, 0x01]
Delta = 0x61C88647

def mx(y, z, sum_val, key, p, e):
    return ((z >> 5 ^ y << 2) + (y >> 3 ^ z << 4)) ^ ((sum_val ^ y) + (key[(p & 3) ^ e] ^ z))

def xxtea_decrypt(data, key):
    n = len(data)
    if n < 2:
        return data.copy()
    
    v = data.copy()
    # rounds 改成了 12
    rounds = 12
    sum_val = (rounds * -Delta) & 0xFFFFFFFF
    y = v[0]
    
    for _ in range(rounds):
        e = (sum_val >> 2) & 3
        
        for p in range(n - 1, 0, -1):
            z = v[p - 1]
            v[p] = (v[p] - mx(y, z, sum_val, key, p, e)) & 0xFFFFFFFF
            y = v[p]
        
        z = v[-1]
        v[0] = (v[0] - mx(y, z, sum_val, key, 0, e)) & 0xFFFFFFFF
        y = v[0]
        sum_val = (sum_val + Delta) & 0xFFFFFFFF
    
    return v

from struct import pack

res = b''.join(pack('<I', x) for x in xxtea_decrypt(to, key))
print(res.decode('ascii', errors='ignore'))
```

*运行程序得到flag*  
总结: tea这种东西真难看