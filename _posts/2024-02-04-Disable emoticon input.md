---
layout: post
title: 禁止使用edittext表情
tags: kotlin\JAVA
author: hulu-knight
date: 2024-02-04 23:15 +0800
---



方法一：

```
val dest = s.toString().toCharArray()
// 已经输入的文字
for (index in dest.indices) {
    val type = Character.getType(dest[index])
    if (type == Character.SURROGATE.toInt() || type == Character.OTHER_SYMBOL.toInt()) {
        // 禁止表情
        isAnswerFormatError = true
    }
}
```

方法二：用正则

```
	/**
	 * 判断是否是Emoji
	 *
	 * @param codePoint 比较的单个字符
	 * @return
	 */
	private static boolean isEmojiCharacter(char codePoint) {
		return (codePoint == 0x0) || (codePoint == 0x9) || (codePoint == 0xA) || (codePoint == 0xD)
				|| ((codePoint >= 0x20) && (codePoint <= 0xD7FF)) || ((codePoint >= 0xE000) && (codePoint <= 0xFFFD))
				|| ((codePoint >= 0x10000) && (codePoint <= 0x10FFFF));
	}
```

```

/**
 * 正则判断emoji表情
 *
 * @param input
 * @return
 */
private boolean isEmoji(String input) {
    Pattern p = Pattern.compile("[\ud83c\udc00-\ud83c\udfff]|[\ud83d\udc00-\ud83d\udfff]|[\ud83e\udc00-\ud83e\udfff]" +
            "|[\u2100-\u32ff]|[\u0030-\u007f][\u20d0-\u20ff]|[\u0080-\u00ff]");
    Matcher m = p.matcher(input);
    return m.find();
}
```



