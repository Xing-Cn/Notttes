# PVZ
*大道至简说是*

- 这道题一点不难, 但是花了我很多时间
> 先来讲我的错误思路

- 拿到题目, 我就被我的惯性思维坑害了  
  ~~我直接跑去看这个程序的源代码了~~
    - 自然, 没看明白(因为有很多很多行, 而且找不到main)
    - 随后使用 DIE, 发现这个是 JAVA 的
    - 经学习得, 这种东西可以用 7-zip 这样的工具直接解压  
    
    其实到此为止没啥大问题

    - 然后问题就出现了:  
      解压完后接着去找源代码了, 什么png, mp3  
      ~~鸟都不鸟你~~
    - 我就这样一头扎进了class文件的海洋里
    - 更可恶的是, 这些文件还被混淆了, 大堆的II1l把我眼睛都看花了
    - 好不容易找到了核心代码:
    ``` JAVA
    // 这里是计算植物的hash值, 一个用于校验是否是神秘仪式, 一个用于生成flag

      public final long O() {
        long l = 3405691582L;
        for (int i = 0; i < 9; ++i) {
            for (int j = 4; -1 < j; --j) {
                Plant plant = this.O(i, j);
                if (plant == null) continue;
                String string = Reflection.getOrCreateKotlinClass(plant.getClass()).getSimpleName();
                int n = string != null ? string.hashCode() : 0;
                l = l << 5 ^ l >>> 27;
                l = l ^ ((long)i << 16 | (long)j) ^ (long)n;
            }
        }
        return l;
    }

    public final long II() {
        long l = 3942562492L;
        for (int i = 9; -1 < i; --i) {
            for (int j = 0; j < 4; ++j) {
                Plant plant = this.O(i, j);
                if (plant == null) continue;
                String string = Reflection.getOrCreateKotlinClass(plant.getClass()).getSimpleName();
                int n = string != null ? string.hashCode() : 0;
                l = l << 5 ^ l >>> 27;
                l = l ^ ((long)i << 16 | (long)j) ^ (long)n;
            }
        }
        return l;
    }
    ```

    ``` JAVA
    // 这一块就是检验神秘仪式的主要部分

        private final void updateHashReversedCheck() {
        long l = this.lawnGroup.II();
        Object object = new long[]{6164215068710761199L, 5267579215675793589L, 6686816132172005092L, 2103177713450963003L, 9190546466593079779L, 3013428267688253536L, 1606735717653979810L, 2696897451854954540L, 3601412872638915688L, 5088639333237956963L};
        if (ArraysKt.contains(object, l)) {
           ... //跳出一个wsdx, 然后把僵尸全杀了
        }
    }
    ```

    ``` JAVA
    //生成flag的代码实在太长, 而且找到阵型后也不用我自己来解, 就不放在这里了...(反正就是7层解密堆堆乐
    ```

- 读完代码后我发现无论如何都得去找到这个阵型, 难倒我了  
  一顿暴力尝试无果后, 我终于想到题目中可能自带提示
    - 打开题目就有一个`help`按钮, 一开始没当回事, 现在终于注意到了
    > 当僵尸出现, 什么都不要做, 僵尸进入你的屋子, 你就赢了
    - 虽然与我理解的代码不同, 但我觉得存在就是有意义的, 我就在屏幕前坐了3分钟, 静静等僵尸走进去, 等失败提示跳出来, 甚至等其他没进去的僵尸全部走进去
    - 但是一直没有flag跳出来~~我好像被耍了~~

- 我尝试寻找下一个提示
    - 当我不知道第几遍打开这个文件夹, 我终于注意到了, 有张图片的名字很奇怪
    > what_is_this.png
    - 开始只是以为是戴夫说的话被作者不小心塞进来了, 但我现在走投无路了, 看什么都像是隐藏着惊天秘密, 我打开了它, 我猜对了:
    - 讲的是从左往右种植向日葵, 豌豆射手, 向日葵, 坚果墙
    - 这就是我要的阵型!!!

*摆出神秘仪式顺利过关, 得到flag*  
总结 : 多翻翻, 别死做

> 其实还有一次尝试, 是在我理解代码之前做的  
> 我用了作弊软件 cheat engine, 直接把hash值修改为目标值, 也成功触发了神秘仪式的检验, 但却没有跳出来flag, 当时我以为题目坏了都没怀疑自己作弊被发现了, 仔细一看, 写的是:      
> **YOU DIRTY HACKER!!!**  
> 倒也因祸得福让我发现检验和生成flag用的是不同的算法, 不过也没什么用就是了