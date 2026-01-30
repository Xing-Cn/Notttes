# XXTEA 加密算法
*一点点特征*

### 基本概念

| 概念       | 说明                                                         |
|------------|--------------------------------------------------------------|
| 分组大小   | 任意长度, 由若干个 32-bit 整数构成(常见 2 ~ N 个 `uint32_t`) |
| 密钥大小   | 128 bit(16 字节), 通常视为 4×32-bit 的数组 `key[4]`         |
| 加密轮数   | 常见为 `6 + 52/n`(n 为块数), CTF 题中经常直接固定为 12/16 等 |
| 算法特点   | 是 TEA 的"扩展版", 支持可变长度分组, CTF/逆向里非常常见       |

典型 XXTEA 加密原型(标准版本):

```c
void XXTEA_encrypt(uint32_t* v, int n, uint32_t key[4]) {
    if (n < 2) return;
    uint32_t y, z, sum = 0, e;
    int p, q = 6 + 52 / n;          // 轮数，可被题目改成固定常数
    while (q-- > 0) {
        sum += 0x9E3779B9;          // DELTA
        e = (sum >> 2) & 3;
        for (p = 0; p < n - 1; p++) {
            y = v[p + 1];
            z = v[p];
            v[p] += (((z >> 5) ^ (y << 2)) +
                     ((y >> 3) ^ (z << 4))) ^
                    ((sum ^ y) + (key[(p & 3) ^ e] ^ z));
        }
        y = v[0];
        z = v[n - 1];
        v[n - 1] += (((z >> 5) ^ (y << 2)) +
                     ((y >> 3) ^ (z << 4))) ^
                    ((sum ^ y) + (key[(p & 3) ^ e] ^ z));
    }
}
```

> 典型特征:  
> - 常量 `0x9E3779B9`(DELTA);  
> - `sum` 按轮累加/递减;  
> - 关键访问 `key[(p & 3) ^ ((sum >> 2) & 3)]`;  
> - 核心混合公式 : `(((z >> 5) ^ (y << 2)) + ((y >> 3) ^ (z << 4))) ^ ((sum ^ y) + (key[...] ^ z))`

---

### 数据块与字节序

在 x86/x86_64 等**小端序**平台上, XXTEA 的输入/输出一般这样处理: 

- 原始字节串被分成 N 个 32-bit 整数:

  - 每 4 字节组成 1 个 `uint32_t`;
  - 小端，即"低地址是低字节"

- 128-bit 密钥通常为 16 个字节, 拆成 4 个 `uint32_t`:

  - `key[0], key[1], key[2], key[3]` 按内存从低到高依次读取

在 CTF 中，常见用法是:

```c
// C 里 : 把 32 字节密文当作 8 个 uint32_t
XXTEA_decrypt((uint32_t*)cipher, 8, key);
```

Python 对应写法:

```python
from struct import unpack
v = list(unpack('<8I', cipher))   # 小端拆成 8 个 32-bit
```

---

### 解密函数(与加密结构对称)

标准 XXTEA 解密（与上面的 XXTEA_encrypt 完全对应）

void XXTEA_decrypt(uint32_t *v, int n, uint32_t key[4]) {
    if (n < 2) return;
    uint32_t y, z, sum, e;
    int p, q = 6 + 52 / n;          // 轮数，要和加密时相同
    sum = q * 0x9E3779B9;           // DELTA * q
    while (q-- > 0) {
        e = (sum >> 2) & 3;
        // 先处理 v[n-1]..v[1]
        for (p = n - 1; p > 0; p--) {
            z = v[p - 1];
            y = v[p];
            v[p] -= (((z >> 5) ^ (y << 2)) +
                     ((y >> 3) ^ (z << 4))) ^
                    ((sum ^ y) + (key[(p & 3) ^ e] ^ z));
        }
        // 再处理 v[0]
        z = v[n - 1];
        y = v[0];
        v[0] -= (((z >> 5) ^ (y << 2)) +
                 ((y >> 3) ^ (z << 4))) ^
                ((sum ^ y) + (key[(p & 3) ^ e] ^ z));
        sum -= 0x9E3779B9;          // sum -= DELTA
    }
}

> 对比加密的差异：  
> - `sum` 初值为 `q * DELTA`，逐轮递减；  
> - 内层循环从 `p = n-1` 往回到 `1`，再处理 `p=0`；  
> - 对 `v[p]` 的更新用的是 `-=` 而不是 `+=`。

---

### XXTEA 逆向 / CTF 操作小结

```text
1. 在 bin 里找到 memcmp / strcmp / strncmp:
   → 通常是把 encrypt(input) 与某个常量比较

2. 找到加/解密函数:
   → 出现 DELTA=0x9E3779B9, sum 累加/递减, e = (sum>>2) & 3
     以及 key[(p & 3) ^ e], 基本可以认定是 XXTEA

3. 确定 key:
   → 查看运行时 key 的来源 : 可能来自反调试, 输入, 构造函数等
   → 像 Hardxxtea, 就是 [pb[0], pb[1], pb[2], n44]

4. 提取密文:
   → 找到比较用的常量缓冲(Buf2 等), 在 Hex View 中复制完整字节
   → 按小端 `<nI` 拆成 uint32 数组

5. 写脚本解密:
   → 用标准 XXTEA 解密函数, 对密文 v 解密
   → 得到明文, 一般就是 flag / 关键串
```