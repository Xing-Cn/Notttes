# Bash
*基础操作*

> Bash (Bourne-Again SHell) 是 `Linux/Unix` 系统的默认命令行解释器  
> 可以执行命令、编写脚本
- 查看文件内容
``` bash
cat file.txt          # 显示整个文件
head -n 10 file.txt   # 显示前10行
tail -n 10 file.txt   # 显示后10行
less file.txt         # 分页查看（按q退出）
```

- 文件信息查看
``` bash
file flag.txt         # 查看文件类型
ls -la                # 详细文件列表（显示隐藏文件）
stat flag.txt         # 文件详细信息
wc -l file.txt        # 统计行数
```

- 搜索文件内容
``` bash
grep "flag{" file.txt      # 搜索包含"flag{"的行
grep -r "password" .       # 递归搜索当前目录
strings binary_file        # 提取可打印字符串
hexdump -C file.bin        # 十六进制查看
xxd file.bin               # 另一种十六进制查看工具
```