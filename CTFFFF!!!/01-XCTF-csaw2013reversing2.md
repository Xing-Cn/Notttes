# csaw2013reversing2
*通过修改汇编代码解决的*

- 拿到题看了代码
  ```c++
  int __cdecl __noreturn main(int argc, const char **argv, const char **envp) {
    int v3; // ecx
    CHAR *lpMem; // [esp+8h] [ebp-Ch]
    HANDLE hHeap; // [esp+10h] [ebp-4h]

    hHeap = HeapCreate(0x40000u, 0, 0);
    lpMem = (CHAR *)HeapAlloc(hHeap, 8u, SourceSize + 1);
    memcpy_s(lpMem, SourceSize, &unk_409B10, SourceSize);
    if ( !sub_40102A() && !IsDebuggerPresent() ) {
        MessageBoxA(0, lpMem + 1, "Flag", 2u); // 弹窗输出flag
        HeapFree(hHeap, 0, lpMem); 
        HeapDestroy(hHeap);
        ExitProcess(0);
    }
    __debugbreak();
    sub_401000(v3 + 4, lpMem);
    ExitProcess(0xFFFFFFFF); // 退出程序
  }
  ```

- 请教题解后发现  
  `IsDebuggerPresent()`是Windows的**反调试函数**,当文件被调试时输出**1**,导致没进入if,应该想办法跳过,*题解上是在汇编的地方把`jnz`改成`jmp`* 
  - jnz : Jump if Not Zero  
  - jmp : 无条件跳转  

  而`__debugbreak()`是中断，汇编上来看是int 3,也要跳掉,*题解告诉我用`nop`*
  - nop : 空操作指令
  
  之后的`sub_401000(v3 + 4, lpMem)`显然是flag的生成函数了，我知道得**先生成flag**,才可以弹窗显示，因此这个函数应该**在if前就实现**,*题解依旧`jmp`*
  
- 最后的结果是
    ```c++
    int __cdecl __noreturn main(int argc, const char **argv, const char **envp) {
    int v3; // ecx
    CHAR *lpMem; // [esp+8h] [ebp-Ch]
    HANDLE hHeap; // [esp+10h] [ebp-4h]

    hHeap = HeapCreate(0x40000u, 0, 0);
    lpMem = (CHAR *)HeapAlloc(hHeap, 8u, SourceSize + 1);
    memcpy_s(lpMem, SourceSize, &unk_EC9B10, SourceSize);
    sub_EC102A();
    sub_EC1000(v3 + 4, (int)lpMem);
    MessageBoxA(0, lpMem + 1, "Flag", 2u);
    HeapFree(hHeap, 0, lpMem);
    HeapDestroy(hHeap);
    ExitProcess(0);
    }
    ```
*然后再运行程序就得到了flag*  
总结 : **这题我通过在汇编界面修改代码完成**✅