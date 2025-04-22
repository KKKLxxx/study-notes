# String.split()用法

先贴上源码

```
public String[] split(String regex, int limit) {
        /* fastpath if the regex is a
         (1)one-char String and this character is not one of the
            RegEx's meta characters ".$|()[{^?*+\\", or
         (2)two-char String and the first char is the backslash and
            the second is not the ascii digit or ascii letter.
         */
        char ch = 0;
        if (((regex.value.length == 1 &&
             ".$|()[{^?*+\\".indexOf(ch = regex.charAt(0)) == -1) ||
             (regex.length() == 2 &&
              regex.charAt(0) == '\\' &&
              (((ch = regex.charAt(1))-'0')|('9'-ch)) < 0 &&
              ((ch-'a')|('z'-ch)) < 0 &&
              ((ch-'A')|('Z'-ch)) < 0)) &&
            (ch < Character.MIN_HIGH_SURROGATE ||
             ch > Character.MAX_LOW_SURROGATE))
        {
            int off = 0;
            int next = 0;
            boolean limited = limit > 0;
            ArrayList<String> list = new ArrayList<>();
            while ((next = indexOf(ch, off)) != -1) {
                if (!limited || list.size() < limit - 1) {
                    list.add(substring(off, next));
                    off = next + 1;
                } else {    // last one
                    //assert (list.size() == limit - 1);
                    list.add(substring(off, value.length));
                    off = value.length;
                    break;
                }
            }
            // If no match was found, return this
            if (off == 0)
                return new String[]{this};

            // Add remaining segment
            if (!limited || list.size() < limit)
                list.add(substring(off, value.length));

            // Construct result
            int resultSize = list.size();
            if (limit == 0) {
                while (resultSize > 0 && list.get(resultSize - 1).isEmpty()) {
                    resultSize--;
                }
            }
            String[] result = new String[resultSize];
            return list.subList(0, resultSize).toArray(result);
        }
        return Pattern.compile(regex).split(this, limit);
    }
```

比较值得注意的有两点

## 一、转义字符

> ".$|()\[{^?*+\\\"

在源码中可以找到上面这行字符串，这里规定了一些字符需要通过转义符"\\\"来使用，一共有12个符号

"."   "$"   "|"  "("  ")"   "\["  "{"  "^"   "?"  "*"   "+"  "\\\"

挑两个具有代表性的来讲

### 1、"."

"."在IP中是非常常见的，可以直接用一个IP举例

```
String IP = "127.0.0.1";
System.out.println(IP);
String\[\] strs = IP.split("\\\.");for (String s : strs) {
    System.out.println(s);
}
```

 输出结果如下

```
127.0.0.1
127
0
0
1
```

除了最后一个"\\\"比较特殊，剩下的用法都是一模一样的了

### 2、"\\\"

其实用法也是一样的，只不过这个有两个字符，所以比较特殊，同样也是举个例子

```
String str = "\\\a\\\b\\\c";
System.out.println(str);
String\[\] strs = str.split("\\\\\\");for (String s : strs) {
    System.out.println(s);
}
```

结果如下

```
\\a\\b\\c

a
b
c
```

## 二、limit可选参数

一般情况下是不需要输入这个参数的，如果没有输入**默认为0**

这个参数的含义就是：**如果limit > 0，那么就要限制匹配次数为（limit - 1）次**

再用IP举例，这里将limit设置为2

```
String IP = "127.0.0.1";
System.out.println(IP);
String[] strs = IP.split("\\.", 2);
for (String s : strs) {
    System.out.println(s);
}
```

结果如下

```
127.0.0.1
127
0.0.1
```

可以看到，只匹配了一次"."。大家可以将limit改为其他值深入体会下