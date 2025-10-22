# IgniteMe
*仔细读代码解密*

- 拿到题, 先看文件类型: **32位Windows**的exe文件,扔进IDA看代码(一大堆东西, 直接找main)
    ```c++
    int __cdecl main(int argc, const char **argv, const char **envp) {
        void *v3; // eax
        void *v4; // eax
        void *v6; // eax
        void *v7; // eax
        size_t i; // [esp+4Ch] [ebp-8Ch]
        char v9[8]; // [esp+50h] [ebp-88h] BYREF
        char Str[128]; // [esp+58h] [ebp-80h] BYREF

        v3 = (void *)sub_402B30(&unk_446360, "Give me your flag:");
        sub_4013F0(v3, (int (__cdecl *)(void *))sub_403670);
        sub_401440(Str, 127);
        if ( strlen(Str) < 0x1E && strlen(Str) > 4 ) { // '0x1E' 就是 24, 这里说明flag的长度在 4 - 24 之间
            strcpy(v9, "EIS{");                        // flag 以 'EIS{' 开头
            for ( i = 0; i < strlen(v9); ++i ) {
                if ( Str[i] != v9[i] )
                    goto LABEL_7;
            }
            if ( Str[28] != 125 ) {                    // '125' 是 '}'
        LABEL_7:
                v6 = (void *)sub_402B30(&unk_446360, "Sorry, keep trying! ");
                sub_4013F0(v6, (int (__cdecl *)(void *))sub_403670);
                return 0;
            }
            if ( sub_4011C0(Str) )                     // 关键在这, 比较函数
                v7 = (void *)sub_402B30(&unk_446360, "Congratulations! ");
            else
                v7 = (void *)sub_402B30(&unk_446360, "Sorry, keep trying! ");
            sub_4013F0(v7, (int (__cdecl *)(void *))sub_403670);
            return 0;
        }
        else {
            v4 = (void *)sub_402B30(&unk_446360, "Sorry, keep trying!");
            sub_4013F0(v4, (int (__cdecl *)(void *))sub_403670);
            return 0;
        }
    }
    ```

- 还好不难读懂, 找到关键函数`sub_4011C0()`, 继续读代码
    ```c++
    bool __cdecl sub_4011C0(char *Str) {
        int v2; // [esp+50h] [ebp-B0h]
        char Str2[32]; // [esp+54h] [ebp-ACh] BYREF
        int v4; // [esp+74h] [ebp-8Ch]
        int v5; // [esp+78h] [ebp-88h]
        size_t i; // [esp+7Ch] [ebp-84h]
        char v7[128]; // [esp+80h] [ebp-80h] BYREF

        if ( strlen(Str) <= 4 )
            return 0;
        i = 4;
        v5 = 0;
        while ( i < strlen(Str) - 1 )
            v7[v5++] = Str[i++];
        v7[v5] = 0;
        v4 = 0;
        v2 = 0;
        memset(Str2, 0, sizeof(Str2));
        for ( i = 0; i < strlen(v7); ++i ) {                  // '32' 很显然是大小写转换
            if ( v7[i] >= 97 && v7[i] <= 122 ) {
                v7[i] -= 32;
                v2 = 1;
            }
            if ( !v2 && v7[i] >= 65 && v7[i] <= 90 )
                v7[i] += 32;
            Str2[i] = byte_4420B0[i] ^ sub_4013C0(v7[i]);     // 还有个异或运算
            v2 = 0;
        }
        return strcmp("GONDPHyGjPEKruv{{pj]X@rF", Str2) == 0; // 密钥在这里了
    }
    ```
    - byte_4420B0:
        > "0Dh, 13h, 17h, 11h, 2, 1, 20h, 1Dh, 0Ch, 2, 19h, 2Fh, 17h, 2Bh, 24h, 1Fh, 1Eh, 16h, 9, 0Fh, 15h, 27h, 13h, 26h, 0Ah, 2Fh, 1Eh, 1Ah, 2Dh, 0Ch, 22h, 4"  
    - 这里我学了个小妙招, 对着这个的地址按 '*', 就可以把多行压缩到一行直接复制出来了

- 这种题不用看题解我自己都能做出来, 先看`sub_4013C0()`
    ```c++
    int __cdecl sub_4013C0(int a1) {
        return (a1 ^ 0x55) + 72; // 最后一层加密了
    }
    ```

- 之前我学过了, 异或操作有**可逆性**, 即 `(A ^ B) ^ B = A`, 因此加密过程可逆,至此就全都明白了, 直接写解密程序:
    ```python
    key = "0Dh, 13h, 17h, 11h, 2, 1, 20h, 1Dh, 0Ch, 2, 19h, 2Fh, 17h, 2Bh, 24h, 1Fh, 1Eh, 16h, 9, 0Fh, 15h, 27h, 13h, 26h, 0Ah, 2Fh, 1Eh, 1Ah, 2Dh, 0Ch, 22h, 4"
    
    # 去掉 key 里的 'h'
    def change(key):
        lst = key.split(", ")
        result = []
        for i in lst:
            if i[-1] == 'h':
                result.append(i[: -1])
            else:
                result.append(i)
        return result
        
    rlkey = change(key)
    target = "GONDPHyGjPEKruv{{pj]X@rF"

    # 解密主体
    def decrypt():
        result = []
        for i in range(len(target)):
            # 得 v7 
            anss = ord(target[i]) ^ int(rlkey[i], 16)
            ori = (anss - 72) ^ 0x55
            # 大小写转换回来
            if chr(ori).isupper(): 
                result.append(chr(ori + 32))
            elif chr(ori).islower():
                result.append(chr(ori - 32))
            else:
                result.append(chr(ori))
            ans = "".join(result)
        return ans
            
    print("EIS{" + decrypt() + "}")
    ```

*运行程序得到flag*  
总结: 
- 这题主要是积累经验, **更加熟练运用IDA**🎉  
- 除此以外, 我发现行内代码只要两个 '`' 就够了 ~~刚开始明明用对的~~ 😒