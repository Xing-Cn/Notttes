# EasyUPX+
*I_HATE_UPXTAT*

- 开始依旧跑去脱壳  
  `upx: main.exe: CantUnpackException: file is possibly modified/hacked/protected; take care!`
    > 天塌了, 没见过这个呀

- 遂求助AI
    - 他说要用 `DIE` 查看壳的类型

    - 他说要在 `CFF` 里改这改那

    - 他说这是魔改的壳要用 `x64dbg` 动态脱壳

    - ...
    > 这样子折腾了我两天半

- 然后受不了了去百度看看, 也有人遇到过这个问题
    - 看他的解决方案, 发现 AI 也指挥过我这样做过

    - 仔细看, 又发现不太一样, AI 的指挥**有误** !!!

        - 这道题就是简单的**区段名**被修改了, 但 AI 在指挥我时要我把名字改成 `'.upx0'`, `'.upx1'`, 我照做发现无效

        - 而现在百度到的, 却是 `'UPX0'`, `'UPX1'`, `'UPX2'`, AI 所谓的大小写无所谓在这里也是**严格区分**的

    - 改完区段名, 顺利脱壳

- 脱壳后就是中规中矩的一道题, 进行了 `base64` 的加密
    - 找出目标的 `str`, 查看加密程序是否标准

    - 果不其然用了**修改过**的 base64表

    - 不过值得一提的是, 在程序的开头  
      `patch_base64_chars_from_self`:
    ``` C++
    // 对魔改后的表再进行了一次 patch
    for ( i = 0; i <= 3; ++i )
        {
          v4 = std::string::operator[](&base64_chars, i);
          *v4 = bytecode[i + 392];
        }
        for ( i_0 = 0; i_0 <= 3; ++i_0 )
        {
          v5 = std::string::operator[](&base64_chars, i_0 + 10);
          *v5 = bytecode[i_0 + 432];
        }
        for ( i_1 = 0; i_1 <= 3; ++i_1 )
        {
          v6 = std::string::operator[](&base64_chars, i_1 + 19);
          *v6 = bytecode[i_1 + 472];
        }
    ```
   
   - 不过, 有了上一道 'EasyUPX+' 的经验, 我也是没犯错直接用**脱壳前的文件**读取了这些位置的值

   - 写出解密代码:
   ``` python 
    import base64
    # 原始带壳的 main.exe
    FNAME = r"C:\Users\a1381\Downloads\EasyUPX++\main.exe"  

    # 初始 base64 表
    orig = "QWERTYUIOPASDFGHJKLZXCVBNMqwertyuiopasdfghjklzxcvbnm0123456789+/"

    # 要 patch 的 3 段：(文件偏移, 对应 base64_chars 起始下标)
    OFFSETS = [
        (0x188, 0),   # bytecode[392..395] -> base64_chars[0..3]
        (0x1B0, 10),  # bytecode[432..435] -> base64_chars[10..13]
        (0x1D8, 19),  # bytecode[472..475] -> base64_chars[19..22]
    ]

    b = list(orig)

    with open(FNAME, "rb") as f:
        for off, idx in OFFSETS:
            f.seek(off)
            chunk = f.read(4)           # 4 个字节
            for i in range(4):
                b[idx + i] = chr(chunk[i])

    patched_table = "".join(b)
    print("patched base64 table:", patched_table)

    org = "WdsaNBP7WWCNB3rhrUiyMpKkS30="
    std_table = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/"

    trans = str.maketrans(patched_table, std_table)
    midd = org.translate(trans)
    print("as standard b64:", midd)

    flag_bytes = base64.b64decode(midd)
    print("raw bytes:", flag_bytes)
    try:
        print("decoded str:", flag_bytes.decode())
    except UnicodeDecodeError:
        pass
    ```

*运行程序得到正确flag*  
总结 : 我讨厌 UPX😑