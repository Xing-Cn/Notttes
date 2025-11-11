# Catch_Tofv
*学习学习...*

- ida, 从main开始一直找找找, 终于找到关键所在
    ``` c++
    int sub_4010C0() {             
        HANDLE FileW; // ebx
        size_t i; // edi
        DWORD v3; // eax
        unsigned int v4; // ebx
        int v5; // edx
        unsigned int v6; // ecx
        unsigned int v7; // eax
        unsigned int v8; // esi
        unsigned int v9; // ebx
        int v10; // edi
        unsigned int v11; // eax
        bool v12; // zf
        unsigned int v13; // edx
        unsigned int *v14; // esi
        int *v15; // ecx
        int v16; // eax
        bool v17; // cf
        unsigned __int8 v18; // al
        unsigned __int8 v19; // al
        unsigned __int8 v20; // al
        unsigned int v21; // eax
        HANDLE hObject; // [esp+4h] [ebp-47Ch]
        unsigned int *v23; // [esp+8h] [ebp-478h]
        int v24; // [esp+Ch] [ebp-474h]
        int v25; // [esp+10h] [ebp-470h]
        int v26; // [esp+14h] [ebp-46Ch]
        unsigned int v27; // [esp+18h] [ebp-468h]
        unsigned int v28; // [esp+1Ch] [ebp-464h]
        unsigned int v29; // [esp+20h] [ebp-460h]
        unsigned int v30; // [esp+24h] [ebp-45Ch]
        unsigned int v31; // [esp+28h] [ebp-458h]
        int v32; // [esp+28h] [ebp-458h]
        unsigned int v33; // [esp+2Ch] [ebp-454h]
        unsigned int v34; // [esp+30h] [ebp-450h]
        unsigned int v35; // [esp+34h] [ebp-44Ch]
        unsigned int v36; // [esp+38h] [ebp-448h]
        DWORD NumberOfBytesRead; // [esp+3Ch] [ebp-444h] BYREF
        int v38[8]; // [esp+40h] [ebp-440h] BYREF
        char Buffer[1028]; // [esp+60h] [ebp-420h] BYREF
        _BYTE Text[24]; // [esp+464h] [ebp-1Ch] BYREF

        FileW = CreateFileW(L"flag.loveyou", 0x80000000, 1u, 0, 3u, 0x80u, 0);
        hObject = FileW;
        if ( !FileW )
            return MessageBoxW(0, L"Can't find you file", L"Error", 0);
        for ( i = 0; ReadFile(FileW, Buffer, 0x400u, &NumberOfBytesRead, 0); i = v3 ) {
            v3 = NumberOfBytesRead;
            if ( !NumberOfBytesRead )
                break;
            if ( NumberOfBytesRead >= 0x401 )
                sub_401B54();
            Buffer[NumberOfBytesRead] = 0;
        }
        v23 = (unsigned int *)calloc(8u, 4u); // 从这里开始, xxtea的加密
        memcpy(v23, Buffer, i);               // 这里是读入八个字符
        v4 = v23[7];
        v5 = 0;
        v6 = v23[2];
        v33 = v23[3];
        v34 = v23[4];
        v35 = v23[5];
        v30 = v23[6];
        v7 = *v23;
        v8 = v23[1];
        *(_DWORD *)&Text[8] = 286331153;      // 按 H 得到是 0x11111111
        qmemcpy(&Text[12], "\"\"\"\"3333DDDD", 12); // 剩下三个key "\" 是 0x22, 3 是 0x33, D 是 0x44, 拼接得出
        v26 = 12;                             // 12 轮
        v31 = v4;                             // 一些反编译出来的东东
        v29 = v6;
        v27 = v4;
        v28 = v7;
        do {                                  // 加密的核心部分
            v9 = ((unsigned int)(v5 - 0x61C88647) >> 2) & 3;
            v25 = *(_DWORD *)&Text[4 * v9 + 8];
            v28 += (((16 * v31) ^ (v8 >> 3)) + ((v31 >> 5) ^ (4 * v8))) ^ (((v5 - 0x61C88647) ^ v8) + (v31 ^ v25));
            v32 = *(_DWORD *)&Text[4 * (v9 ^ 1) + 8];
            v8 += ((v28 ^ v32) + ((v5 - 0x61C88647) ^ v29)) ^ (((16 * v28) ^ (v29 >> 3)) + ((v28 >> 5) ^ (4 * v29)));
            v24 = *(_DWORD *)&Text[4 * (v9 ^ 2) + 8];
            v10 = *(_DWORD *)&Text[4 * (v9 ^ 3) + 8];
            v29 += ((v8 ^ v24) + ((v5 - 0x61C88647) ^ v33)) ^ (((16 * v8) ^ (v33 >> 3)) + ((v8 >> 5) ^ (4 * v33)));
            v33 += ((v29 ^ v10) + ((v5 - 0x61C88647) ^ v34)) ^ (((16 * v29) ^ (v34 >> 3)) + ((v29 >> 5) ^ (4 * v34)));
            v34 += (((v5 - 0x61C88647) ^ v35) + (v33 ^ v25)) ^ (((16 * v33) ^ (v35 >> 3)) + ((v33 >> 5) ^ (4 * v35)));
            v35 += ((v34 ^ v32) + ((v5 - 0x61C88647) ^ v30)) ^ (((16 * v34) ^ (v30 >> 3)) + ((v34 >> 5) ^ (4 * v30)));
            v30 += ((v35 ^ v24) + ((v5 - 0x61C88647) ^ v27)) ^ (((16 * v35) ^ (v27 >> 3)) + ((v35 >> 5) ^ (4 * v27)));
            v11 = ((((v5 - 0x61C88647) ^ v28) + (v30 ^ v10)) ^ (((16 * v30) ^ (v28 >> 3)) + ((v30 >> 5) ^ (4 * v28)))) + v27;
            v12 = v26-- == 1;
            v5 -= 0x61C88647;
            v27 = v11;
            v31 = v11;
        } while ( !v12 );
        v36 = v8;
        v13 = 28;
        v14 = v23;
        v38[0] = 0xA050A8B8;                // 目标
        v38[1] = 0xA9CE2ED5;
        v38[2] = 0x5ECCD823;
        v23[7] = v11;
        v23[5] = v35;
        v23[4] = v34;
        v23[3] = v33;
        v23[1] = v36;
        v23[6] = v30;
        v15 = v38;
        v23[2] = v29;
        *v23 = v28;
        v38[3] = 0x3DFDD731;
        v38[4] = 0x1E1D219F;
        v38[5] = 0x661F3884;
        v38[6] = 0x72BB3281;
        v38[7] = 0x985A7480;
        while ( *v14 == *v15 ) {
            ++v14;
            ++v15;
            v17 = v13 < 4;
            v13 -= 4;
            if ( v17 ) {
            v16 = 0;
            goto LABEL_19;
            }
        }
        v17 = *(_BYTE *)v14 < *(_BYTE *)v15;
        if ( *(_BYTE *)v14 == *(_BYTE *)v15
            && (v18 = *((_BYTE *)v14 + 1), v17 = v18 < *((_BYTE *)v15 + 1), v18 == *((_BYTE *)v15 + 1))
            && (v19 = *((_BYTE *)v14 + 2), v17 = v19 < *((_BYTE *)v15 + 2), v19 == *((_BYTE *)v15 + 2))
            && (v20 = *((_BYTE *)v14 + 3), v17 = v20 < *((_BYTE *)v15 + 3), v20 == *((_BYTE *)v15 + 3)) ) {
            v16 = 0;
        }
        else {
            v16 = v17 ? -1 : 1;
        }
        LABEL_19:
        v12 = v16 == 0;
        v21 = 16;
        if ( v12 ) {
            strcpy(&Text[16], "FFF\t\t\t");
            *(__m128 *)Text = _mm_xor_ps((__m128)xmmword_4032B0, (__m128)xmmword_4032C0);
            do
            Text[v21++] ^= 0x28u;
            while ( v21 < 0x16 );
        }
        else {
            *(_DWORD *)&Text[16] = 1852707089;
            *(_DWORD *)&Text[20] = 131586;
            *(__m128 *)Text = _mm_xor_ps((__m128)xmmword_4032A0, (__m128)xmmword_4032D0);
            do
            Text[v21++] ^= 0x21u;
            while ( v21 < 0x17 );
        }
        MessageBoxA(0, Text, "Message", 0);
        return CloseHandle(hObject);
    }
    ```
    > 这里学到又一神技 : 对着数字按 `H` 可以很方便的转为16进制(不管是正的还是负的)

- 判断这个是xxtea, 是因为我看到: 
    - `v28 += (((16 * v31) ^ (v8 >> 3)) + ((v31 >> 5) ^ (4 * v8))) ^ (((v5 - 0x61C88647) ^ v8) + (v31 ^ v25));`   
    - 这刚好和**标准xxtea**里的`(((z>>5^y<<2) + (y>>3^z<<4)) ^ ((sum^y) + (key[(idx&3)^e] ^ z)))`相符合
- 于是学习xxtea, 先找**关键参数** : rounds已知, delta = -0x61C88647(标准的加密里是 sum + delta), key 和 目标结果(我记作to)也已知
    - 学习这个解密确实花了我不少的时间 : 一开始判断错了加密方式, 后来ai识别参数错误
    - 不过最后还是成功了

 - 接下来就是完整的解密了:
    ```python
    delta = -0x61C88647
    rounds = 12

    def xxtea_decrypt(data, key):
        v = data.copy()
        sum_val = delta * rounds & 0xFFFFFFFF
        n = len(v)

        for i in range(rounds):
            key_index  = (sum_val >> 2) & 3

            for j in range(7, -1, -1):
                nxt = (j + 1) % n
                bfo = (j - 1) % n

                term1 = ((sum_val ^ v[nxt]) + (v[bfo] ^ key[key_index ^ (j & 3)]))
                term2 = ((v[bfo] >> 5) ^ (v[nxt] << 2)) + ((v[nxt] >> 3) ^ (v[bfo] << 4))
                v[j] = (v[j] - (term1 ^ term2)) & 0xFFFFFFFF

            sum_val = (sum_val - delta) & 0xFFFFFFFF
        return v

    to = [0xA050A8B8, 0xA9CE2ED5, 0x5ECCD823, 0x3DFDD731, 0x1E1D219F, 0x661F3884, 0x72BB3281, 0x985A7480]
    key = [0x11111111, 0x22222222, 0x33333333, 0x44444444]

    result = xxtea_decrypt(to, key)

    # 神奇的输出, 这里是由ai完成的, 说是Windows要按小端序输出
    import struct
    bytes_result = b''.join(struct.pack('<I', x) for x in result)
    print("字节形式:", bytes_result)
    ```
    > '<I' 是按小端序, '>I'则是大端序  
    > 只能说是稍微理解了一下

*总之, 运行程序得到flag*  
总结:   
- 血与泪啊, **不能太依赖ai**, 错误的参数真的浪费了好长的时间  
- 不过ai倒是教会了我怎么玩这个小程序, ~~把窗口调小, 图标就没法乱动了😁~~