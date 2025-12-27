# base_emoji
*安卓逆向题*

- 开始给我一个 `apk` 文件着实吓了我一跳, 查阅资料后下载了 **jadx-gui**  
  文件拖进去直接就能看到代码还是很方便的

- 主要部分:
``` java

...

public String base_emoji = "🎂🎃🎄🎅🎆🎇🎈🎉🎊🎋🎌🎍🎎🎏🎐🎑🎒🎓🎔🎕🎖🎗🎘🎙🎚🎛🎜🎝🎞🎟🎠🎡🎢🎣🎤🎥🎦🎧🎨🎩🎪🎫🎬🎭🎮🎯🎰🎱🎲🎳🎴🎵🎶🎷🎸🎹🎺🎻🎼🎽🎾🏀🏁🏂🏃🏄🏅🏆🏇🏈🏉🏊🏋🏌🏍🏎🏏🏐🏑🏒🏓🏔🏕🏖🏗🏘🏙🏚🏛🏜🏝🏞🏟🏠🏡🏢🏣🏤🏥🏦🏧🏨🏩🏪🏫🏬🏭🏮🏯🏰🏱🏲🏳🏴🏵🏶🏷🏸🏹🏺🏻🏼🏽🏾🐀🐁🐂🐃";

            @Override // android.view.View.OnClickListener
            public void onClick(View v) {
                try {
                    String input_str = myinput.getText().toString();
                    int len = input_str.length() + (7 - (input_str.length() % 7));
                    byte[] input_pad = new byte[len];
                    System.arraycopy(input_str.getBytes(), 0, input_pad, 0, input_str.length());
                    String enc = new String(encode(input_pad, input_pad.length));
                    myenc.setText(enc);
                    int ret = enc.compareTo("🏙🎦🏔🎊🎗🎕🏰🏅🏱🎾🏞🏑🏙🏬🏜🏍🏵🎱🏛🏉🏊🎝🎗🏁🎳🎻🏚🏱🎅🎅🏏🏭🏬🎳🎝🎎🏚🏫🎂🎂");
                    if (ret != 0) {
                        showAlertDialog("wrong", "Result");
                    } else {
                        showAlertDialog("right", "Result");
                    }
                } catch (Exception e) {
                    showAlertDialog(e.toString(), "error");
                }
            }

            public String encode(byte[] m, int len) throws UnsupportedEncodingException {
                int i = 0;
                int j = 0;
                int len1 = (len / 7) * 8;
                int[] index = new int[len1];
                while (i < len1) {
                    // 每7个字节转换为8个7位索引
                    index[i] = m[j] & ByteCompanionObject.MAX_VALUE;
                    index[i + 1] = ((m[j] << 7) | (m[j + 1] >> 1)) & WorkQueueKt.MASK;
                    index[i + 2] = ((m[j + 1] << 6) | (m[j + 2] >> 2)) & WorkQueueKt.MASK;
                    index[i + 3] = ((m[j + 2] << 5) | (m[j + 3] >> 3)) & WorkQueueKt.MASK;
                    index[i + 4] = ((m[j + 3] << 4) | (m[j + 4] >> 4)) & WorkQueueKt.MASK;
                    index[i + 5] = ((m[j + 4] << 3) | (m[j + 5] >> 5)) & WorkQueueKt.MASK;
                    index[i + 6] = ((m[j + 5] << 2) | (m[j + 6] >> 6)) & WorkQueueKt.MASK;
                    index[i + 7] = (m[j + 6] << 1) & WorkQueueKt.MASK;
                    i += 8;
                    j += 7;
                }
                try {
                    // 将索引映射为emoji字符
                    byte[] byte_base_emoji = this.base_emoji.getBytes("unicode");
                    byte[] out = new byte[(len1 * 4) + 2];
                    out[1] = -1;
                    out[0] = -2;
                    // 每个索引对应一个emoji字符(4字节)
                    for (int i2 = 0; i2 < len1; i2++) {
                        out[(i2 * 4) + 2] = byte_base_emoji[(index[i2] * 4) + 2];
                        out[(i2 * 4) + 2 + 1] = byte_base_emoji[(index[i2] * 4) + 2 + 1];
                        out[(i2 * 4) + 2 + 2] = byte_base_emoji[(index[i2] * 4) + 2 + 2];
                        out[(i2 * 4) + 2 + 3] = byte_base_emoji[(index[i2] * 4) + 2 + 3];
                    }
                    String output = new String(out, "unicode");
                    return output;
                } catch (Exception e) {
                    showAlertDialog(e.toString(), "error");
                    return null;
                }
            }

...

```

- 说实话, 这里的加密处理对于我来说过于复杂了, 无奈求助AI
``` python
def decode_emoji(encoded_str):
    # base_emoji映射
    base_emoji = "🎂🎃🎄🎅🎆🎇🎈🎉🎊🎋🎌🎍🎎🎏🎐🎑🎒🎓🎔🎕🎖🎗🎘🎙🎚🎛🎜🎝🎞🎟🎠🎡🎢🎣🎤🎥🎦🎧🎨🎩🎪🎫🎬🎭🎮🎯🎰🎱🎲🎳🎴🎵🎶🎷🎸🎹🎺🎻🎼🎽🎾🏀🏁🏂🏃🏄🏅🏆🏇🏈🏉🏊🏋🏌🏍🏎🏏🏐🏑🏒🏓🏔🏕🏖🏗🏘🏙🏚🏛🏜🏝🏞🏟🏠🏡🏢🏣🏤🏥🏦🏧🏨🏩🏪🏫🏬🏭🏮🏯🏰🏱🏲🏳🏴🏵🏶🏷🏸🏹🏺🏻🏼🏽🏾🐀🐁🐂🐃"
    
    # 创建emoji到索引的映射
    emoji_to_index = {emoji: i for i, emoji in enumerate(base_emoji[:128])}
    
    # 将加密字符串转换为索引
    indices = []
    for char in encoded_str:
        indices.append(emoji_to_index[char])
    
    # 解密：每8个索引恢复7个字节
    result_bytes = bytearray()
    
    for i in range(0, len(indices), 8):
        i0, i1, i2, i3, i4, i5, i6, i7 = indices[i:i+8]
        
        # 恢复7个字节
        m0 = i0
        m1 = (i1 << 1) | (i2 >> 6)
        m2 = ((i2 & 0x3F) << 2) | (i3 >> 5)
        m3 = ((i3 & 0x1F) << 3) | (i4 >> 4)
        m4 = ((i4 & 0x0F) << 4) | (i5 >> 3)
        m5 = ((i5 & 0x07) << 5) | ((i6 >> 2) & 0x1F)
        m6 = ((i6 & 0x03) << 6) | (i7 >> 1)
        
        result_bytes.extend([m0, m1, m2, m3, m4, m5, m6])
    
    # 转换为字符串
    result = result_bytes.decode('utf-8').rstrip('\x00')
    return result
```

*运行程序得到flag*  
总结 : 见识到了新东西
