# Signal_Storm
*题目骗人*

- 拿到手先伪代码看`main`:
``` C++
// 精简版 main 逻辑
sigaction(11, ..., sub_1640);  // SIGSEGV
sigaction(8,  ..., sub_16E0);  // SIGFPE
sigaction(5,  ..., sub_1740);  // SIGTRAP
fgets(s, 100, stdin);           // 读取 32 字节 flag
sub_1780();                     // 初始化
n0x1F = 0;
do {
    if (!sigsetjmp) BUG();      // 触发 SIGSEGV
    if (!sigsetjmp) raise(5);   // 触发 SIGTRAP
    sigsetjmp(...);             // 隐式触发 SIGFPE
    ++n0x1F;
} while (n0x1F <= 0x1F);        // 循环 32 次
// 校验：加密后的 s 是否等于 4 个 64 位常量
```

> 陷阱出现了:  
>   - "下断点就能拿flag" -> 诱导下断点调试  
>   - 实际上 : 调试器会拦截信号, 导致信号处理函数不执行 -> 校验失败  
>   - "尊都假都" -> 假都!

- 然后去看`sub_1640`:
``` C++
// RC4 PRGA 核心
i = (i+1) % 256;
j = (j + S[i] + current_key[i % 21]) % 256;  // 关键：动态索引密钥
swap(S[i], S[j]);
longjmp → 回到循环
```
> - 作用 : 更新S盒状态(RC4伪随机生成)
> - 不是标准的RC4

- 往下是`sub_16E0`:
```c++
// 其实是 PRGA 的最后一步
if (n <= 0x63)  // 实际只处理前 32 字节
    s[n] ^= S[(S[i] + S[j]) % 256];  // 生成密钥流字节并异或
longjmp → 回到循环
```

- 接着是`sub_1740`:
```C++
// 循环密钥
memmove(key, key+1, 20);
key[20] = original_first_byte;  // 循环左移 1 字节
longjmp → 回到循环
```

- 在最底下还有`sub_1780`:
```C++
// 1. SIMD 循环：将 byte_4100 (S 盒) 初始化为 [0,1,2,...,255]
// 2. 标准 RC4 KSA：用固定密钥 "C0lm_be4ore_7he_st0rm" 混淆 S 盒
for i in 0..255:
    j = (j + S[i] + key[i%21]) % 256
    swap(S[i], S[j])
```

- 所以其实就一个魔改的RC4加密, 解密代码:
``` python
import struct

# ==================== 1. 目标密文(校验常量, 小端序拼接) ====================
targets = [
    0x8260C1C9C8D936E3,  # s[0:8]
    0x1C4BB2D52511D975,  # s[8:16]
    0xF11CAF1C716DE64D,  # s[16:24]
    0x1A5AF67F261CA506   # s[24:32]
]
target_bytes = b"".join(struct.pack("<Q", t) for t in targets)  # 32字节

# ==================== 2. S 盒初始化(sub_1780 全流程) ====================
S = list(range(256))  # SIMD 循环效果：S = [0,1,2,...,255]
key_bytes = b"C0lm_be4ore_7he_st0rm"  # 21字节固定密钥
j = 0
for i in range(256):  # 标准 RC4 KSA
    j = (j + S[i] + key_bytes[i % 21]) % 256
    S[i], S[j] = S[j], S[i]

# ==================== 3. 生成 32 字节 keystream(模拟信号风暴) ====================
i = j = 0
current_key = bytearray(key_bytes)  # 用于 sub_1640 的动态密钥
keystream = []

for n in range(32):
    # --- sub_1640 (SIGSEGV): 更新 S 盒 ---
    i = (i + 1) % 256
    # 关键修正：使用 idx_i % 21 动态索引 current_key(非 key[0]!)
    j = (j + S[i] + current_key[i % 21]) % 256
    S[i], S[j] = S[j], S[i]
    
    # --- sub_16E0 (SIGFPE): 生成密钥流字节 ---
    t = (S[i] + S[j]) % 256
    k = S[t]
    keystream.append(k)
    
    # --- sub_1740 (SIGTRAP): 密钥字符串循环左移(影响下一轮) ---
    current_key = current_key[1:] + current_key[:1]

# ==================== 4. 恢复原始 flag ====================
flag_bytes = bytes([target_bytes[i] ^ keystream[i] for i in range(32)])

# ==================== 5. 输出(ai大哥教我高水平输出) ====================
print("=" * 50)
print("🌪️  SIGNAL STORM FLAG RECOVERED 🌪️")
print("=" * 50)
print(f"Flag (hex)   : {flag_bytes.hex()}")
try:
    print(f"Flag (UTF-8) : {flag_bytes.decode('utf-8', errors='strict')}")
except:
    print(f"Flag (raw)   : {flag_bytes}")
    print("\n⚠️  Contains non-printable chars. Submit hex or raw bytes per challenge rules.")
print("=" * 50)
```

*运行程序得到flag*  
总结 : 信号题先静态分析