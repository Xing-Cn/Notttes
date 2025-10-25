# re-for-50-plz-50
*通过Ghidra解决问题*

- 查完成分, 直接IDA启动, 却发现**无法识别MIPS架构文件**!!!

- 很难过, deepseek找办法, 出现神奇软件 : **Ghidra** !!!
    > 这里附上我在电脑上使用Ghidra的办法
    > 由于不知为何java加不进path里, 只好选择在线使用  
    > java 环境配置:   
    > $env:JAVA_HOME = "D:\pcl\JDK\zulu21.34.19-ca-jdk21.0.3-win_x64"  
    > $env:PATH = "$env:JAVA_HOME\bin;$env:PATH"
    > 启动 Ghidra:  
    > .\ghidraRun.bat
    > 然后一个神奇的绿色恐龙出现, 将文件拖进去就可以开始使用了

- 之后就是常规操作, 这道题没什么难度, 这里贴一下代码:
    ```c++
    undefined4 main(undefined4 param_1, int param_2) {
        int local_10;
        
        for (local_10 = 0; local_10 < 0x1f; local_10 = local_10 + 1) {
            if (meow[local_10] != (*(byte *)(*(int *)(param_2 + 4) + local_10) ^ 0x37)) {
            print("NOOOOOOOOOOOOOOOOOO\n");
            exit_funct();
            }
        }
        puts("C0ngr4ssulations!! U did it.");
        exit_funct();
        return 1;
    }
    ```

- 之后再瞄了一眼`meow`:
    > 63 62 74 63 71 4c 55 42 43 68 45 52 56 5b 5b 4e 68 40 5f 58 5e 44 5d 58 5f 59 50 56 5b 43 4a

*之后把`meow`的值交给deepseek, 异或出flag*  
总结 : 新工具 Get!✌️