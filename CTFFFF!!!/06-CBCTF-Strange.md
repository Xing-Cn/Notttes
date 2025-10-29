# Strange
*花指令初体验*

- 拿到题, IDA, 然后蒙了, 由于做这道题的时间跨度较长, 当时我根本无从下手
    - 稍微练了会级, 回来依旧不会, 无奈请教学长:
        > 花园就是花指令  
    - 学长如是说, 顺便带着我学了解逆向题的一般思路:
        - 可以从输出反推进行检查
        - 在string界面按 `Ctrl` + `F` 可以快速搜索
    - 这道题学会的东西有点多, 写的时候思路有些混乱, ~~就全堆在这了~~

- 好, 接下来就是解决花指令的问题
    - 由于无法查看伪代码, 接下来的操作只能在汇编界面进行(花了我好长时间TAT)
    - 首先, 在string界面按下 `Ctrl` + `F`, 搜索 `success`
        > 这是因为运行程序之后显示 : "请输入flag"  
        > 随便输入几个数后, 显示 : "success" 或 "fail"  
    - 找到 `success` 按住 `x` 不断跟进, 一直向上, 直到红色区域的边缘终于找到问题所在:
        ```c++
        .text:004011C0 loc_4011C0:                             ; CODE XREF: sub_40174D+F8↓p
        .text:004011C0                 push    ebp
        .text:004011C1                 mov     ebp, esp
        .text:004011C3                 sub     esp, 11Ch
        .text:004011C9                 mov     eax, ___security_cookie
        .text:004011CE                 xor     eax, ebp
        .text:004011D0                 mov     [ebp-4], eax
        .text:004011D3                 push    ebx
        .text:004011D4                 push    esi
        .text:004011D5                 push    edi
        .text:004011D6                 mov     dword ptr [ebp-11Ch], 1
        .text:004011E0                 cmp     dword ptr [ebp-11Ch], 0
        .text:004011E7                 jz      short loc_4011EB
        .text:004011E9                 jmp     short near ptr loc_4011EB+2
        ```

    - 经过指点后看出 : `mov` 将 `1` 赋值给了这个 `ptr`, 后面又将其与 `0` 比较, 导致 `loc_4011EB` **根本进不去**, 而这个函数却是跳到 `success` 的关键所在, 于是:
        > 把 `jz` 改成 `jmp`
    
    - 这时再对红色区域的开头按 `P`, 发现红色区域**恢复正常**！
    - 按下 `F5` 爽爽查看伪代码
        > 其实这里我没有完全改正的, 底下有一处地址也是有问题:  
        > `call    near ptr 0E0407AEFh`   
        > 但我改成 `nop` 的话, 变成:  
        > `jmp     fword ptr [eax+0] ; "Enter input: "`  
        > 仍旧有问题, 但再 `nop` 的话, "请输入" 部分就要整没了      
        > 学长是把这里的改成了优美的 `push    offset aEnterInput ; "Enter input: "`          
        > 没招了, 先这样吧
    
- 展示代码:
    ``` C++
    int __usercall sub_4011C0@<eax>(int a1@<ebx>) {
        _BYTE *v1; // eax
        __int16 v2; // cx
        int v3; // eax
        int v4; // eax
        int v6; // eax
        int v7; // eax
        int v8; // eax
        unsigned int v9; // [esp+14h] [ebp-114h]
        unsigned int v10; // [esp+18h] [ebp-110h] BYREF
        unsigned int i; // [esp+1Ch] [ebp-10Ch]
        LPVOID lpMem; // [esp+20h] [ebp-108h]
        char v13[256]; // [esp+24h] [ebp-104h] BYREF

        v1 = (_BYTE *)MEMORY[0xE0407AEF](); // 这里是我改不来了
        LOBYTE(v1) = ((unsigned __int16)(v2 + 1) >> 8) + (_BYTE)v1;
        *(_BYTE *)(a1 + 6948036) += *v1 + (_BYTE)v1;
        v3 = sub_4059A8();
        if ( sub_405B85(v13, 256, v3) ) {
            v9 = sub_4079B0(v13, asc_41E01C);
            if ( v9 >= 0x100 )
                sub_40159D();
            v13[v9] = 0;
            v10 = 0;
            v6 = sub_407A00(v13);
            lpMem = (LPVOID)sub_401000(v13, v6, &v10);
            if ( lpMem ) {
                for ( i = 0; i < v10; ++i )
                    *((_BYTE *)lpMem + i) ^= 0x16u;
                v8 = sub_407A00(aFuxnfbQxrXeFLq);
                if ( v10 != v8 || sub_4021DB(lpMem, aFuxnfbQxrXeFLq, v10) ) {
                    sub_401440(aFail);
                    sub_401440(a99s);
                }
                else {
                    sub_401440(aSuccess);
                }
                sub_407990(lpMem);
                return 0;
            }
            else {
                v7 = sub_4059A8();
                sub_401400(v7, aMemoryError);
                return 1;
            }
        }
        else {
            v4 = sub_4059A8();
            sub_401400(v4, aReadError);
            return 1;
        }
    }
    ```

- 看上去是 `sub_401000` + `^0x16`
    - 这里的 `aFuxnfbQxrXeFLq` 是:
        > FuxNFb|$QXR#Xe#}F#lqZ[`T@Q&&
    - 查看 `sub_401000` : 是base64的加密
        ``` C++
        int __cdecl sub_401000(int a1, unsigned int a2, _DWORD *a3) {
            char v4; // [esp+Ch] [ebp-24h]
            char v5; // [esp+10h] [ebp-20h]
            int v6; // [esp+14h] [ebp-1Ch]
            int v7; // [esp+18h] [ebp-18h]
            int v8; // [esp+1Ch] [ebp-14h]
            unsigned int v9; // [esp+20h] [ebp-10h]
            int v10; // [esp+24h] [ebp-Ch]
            int v11; // [esp+28h] [ebp-8h]
            int v12; // [esp+28h] [ebp-8h]
            int v13; // [esp+28h] [ebp-8h]
            int v14; // [esp+28h] [ebp-8h]
            unsigned int v15; // [esp+2Ch] [ebp-4h]

            *a3 = 4 * ((a2 + 2) / 3);
            v10 = sub_4079A0(*a3 + 1);
            if ( !v10 )
                return 0;
            v15 = 0;
            v11 = 0;
            while ( v15 < a2 ) {
                v8 = *(unsigned __int8 *)(v15 + a1);    // 第一个字节
                if ( ++v15 >= a2 ) {                    // v15 偷偷 +1, 指向下一字节
                    v7 = 0;
                }
                else {
                    v7 = *(unsigned __int8 *)(v15 + a1);// 第二个字节
                    ++v15;                              
                }
                if ( v15 >= a2 ) {
                    v6 = 0;
                } 
                else {
                    v6 = *(unsigned __int8 *)(v15 + a1);// 第三个字节
                    ++v15;
                }
                v9 = v6 | (v7 << 8) | (v8 << 16);       // 位移拼接
                *(_BYTE *)(v11 + v10) = byte_417180[(v9 >> 18) & 0x3F]; // 再分第一组, 并映射到表
                v12 = v11 + 1;
                *(_BYTE *)(v12 + v10) = byte_417180[(v9 >> 12) & 0x3F]; // 第二组
                v13 = v12 + 1;
                if ( v15 <= a2 + 1 )                    // 还有 2 个可读
                    v5 = byte_417180[(v9 >> 6) & 0x3F]; // 第三组
                else
                    v5 = 61;                            // 填充等号
                *(_BYTE *)(v13 + v10) = v5;             
                v14 = v13 + 1;
                if ( v15 <= a2 )
                    v4 = byte_417180[v9 & 0x3F];        // 最后一组
                else
                    v4 = 61;
                *(_BYTE *)(v14 + v10) = v4;
                v11 = v14 + 1;
            }
            *(_BYTE *)(v10 + *a3) = 0;
            return v10;
        }
        ```
    - 这里相当的阴啊, base64的加密出题人使用了自定义的编码表
        > 标准 : `ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/`  
        > 这里 : `0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz+/`
        
- 学习后得到以下代码: 
    ```  python 3
    # 尝试写一下, base64的解码
    import base64

    org = 'FuxNFb|$QXR#Xe#}F#lqZ[`T@Q&&'
    midd = str()

    # 异或还原
    for i in org:
        midd = midd + chr(ord(i) ^ 0x16)

    xian_biao = '0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz+/'
    yuan_biao = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/'

    # 替换还原
    midd = midd.translate(str.maketrans(xian_biao, yuan_biao))
    ans = base64.b64decode(midd).decode()

    print(ans)
    ```

*运行程序得出flag*  
总结 : 见识了一下花指令👍