# dmd-50
*依旧读代码解决问题*

- 查完成分, 看源代码
    ```c++
    int __fastcall main(int argc, const char **argv, const char **envp) {
        __int64 v3; // rax
        __int64 v4; // rax
        __int64 v5; // rax
        __int64 v6; // rax
        __int64 v7; // rax
        __int64 v8; // rax
        __int64 v9; // rax
        __int64 v10; // rax
        __int64 v11; // rax
        __int64 v12; // rax
        __int64 v13; // rax
        __int64 v14; // rax
        __int64 v15; // rax
        __int64 v16; // rax
        __int64 v17; // rax
        __int64 v18; // rax
        __int64 v19; // rax
        __int64 v20; // rax
        __int64 v21; // rax
        __int64 v23; // rax
        __int64 v24; // rax
        __int64 v25; // rax
        __int64 v26; // rax
        __int64 v27; // rax
        __int64 v28; // rax
        __int64 v29; // rax
        __int64 v30; // rax
        __int64 v31; // rax
        __int64 v32; // rax
        __int64 v33; // rax
        __int64 v34; // rax
        __int64 v35; // rax
        __int64 v36; // rax
        __int64 v37; // rax
        char v38; // [rsp+Fh] [rbp-71h] BYREF
        char v39[16]; // [rsp+10h] [rbp-70h] BYREF
        char v40[8]; // [rsp+20h] [rbp-60h] BYREF
        __int64 v41; // [rsp+28h] [rbp-58h]
        char v42[56]; // [rsp+30h] [rbp-50h] BYREF
        unsigned __int64 v43; // [rsp+68h] [rbp-18h]

        v43 = __readfsqword(0x28u);
        std::operator<<<std::char_traits<char>>(&std::cout, "Enter the valid key!\n", envp);
        std::operator>><char,std::char_traits<char>>(&edata, v42);
        std::allocator<char>::allocator(&v38);
        std::string::string(v39, v42, &v38);
        md5((MD5 *)v40, (const std::string *)v39);
        v41 = std::string::c_str((std::string *)v40);
        std::string::~string((std::string *)v40);
        std::string::~string((std::string *)v39);
        std::allocator<char>::~allocator(&v38);
        if ( *(_WORD *)v41 == '87'
            && *(_BYTE *)(v41 + 2) == 48
            && *(_BYTE *)(v41 + 3) == 52
            && *(_BYTE *)(v41 + 4) == 51
            && *(_BYTE *)(v41 + 5) == 56
            && *(_BYTE *)(v41 + 6) == 100
            && *(_BYTE *)(v41 + 7) == 53
            && *(_BYTE *)(v41 + 8) == 98
            && *(_BYTE *)(v41 + 9) == 54
            && *(_BYTE *)(v41 + 10) == 101
            && *(_BYTE *)(v41 + 11) == 50
            && *(_BYTE *)(v41 + 12) == 57
            && *(_BYTE *)(v41 + 13) == 100
            && *(_BYTE *)(v41 + 14) == 98
            && *(_BYTE *)(v41 + 15) == 48
            && *(_BYTE *)(v41 + 16) == 56
            && *(_BYTE *)(v41 + 17) == 57
            && *(_BYTE *)(v41 + 18) == 56
            && *(_BYTE *)(v41 + 19) == 98
            && *(_BYTE *)(v41 + 20) == 99
            && *(_BYTE *)(v41 + 21) == 52
            && *(_BYTE *)(v41 + 22) == 102
            && *(_BYTE *)(v41 + 23) == 48
            && *(_BYTE *)(v41 + 24) == 50
            && *(_BYTE *)(v41 + 25) == 50
            && *(_BYTE *)(v41 + 26) == 53
            && *(_BYTE *)(v41 + 27) == 57
            && *(_BYTE *)(v41 + 28) == 51
            && *(_BYTE *)(v41 + 29) == 53
            && *(_BYTE *)(v41 + 30) == 99
            && *(_BYTE *)(v41 + 31) == 48 ) {
            v3 = std::operator<<<std::char_traits<char>>(&std::cout, 'T');
            v4 = std::operator<<<std::char_traits<char>>(v3, 'h');
            v5 = std::operator<<<std::char_traits<char>>(v4, 'e');
            v6 = std::operator<<<std::char_traits<char>>(v5, ' ');
            v7 = std::operator<<<std::char_traits<char>>(v6, 'k');
            v8 = std::operator<<<std::char_traits<char>>(v7, 'e');
            v9 = std::operator<<<std::char_traits<char>>(v8, 'y');
            v10 = std::operator<<<std::char_traits<char>>(v9, ' ');
            v11 = std::operator<<<std::char_traits<char>>(v10, 'i');
            v12 = std::operator<<<std::char_traits<char>>(v11, 's');
            v13 = std::operator<<<std::char_traits<char>>(v12, ' ');
            v14 = std::operator<<<std::char_traits<char>>(v13, 'v');
            v15 = std::operator<<<std::char_traits<char>>(v14, 'a');
            v16 = std::operator<<<std::char_traits<char>>(v15, 'l');
            v17 = std::operator<<<std::char_traits<char>>(v16, 'i');
            v18 = std::operator<<<std::char_traits<char>>(v17, 'd');
            v19 = std::operator<<<std::char_traits<char>>(v18, ' ');
            v20 = std::operator<<<std::char_traits<char>>(v19, ':');
            v21 = std::operator<<<std::char_traits<char>>(v20, ')');
            std::ostream::operator<<(v21, &std::endl<char,std::char_traits<char>>);
            return 0;
        }
        else {
            v23 = std::operator<<<std::char_traits<char>>(&std::cout, 'I');
            v24 = std::operator<<<std::char_traits<char>>(v23, 'n');
            v25 = std::operator<<<std::char_traits<char>>(v24, 'v');
            v26 = std::operator<<<std::char_traits<char>>(v25, 'a');
            v27 = std::operator<<<std::char_traits<char>>(v26, 'l');
            v28 = std::operator<<<std::char_traits<char>>(v27, 'i');
            v29 = std::operator<<<std::char_traits<char>>(v28, 'd');
            v30 = std::operator<<<std::char_traits<char>>(v29, ' ');
            v31 = std::operator<<<std::char_traits<char>>(v30, 'K');
            v32 = std::operator<<<std::char_traits<char>>(v31, 'e');
            v33 = std::operator<<<std::char_traits<char>>(v32, 'y');
            v34 = std::operator<<<std::char_traits<char>>(v33, '!');
            v35 = std::operator<<<std::char_traits<char>>(v34, ' ');
            v36 = std::operator<<<std::char_traits<char>>(v35, ':');
            v37 = std::operator<<<std::char_traits<char>>(v36, '(');
            std::ostream::operator<<(v37, &std::endl<char,std::char_traits<char>>);
            return 0;
        }
    }
    ```

- 这玩意是真有点抽象了, 不过也就是苦茶子拉得大了点, 实际上没啥. **关键在这`if`这一块**
    - 从别人的题解那学到了一个小妙招 : 在IDA中对着ASCII码**按R**就可以转换为字符
    - 于是得到 : `870438d5b6e29db0898bc4f0225935c0`

- 原文中藏了一个`md5((MD5 *)v40, (const std::string *)v39)`, 不难得出这玩意就是一个md5密文, 解密得 : `grape`

- 阴的地方就来了, 这里是对MD5格式的字符再进行了一次md5加密,也就是说这玩意加密了两次, 所以这里解密出来的是解密了两遍的东西, 而实际上程序**只进行了一次加密**, 所以得对解密得到的东西再加密一次
    > 这里的密文类型检测出来是`md5(md5($pass))`

*由此得出flag*
总结 : 积累经验, 了解更多加密方式🥲