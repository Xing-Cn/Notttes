# EasyUPX
*UPX脱壳的练习*

- 到手IDA发现查看不了
    - 用UPX看看
    ``` bash
    .\upx -l main.exe
    ```
    - 有壳, 脱了
    ``` bash
    .\upx -d main.exe -o main_unpacked.exe
    ```

- 这下可以用IDA查看了
    - `main`:
        ``` C++   

        // 初始化加输入
        ...

        // 对输入内容进行 base64 加密
        base64_encode((const char *)&encoded, in_len);

        for ( i = 0; ; ++i ) {
            i_1 = i;
            if ( i_1 >= std::string::length(&original) )
                break;
            v4 = std::string::operator[](&encoded, i);
            // 逐个与 str 比较
            if ( *v4 != str[i] ) {
                printf("No...");
                goto LABEL_7;
            }
        }
        printf("Yes!!!");
        LABEL_7:
        std::string::~string(&encoded);
        std::string::~string(&original);
        return 0;
        ```
        > str = "IzyxLKW2IIOLpQEwnmAxsD9X"
    - `base64_encode`:
        ``` C++
        bytes_to_encodea = *(const char **)&in_len;
        in_lena = in_lena_1;
        std::string::string((std::string *)bytes_to_encode);
        i = 0;
        j = 0;
        while ( in_lena-- )
        {
            v3 = i++;
            v4 = (unsigned __int8 *)bytes_to_encodea++;
            char_array_3[v3] = *v4;
            if ( i == 3 )
            {
            char_array_4[0] = char_array_3[0] >> 2;
            char_array_4[1] = 16 * (char_array_3[0] & 3) + (char_array_3[1] >> 4);
            char_array_4[2] = 4 * (char_array_3[1] & 0xF) + (char_array_3[2] >> 6);
            char_array_4[3] = char_array_3[2] & 0x3F;
            for ( i = 0; i <= 3; ++i )
            {
                // 发现 base64_chars 是字符表, 一路按 x 寻找
                std::string::operator[](&base64_chars, char_array_4[i]);
                std::string::operator+=((std::string *)bytes_to_encode);
            }
            i = 0;
            }
        }
        if ( i )
        {
            for ( j = i; j <= 2; ++j )
            char_array_3[j] = 0;
            char_array_4[0] = char_array_3[0] >> 2;
            char_array_4[1] = 16 * (char_array_3[0] & 3) + (char_array_3[1] >> 4);
            char_array_4[2] = 4 * (char_array_3[1] & 0xF) + (char_array_3[2] >> 6);
            for ( j = 0; i + 1 > j; ++j )
            {
            std::string::operator[](&base64_chars, char_array_4[j]);
            std::string::operator+=((std::string *)bytes_to_encode);
            }
            while ( 1 )
            {
            n2 = i++;
            if ( n2 > 2 )
                break;
            std::string::operator+=((std::string *)bytes_to_encode);
            }
        }
        return (std::string)bytes_to_encode;
        ```
        > base64_chars = "NOPQRSTUVWXYZABCDEFGHIJKLMnopqrstuvwxyzabcdefghijklm9876543210+/"  
        > 对字母进行了BOT13

- 以前遇到过这种情况了
    - 复制代码, 稍作修改
        ``` python
            import base64

            org = 'IzyxLKW2IIOLpQEwnmAxsD9X'

            xian_biao = 'NOPQRSTUVWXYZABCDEFGHIJKLMnopqrstuvwxyzabcdefghijklm9876543210+/'
            yuan_biao = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/'

            midd = org.translate(str.maketrans(xian_biao, yuan_biao))
            ans = base64.b64decode(midd).decode()

            print(ans)
        ```

*运行程序得到flag*  
总结 : 简单的UPX复习🎉