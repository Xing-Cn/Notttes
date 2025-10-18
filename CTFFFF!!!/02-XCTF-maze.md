# maze
*通过阅读代码解决问题*

- 拿到题看了代码
  ```c++
  __int64 __fastcall main(int a1, char **a2, char **a3) {
    __int64 v3; // rbx
    int v4; // eax
    bool v5; // bp
    bool v6; // al
    const char *v7; // rdi
    int v9; // [rsp+0h] [rbp-28h] BYREF
    int v10[9]; // [rsp+4h] [rbp-24h] BYREF

    v10[0] = 0;
    v9 = 0;
    puts("Input flag:");
    scanf("%s", &s1);
    // 长度总共为 24, 去头去尾还剩 18
    if ( strlen(&s1) != 24 || strncmp(&s1, "nctf{", 5uLL) || *(&byte_6010BF + 24) != 125 ) { 
    LABEL_22:
        puts("Wrong flag!");
        exit(-1);
    }
    v3 = 5LL;
    if ( strlen(&s1) - 1 > 5 ) {
        while ( 1 ) {
        v4 = *(&s1 + v3);
        v5 = 0;
        if ( v4 > 78 ) {
            if ( (unsigned __int8)v4 == 79 )
            {
            v6 = sub_400650(v10);
            goto LABEL_14;
            }
            if ( (unsigned __int8)v4 == 111 )
            {
            v6 = sub_400660(v10);
            goto LABEL_14;
            }
        }
        else {
            if ( (unsigned __int8)v4 == 46 )
            {
            v6 = sub_400670(&v9);
            goto LABEL_14;
            }
            if ( (unsigned __int8)v4 == 48 )
            {
            v6 = sub_400680(&v9);
    LABEL_14:
            v5 = v6;
            }
        }
        if ( !(unsigned __int8)sub_400690((__int64)asc_601060, v10[0], v9) )
            goto LABEL_22;
        if ( ++v3 >= strlen(&s1) - 1 ) {
            if ( v5 )
            break;
    LABEL_20:
            v7 = "Wrong flag!";
            goto LABEL_21;
        }
        }
    }
    if ( asc_601060[8 * v9 + v10[0]] != 35 )
        goto LABEL_20;
    v7 = "Congratulations!";
    LABEL_21:
    puts(v7);
    return 0LL;
    }
    ```
  >**ASCII**里```125```是'}', ```79```是'O', ```111```是'o', ```46```是'.', ```48```是'0', ```35```是'#'
- 有点复杂了,请教题解  
  ```asc_601060[8 * v9 + v10[0]] != 35```是决定性函数, 这是对坐标进行计算: 
  - 行坐标 * 每行的个数 + 列坐标 = 字符串中的位置  

  其从中可得```v10[0]```为**x坐标**, ```v9```为**y坐标**  
  >这里```asc_601060```的值是'  *******   *  **** * ****  * ***  *#  *** *** ***     *********'  
  转化为 8 * 8：
  ```
  '  ******'
  '*   *  *'
  '*** * **'
  '**  * **'
  '*  *#  *'
  '** *** *'
  '**     *'
  '********'
  ```
  然后观察四个主要的函数  
  ```sub_400650```: 
  ```c++
  bool __fastcall sub_400650(_DWORD *a1) { // v10 --
  int v1; // eax

  v1 = (*a1)--;
  return v1 > 0;
  }
  ```
  ```sub_400660```:
  ```c++
  bool __fastcall sub_400660(int *a1) { // v10 ++
  int v1; // eax

  v1 = *a1 + 1;
  *a1 = v1;
  return v1 < 8;
  }
  ```
  ```sub_400670```:
  ```c++
  bool __fastcall sub_400670(_DWORD *a1) { // v9 --

  v1 = (*a1)--;
  return v1 > 0;
  }
  ```
  ```sub_400680```:
  ```c++
  bool __fastcall sub_400680(int *a1) { // v9 ++ 
  int v1; // eax

  v1 = *a1 + 1;
  *a1 = v1;
  return v1 < 8;
  }
  ```
  分别是```O``` : **向左**, ```o``` : **向右**, ```.``` : **向上**, ```0``` : **向下**  

- 接下来就是走迷宫了  

*从(0, 0)走到'#'得出flag*  
总结 : **耐下性子来读代码吧**😒

