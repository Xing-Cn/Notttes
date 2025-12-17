# EasyUPX+
*难的是UPX吗?难的就是UPX, TAT*

- 根据前车之鉴, 扔到UPX里脱了壳  
  ~~此时我还没有意识到这 `UPX+` 到底 **+** 在了哪里~~

- 看`main`, 看`EntryBuffer`, 看`readSelfExeBytecode`...
    - 发现有个魔改的rc4:
    ``` C++
    rc4_init(&rc4, key, keylen);
    for ( dn = 0; dn < datalen; ++dn )
    {
        i = (i + 1) % 256;
        j = (j + rc4.s_box[i]) % 256;
        tmp = rc4.s_box[i];
        rc4.s_box[i] = rc4.s_box[j];
        rc4.s_box[j] = tmp;
        t = (unsigned __int8)(rc4.s_box[i] + rc4.s_box[j]);

        // 标准 RC4 是 '^', 这里是 '+'
        data[dn] += rc4.s_box[t];
    }
    ``` 

    - `rc4_init`:
    ``` c++
    j = 0;
    if ( prc4 )
    {
        for ( i = 0; i <= 255; ++i )
        {
        // 独具一格的 -(char)i, 一般就是 i 
        prc4->s_box[i] = -(char)i;
        prc4->t_box[i] = key[i % keylen];
        }
        for ( ia = 0; ia <= 255; ++ia )
        {
        j = (prc4->s_box[ia] + j + prc4->t_box[ia]) % 256;
        tmp = prc4->s_box[ia];
        prc4->s_box[ia] = prc4->s_box[j];
        prc4->s_box[j] = tmp;
        }
    }
    ```
    > 说实话我以为一个魔改的rc4就已经更可怕了, ~~没想到~~
- 接下来找关键参数
    - 扣扣扣扣找到 `str`:
    ``` 
    unsigned __int8 Str[33]
    db 6Fh, 56h, 0C9h, 0E8h, 0DAh, 56h, 0F7h, 63h, 71h, 1Bh    
    db 48h, 44h, 0ECh, 0EDh, 2Ch, 0F6h, 19h, 0D7h, 0ACh, 7Ah
    db 5Ah, 5Eh, 8Fh, 0CDh, 8Fh, 70h, 0FAh, 0Dh, 0E4h, 0F3h
    db 3Eh, 60h, 0D0h
    ```

    - 又去找 `key`:
    ``` C++
    memcpy(key, "Real_Key", 8u);
    readSelfExeBytecode();
    rc4_crypt(data, datalen, key, 0xAu);
    ```
    > 一开始以为就Real_Key, 挺高兴  
    > 结果一看下面 0xAu, 长度为 10

    - 继续看`readSelfExeBytecode`:
    ``` C++
    // 在一个角落里找到了 key
    for ( i = 512; i <= 519; ++i )
        key[i - 512] = bytecode[i];
    key[4] = 46;
    ```
    > 坏人啊TAT, 痛苦从此开始...

    - 问了ai, 解释这解释那
    ``` python
    # 最后写了这么一个玩意去读 key, 结果怪怪的
    with open(r"C:\Users\a1381\Downloads\EasyUPX+\main_unpacked.exe", "rb") as f:
    f.seek(512)
    k0_7 = bytearray(f.read(8))

    k0_7[4] = 0x2E

    print("key[0..7] (hex):", " ".join(f"{b:02X}" for b in k0_7))
    print("key[0..7] (chr):", "".join(chr(b) if 32 <= b < 127 else '.' for b in k0_7))
    ```

    - 512 - 519, **不够位数**
    ```
    unsigned __int8 key[10]
    ```
    > 在`.bss`里找着这么一个玩意, 发现最后两位默认, 是 '0'

    - 当时我就想:
    > 嘿嘿, 找齐了, 草草写了个解密

    - 不对, 然后开始质疑:
    > key错了吧...这么奇怪

    - 动调, 也是这么一个值
    > 难不成是str?
    
    - 动调, 也是一样...

    - 然后没招了
    > 什么解密写错了, 什么读取时动手脚了, 什么是不是str长度不对... 
    > 想了个遍, **做不出来**

- 反正就是一大堆过程, 最后想到 : **是不是和 UPX 有关!** ~~早该想到的~~

    - 发现: 
    > 脱壳之后文件的第512位和脱壳前时**不一样**的!

    - 成功找到正确的key:
    ```
    key[0..7] (hex): 34 2E 32 34 2E 55 50 58
    key[0..7] (chr): 4.24.UPX
    ```

    - 编写正确的解密代码:
    ``` python 
    # .data 里的 Str（33 字节）
    cipher = bytes([
        0x6F, 0x56, 0xC9, 0xE8, 0xDA, 0x56, 0xF7, 0x63, 0x71, 0x1B,
        0x48, 0x44, 0xEC, 0xED, 0x2C, 0xF6, 0x19, 0xD7, 0xAC, 0x7A,
        0x5A, 0x5E, 0x8F, 0xCD, 0x8F, 0x70, 0xFA, 0x0D, 0xE4, 0xF3,
        0x3E, 0x60, 0xD0
    ])

    # 壳内真实 key（来自 main.exe 偏移 512..519）
    key = bytes([
        0x34, 0x2E, 0x32, 0x34, 0x2E, 0x55, 0x50, 0x58, 0x00, 0x00
    ])
    # b"4.24.UPX\x00\x00"

    def rc4_init_exact(key: bytes):
        s_box = [0] * 256
        t_box = [0] * 256
        keylen = len(key)

        # rc4_init 第一轮：s_box[i] = -(char)i; t_box[i] = key[i % keylen];
        for i in range(256):
            s_box[i] = (-i) & 0xFF
            t_box[i] = key[i % keylen]

        # rc4_init 第二轮：KSA
        j = 0
        for ia in range(256):
            j = (s_box[ia] + j + t_box[ia]) & 0xFF
            tmp = s_box[ia]
            s_box[ia] = s_box[j]
            s_box[j] = tmp

        return s_box

    def rc4_decrypt_add(cipher: bytes, key: bytes) -> bytes:
        s_box = rc4_init_exact(key)
        i = 0
        j = 0
        out = bytearray()

        for b in cipher:
            # PRGA（和题目里的 rc4_crypt 完全一致）
            i = (i + 1) & 0xFF
            j = (j + s_box[i]) & 0xFF

            tmp = s_box[i]
            s_box[i] = s_box[j]
            s_box[j] = tmp

            t = (s_box[i] + s_box[j]) & 0xFF
            k = s_box[t]

            # 加密：data[dn] += k
            # 解密：plain = (cipher - k) & 0xFF
            out.append((b - k) & 0xFF)

        return bytes(out)

    if __name__ == "__main__":
        plain = rc4_decrypt_add(cipher, key)
        print("解密结果(hex):", plain.hex())
        print("解密结果(str):", "".join(chr(c) if 32 <= c < 127 else '.' for c in plain))
    ```

*运行程序得到flag*  
总结 : 不容易哇😭