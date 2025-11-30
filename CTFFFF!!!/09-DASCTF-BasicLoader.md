# BasicLoader
*相当漫长的一题*

- 和之前做过的题一样, IDA:
    - `main`里找到一个`sub_140001140`
        ``` C++
        if ( 
            *a1 != 68           //"D"
            || a1[1] != 65      //"A"
            || a1[2] != 83      //"S"
            || a1[3] != 67      //"C"
            || a1[4] != 84      //"T"
            || a1[5] != 70      //"F"
            || a1[6] != 123     //"{"
            || a1[39] != 125    //"}", 长度为40
        ) {
            return 0;
        }
        ```

    - 接下来想去看`main`里的`StartAddress`, 显示**太大了**看不了  
    按`X`查看还有哪里提到了`StartAddress`, 找到`TlsCallback_0`函数
        > **TLS回调函数**(Thread Local Storage Callback Function)是一种在程序启动时  
        > 特别是在主线程的入口点(Entry Point)代码执行之前被调用的函数  
        > 这种机制允许开发者在程序的主逻辑开始**之前**执行特定的代码  
        > 常用于初始化操作或**反调试技术**
        ``` C++
        if ( a2 == 1 ) {
            if ( (unsigned int)sub_140001000() ) {  //反调试
                ...
                ExitProcess(0);
            }
            // 解壳：把真正线程代码拷到 StartAddress
            VirtualProtect(StartAddress, 0x26F0u, 0x40u, &flOldProtect);
            qmemcpy(StartAddress, &src_, 0x26F0u);
        }
        ```
    - 双击查看`&src_`, 按`P`建立函数, 发现:
        ``` C++
        __int64 __fastcall src_(__int64 a1) {
            return sub_140032EC0(a1);
        }
        ```
    - 继续查看`sub_140032EC0`:
        ``` C++
        // 进入第二阶段了(yeah
        strcpy(Shellcode_loaded_, "Shellcode loaded");
        strcpy(FLAG_IS_WHAT_U_INPUT__, "FLAG IS WHAT U INPUT!");
        strcpy(FAILED_, "FAILED");
        strcpy(Checking_flag_, "Checking flag");
        strcpy(WELCOM2STEP2_, "WELCOM2STEP2");

        // 各种初始化...
        ...
        // 定义了一大长串v9, 从v9[0]到v9[147]
        ...

        qmemcpy(v10, "UXJO", 4);

        // 一堆神奇处理
        ...

        // key
        qmemcpy(v5, "babyflag", sizeof(v5));
        // 长度为152, 查看汇编, 发现挨着的是v10
        sub_140031570(v9, 152, v5, 8);

        v14(sub_140032620, 153, 64, &v11);
        // 把 v9[0..151] 拷贝成代码    
        for ( n152 = 0; n152 < 152; ++n152 )
            *((_BYTE *)sub_140032620 + n152) = v9[n152];

        // 调用这段动态生成的代码，参数是 a1（32 字节）
        if ( (unsigned __int8)sub_140032620(a1) )
            return ((__int64 (__fastcall *)(_QWORD, char *, char *, _QWORD))v13)(
                    0,
                    FLAG_IS_WHAT_U_INPUT__,
                    Shellcode_loaded_,
                    0);
        else
            return ((__int64 (__fastcall *)(_QWORD, char *, char *, _QWORD))v13)(0, FAILED_, Shellcode_loaded_, 0);
        ```

- 接下来就是想办法得到这个v9生成的代码
    - 查看`sub_140031570`
        ``` C++
        // 初始化
        for ( n256 = 0; n256 < 256; ++n256 )
        {
            v13[n256] = n256;
            v15[n256] = *(_BYTE *)(a3 + n256 % 8);
        }

        // KSA 密钥拓展 : 用 key 混淆 S 盒
        for ( n256_1 = 0; n256_1 < 256; ++n256_1 )
        {
            v10 = ((unsigned __int8)v15[n256_1] + (unsigned __int8)v13[n256_1] + v10) % 256;
            v8 = v13[n256_1];
            v13[n256_1] = v13[v10];
            v13[v10] = v8;
        }

        // PRGA : 每次生成一个 keystream 字节, 与数据 XOR   
        while ( n152_1 < 152 )
        {
            v7 = (v7 + 1) % 256;
            v11 = ((unsigned __int8)v13[v7] + v11) % 256;
            v9 = v13[v7];
            v13[v7] = v13[v11];
            v13[v11] = v9;
            *(_BYTE *)(a1 + n152_1) ^= v13[((unsigned __int8)v13[v11] + (unsigned __int8)v13[v7]) % 256];
            result = (unsigned int)++n152_1;
        }
        return result;
        ```
    - 推断出是**RC4**:
        - 编译时 : 作者用 `RC4`(key="babyflag") 对明文 `shellcode` 做 `XOR` 得到密文, 写进 `v9`
        - 运行时 : 程序再用同一个 `key` 对 `v9` 做 `XOR`, 还原出明文 `shellcode`
        - 所以我们只要在本地脚本里跑一遍 `RC4(v9, "babyflag")`, 就能得到 `shellcode` 的明文
        > 注意 : 加密/解密用的是同一套操作(**流密码**的特性), 因为:
        > - C = P XOR K
        > - P = C XOR K
    - 通过抄标准代码写出解密函数
        ``` Python
        # 看 shellcode 生成在哪里
        import os
        print("CWD:", os.getcwd())

        v9 = [
            0x68, 0x60, 0x0C, 0x1B, 0x2A, 0xB3, 0xEE, 0x4A, 0x17, 0x7C, 0xB7, 0xF6, 0x91,  
            0xEA, 0x92, 0x2D, 0x6B, 0xAD, 0x61, 0xC2, 0x5F, 0x70, 0x2C, 0x14, 0x74, 0x0E,   
            0xA2, 0xAF, 0x8A, 0x57, 0xFF, 0x16, 0xD2, 0x18, 0xDF, 0x4C, 0xB4, 0x4D, 0x80,   
            0x8C, 0xDA, 0xB0, 0x81, 0x41, 0xB5, 0x64, 0x8B, 0x71, 0xE5, 0x36, 0x39, 0x46,   
            0x10, 0xF2, 0x97, 0x25, 0xB0, 0x05, 0x10, 0x00, 0x7F, 0x96, 0xE4, 0x64, 0x0C,   
            0x0B, 0x14, 0xBC, 0x52, 0xEA, 0x64, 0xB6, 0xE5, 0xDE, 0x03, 0xB5, 0x52, 0x4E,   
            0x8D, 0x1F, 0x66, 0xCD, 0x68, 0x19, 0x65, 0x93, 0x5F, 0xC1, 0x30, 0xBC, 0xD0,   
            0x52, 0x86, 0x01, 0x4D, 0xB6, 0x99, 0x45, 0x40, 0x66, 0x3B, 0xBE, 0x13, 0x42,   
            0x4E, 0x9B, 0x18, 0x6D, 0xBA, 0x00, 0x74, 0x99, 0xB2, 0x65, 0xEC, 0x6C, 0xDF,  
            0x51, 0x17, 0x8A, 0x84, 0x3A, 0xF3, 0x5D, 0xC8, 0xE9, 0x88, 0x65, 0x9D, 0x5B,   
            0x4F, 0x1D, 0xC1, 0x16, 0xB5, 0x96, 0xC4, 0x8C, 0xFB, 0xEA, 0xA2, 0x16, 0x23,   
            0x38, 0x8E, 0xE4, 0x09, 0x99, 0x55, 0x58, 0x4A, 0x4F
        ]
        key = b"babyflag"

        def KSA(key):
            S = list(range(256))
            j = 0
            for i in range(256):
                j = (j + S[i] + key[i % len(key)]) & 0xFF
                S[i], S[j] = S[j], S[i]
            return S
        
        def PRGA(S):
            i, j = 0, 0
            while True:
                i = (i + 1) & 0xFF
                j = (j + S[i]) & 0xFF
                S[i], S[j] = S[j], S[i]
                K = S[(S[i] + S[j]) & 0xFF]
                yield K
        
        def RC4(key, text):
            S = KSA(key)
            keystream = PRGA(S)
            res = []
            for char in text:
                res.append(char ^ next(keystream))
            return bytes(res)

        # 标准代码里是两个字符串
        enc = bytes(v9)
        ans = RC4(key, enc)

        print("len:", len(ans))
        print("first 16 bytes:", ans[:16].hex())

        with open("shellcode.bin", "wb") as f:
            f.write(ans)

        print("shellcode.bin written, size:", len(ans))
        ```

- 折腾了半天, 最后将生成的`shellcode.bin`**直接**用IDA打开
    > 开始的时候问ai, ai让我用IDA里的`Additional binary file`打开, 编译死活不对  
    > 为此还去升级了IDA的版本, 最后发现**直接打开**就可以了
    - 最终看到`sub_0`:
        ``` C++
        if ( !a4 )
        // 结束程序
        LABEL_6:
            JUMPOUT(0x98);
        n0x20 = 0;

        v9[0] = 536952951;
        v9[1] = 1997148275;
        v9[2] = 2081112864;
        v9[3] = 1934771314;
        v9[4] = 1934627112;
        v9[5] = 542524535;
        v9[6] = 1996755745;
        v9[7] = 1917994785;

        // key : 0x11223344
        n1144201745 = 1144201745;

        // 32 次循环
        do
        {
            // 逻辑就是 : T[i] = buf[i] ^ key[i & 3]
            if ( *((unsigned __int8 *)v9 + n0x20) != (*((char *)v9 + n0x20 + a4 - (_QWORD)v9)
                                                    ^ *((unsigned __int8 *)&n1144201745 + (n0x20 & 3))) )
            goto LABEL_6;
            ++n0x20;
        }
        while ( n0x20 < 0x20 );
        return 1;
        ```
    - 据此写出解密函数:
        ``` python
        v9 = [
            536952951, 1997148275, 2081112864, 1934771314,   
            1934627112, 542524535, 1996755745, 1917994785 
        ]

        # 把 v9 分成 32 份
        T = list()
        for w in v9:
            T.append(w & 0xFF)
            T.append((w >> 8) & 0xFF)
            T.append((w >>16) & 0xFF)
            T.append((w >>24) & 0xFF)

        # key[i & 3]
        k = [0x11, 0x22, 0x33, 0x44]

        for i in range(32):
            T[i] ^= k[i % 4]

        print('DASCTF{' + ''.join(chr(c) for c in T) + '}')
        ```
 
*运行程序得到flag*  
总结 : 我还是远远不够啊🥲