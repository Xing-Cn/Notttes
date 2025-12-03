# Simple_Bash
*第一次接触bash*

- `cat`操作查看文件:
    - `checker_obfuscated.sh`:
        ``` bash
        #!/bin/bash

        // 加解密部分
        rot13() {
            local input="$1"
            local output=""
            local char

            // 循环输入, ${#input} 获取字符串长度
            for (( i=0; i<${#input}; i++ )); do
                
                // ${input:$i:1} 从位置i开始取1个字符
                char="${input:$i:1}"
                case "$char" in

                    // a - m 范围, 右移 13
                    [a-m]|[A-M])
                        output+=$(printf "\\$(printf '%03o' "$(( $(printf '%d' "'$char") + 13 ))")")
                        ;;

                    // n - z 范围, 左移 13
                    [n-z]|[N-Z])
                        output+=$(printf "\\$(printf '%03o' "$(( $(printf '%d' "'$char") - 13 ))")")
                        ;;
                    *)

                    // 其他的, 不变
                        output+="$char"
                        ;;
                esac
            done
            printf "%s" "$output"
        }

        // 读取文件
        encoded_flag=$(cat flag.enc)

        echo "Enter the flag:"
        read user_input

        // 将命令拆分成多个部分
        cmd1="gzi"
        cmd2="p"
        cmd3="base"
        cmd4="64"
        opt1="-c"       // gzip - 标准输出
        opt2="-w 0"     // base64 - 不换行

        // 指令拼接
        compress_cmd="${cmd1}${cmd2} ${opt1}"    // "gzip -c"
        encoder_cmd="${cmd3}${cmd4} ${opt2}"     // "base64 -w 0"

        // 
        user_input_encoded=$(rot13 "$user_input" | $compress_cmd | $encoder_cmd)

        if [ "$user_input_encoded" == "$encoded_flag" ]; then
        echo "correct!"
        else
        echo "incorrect!"
        fi
        ```

        > ROT13(回转13位, rotate by 13 places, 有时中间加了个连字符称作ROT-13)是一种简易的替换式密码, 也是过去在古罗马开发的凯撒加密的一种变体

    - `flag.enc`:
        ``` 
        H4sIAAAAAAAAA/MsK8xLrfYyrywwqCqKN4o3Ly2Kt8hLK43PMkg1LFSsBQAKSq6gIAAAAA==
        ```
    
- 读懂程序, 写解密程序:
    - 原文件加密流程 : 原始flag -> rot13加密  -> gzip压缩 -> base64编码 -> flag.enc

    - 逆向解密流程 : flag.enc -> base64解码 -> gzip解压 -> rot13解密 -> 原始flag

    - `solve.py`:
        ``` python
        import base64
        import gzip

        # 加解密部分, +13 和 -13 实质上一样的(一共就 26 个字母)
        def rot13(text):
            result = ''
            for char in text:
                if 'a' <= char <= 'z':
                    result += chr((ord(char) - ord('a') + 13) % 26 + ord('a'))
                elif 'A' <= char <= 'Z':
                    result += chr((ord(char) - ord('A') + 13) % 26 + ord('A'))
                else:
                    result += char
            return result

        encoded = "H4sIAAAAAAAAA/MsK8xLrfYyrywwqCqKN4o3Ly2Kt8hLK43PMkg1LFSsBQAKSq6gIAAAAA=="

        # 解密流程
        decoded = base64.b64decode(encoded)
        decompressed = gzip.decompress(decoded)

        # 转换为字符串供 rot13 使用
        original = decompressed.decode('ascii')

        print("rot13解密后:", rot13(original))
        ```


*运行程序得到flag*  
总结 : 接触新知识😶‍🌫️