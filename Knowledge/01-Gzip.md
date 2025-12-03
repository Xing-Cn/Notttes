# Gzip 压缩
*常用命令*

``` bash
gzip -c file.txt        # 压缩到标准输出
```
- 标准输入 : `stdin`          # 由键盘输入  
- 标准输出 : `stdout`         # 输出到屏幕

``` bash
gzip -d file.txt.gz     # 解压 .gz 文件    
gunzip file.txt.gz      # 同上, 更常用   
gzip -dk file.txt.gz    # -k: keep (保留原文件)  
cat file.txt.gz | gzip -d  # 解压并显示
```

``` bash
echo "text" | gzip -c | base64 # 管道操作
```
- `echo` 输出文本  
- `|` 管道, 将上一步转移到下一步  
- `gzip -c` 压缩 "text\n"  
- `base64` 编码为文本  