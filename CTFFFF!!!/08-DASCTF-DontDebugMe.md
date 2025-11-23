# DontDebugMe
*可恶的题*

- 现在已经变成无脑扔进IDA了, 有壳的话再说:
    - main:
        ```c++
        int __cdecl main(int argc, const char **argv, const char **envp) {
            int i; // [esp+E8h] [ebp-12Ch]
            int v5; // [esp+100h] [ebp-114h]
            _WORD Buf2[130]; // [esp+10Ch] [ebp-108h] BYREF

            __CheckForDebuggerJustMyCode(&unk_452015);
            memset(Buf2, 0, 0x100u);
            sub_401550("Please input the flag and I will verify it: ");
            sub_4015C0("%256s", (const char *)Buf2);
            if ( IsDebuggerPresent() )
                _loaddll(0);
            v5 = sub_401000();
            for ( i = 0; i < 20; ++i ) {
                Buf2[i] += HIWORD(v5);
                Buf2[i] ^= v5;
            }
            if ( !memcmp(&unk_450000, Buf2, 0x28u) )
                sub_401550("Right flag!\n");
            else
                sub_401550("Wrong flag\n");
            return 0;
        }
        ```

    - 看到v5, 想着和catchtofv一样解出来
        ```c++
        unsigned int sub_401000() {
            int v1; // [esp+Ch] [ebp-F8h]
            unsigned int v2; // [esp+10h] [ebp-F4h]
            int v3; // [esp+10h] [ebp-F4h]
            unsigned int i; // [esp+E4h] [ebp-20h]
            unsigned int v5; // [esp+F0h] [ebp-14h]
            int v6; // [esp+FCh] [ebp-8h]

            __CheckForDebuggerJustMyCode(&unk_452015);
            v6 = -17958194;
            v5 = 2105330038;
            for ( i = 0; i < 0x20; ++i ) {
                if ( (i & 0xAAAAAAAA) != 0 ) {
                    v2 = v5 >> (32 - i % 0x20);
                }
                else {
                    if ( (v5 & 1) != 0 )
                        v1 = ((v5 ^ 0xDEADCAFE) >> 1) + (v5 ^ 0xB00B135);
                    else
                        v1 = v5 ^ (HIBYTE(v5) | (v5 >> 8) & 0xFF00 | ((v5 & 0xFF00) << 8) | ((unsigned __int8)v5 << 24));
                    v2 = v1;
                }
                v5 = v2 | (v5 << (i % 0x20));
                if ( (v5 & 2) != 0 )
                    v3 = -1;
                else
                    v3 = 464371934;
                v6 += v3 * (i ^ 0xDEADBABE);
            }
            return ((((v5 & 0xF0F0F0F | v6 & 0xF0F0F0F0) >> 24) | ((v5 & 0xF0F0F0F | v6 & 0xF0F0F0F0) >> 8) & 0xFF00 | ((v5 & 0xF00 | v6 & 0xF000) << 8) | ((v5 & 0xF | v6 & 0xF0) << 24)) ^ (((v5 & 0xF0F0F0F | v6 & 0xF0F0F0F0) ^ 0xDEADBEEF) + dword_45002C + 11) ^ 0xFFFF0000) & 0xDEAD0000 ^ 0xA600 | (((v5 & 0xF0F0F0F | v6 & 0xF0F0F0F0) >> 24) | ((v5 & 0xF0F0F0F | v6 & 0xF0F0F0F0) >> 8) & 0xFF00 | (unsigned __int16)((v5 & 0xF00 | v6 & 0xF000) << 8)) ^ (unsigned __int16)(((v5 & 0xF0F | v6 & 0xF0F0) ^ 0xBEEF) + dword_45002C + 11);
        }
        ```
    - 然而结果是 : 用python, 用c, 怎么也做不出正确答案(甚至可以说是千奇百怪)

- 实在做不出来, 求助伟大的学长 : **动态调试**
    - 这玩意我确实也不太熟, 开始时想着把所有的反调试都nop掉, 但折腾半天, 程序还是会自己中断
    - 一气之下给所有看得到的地方都下了断点, 一路`F7`按下去, 发现 : **明明改成jmp了却还显示jnz**
    - 明白此法行不通, 尝试在调试过程中改变`eax`的值, 使其误以为没有调试
    - 汇编里是这样的:
        > cmp     eax, 1  
        > jnz     short loc_40141A
    - 然后就血淋淋的过去了, 成功运行到`sub_401000()`
    - 再按F8往下走一步, 心心念念的v5就在eax里出现了

- 接下来就没什么难度了
    - 根据`main`成功写出解决代码:
        ```python
        def IneedV5():
            return 0x0685EE20

        to = [
            0xE9, 0xA9, 0xF8, 0xA7, 0xF9, 0xA2, 0x20, 0xD6, 0x9A, 0xD6, 
            0xC8, 0xD9, 0x99, 0xD3, 0xCB, 0x85, 0x9B, 0xD2, 0xC7, 0xD5, 
            0x96, 0x84, 0xC9, 0xD4, 0x9A, 0xD8, 0xCA, 0xD7, 0x9C, 0xD5, 
            0xC8, 0x85, 0x97, 0xD5, 0x9E, 0x85, 0x9C, 0xD4, 0xCA, 0x6D, 
            0x00, 0x00, 0x00, 0x00]

        relto = []
        for i in range(0, 40, 2):
            relto.append(to[i] | (to[i+1] << 8))

        v5 = IneedV5()
        for i in range(20):
            relto[i] = (relto[i] ^ v5) & 0xFFFF
            relto[i] = (relto[i] - (v5 >> 16 & 0xFFFF)) & 0xFFFF

        flag = b""
        for val in relto:
            flag += val.to_bytes(2, 'little')
        print(flag.decode(errors='ignore'))
        ```
    - 其实前面的尝试也不是没有学到东西:
        > `HIWORD` 的定义 : #define HIWORD(I) ( ( WORD ) ( ( ( DWORD )( I ) >> 16) & 0xFFFF ) )  
        > `HIBYTE` 的定义 : #define HIBYTE(w) ((BYTE)((((DWORD_PTR)(w)) >> 8) & 0xff))  
        > `_WORD` 得是两个字符拼接在一起  
        > windows系统 默认按 `小端序` 输出
    - 还有python 和 C 里的差异:
        > unsigned char / uint8_t -> 0xFF  
        > unsigned short / uint16_t -> 0xFFFF  
        > unsigned int / uint32_t -> 0xFFFFFFFF  
        > unsigned long long / uint64 -> 0xFFFFFFFFFFFFFFFF

*总之运行程序得到flag*  
总结 : 学会了简单的绕过**反调试**✌️