# sudoku
*python逆向题*

- 拿到是一个exe文件, 运行后要我输入, 输入但是报错

- 搜索如何反编译python:
    - 先看是用什么工具打包的(使用`PE-bear`)
    > 打包工具: **PyInstaller**[modified]

    - 下载`PyInstaller`, 尝试解包
    > 初次尝试显示:     
    >   [!] Please run this script in **Python 3.8** to prevent extraction errors during unmarshalling

    - 费尽心思下好正确版本, 成功解包  
      直接就可以查看源代码了

- 反编译完成, 却让我看到了更大的问题

    > **一个 16 * 16 的数独**

    - 显然我是做不出来这个了

    - 求助AI, 发现AI是笨蛋, 长度都能搞错

    - 遂求助伟大的搜索引擎

- 还真让我做出来了

*把答案套上层hgame{}得出flag*  
总结: python也是可以反编译的!