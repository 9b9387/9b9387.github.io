---
layout: post
title: "Lua的continue实现"
data: 2017-04-28 14:37:00 +0800
---
Lua中没有continue关键字，但是有两种方法可以曲线救国。

1. 使用`while，break`

```
for i = 1, 5 do
    while true do

        if i == 3 then
            break
        end

        print(i)

        break
    end
end
```

打印结果

```
1
2
4
5
```

2. 使用`goto`关键字 需要Lua 5.2以上版本

```
for i = 1, 5 do

    if i == 3 then
        goto continue
    end

    print(i)

    ::continue::
end
```

打印结果

```
1
2
4
5
```
