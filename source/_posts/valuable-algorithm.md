---
title: 一些其他的算法题题解
categories: 算法
mathjax: true
---



## 字符串的split函数

* Split函数可以将一个字符串, 按照其中出现的某个字符`t`, 切割成若干的pieces.
* 实现可参考:

```cpp
vector<string> split(string str, char t) {
    str += t;
    vector<string> items;
    for (int i = 0; i < str.size(); i ++) {
      // 对于每一个i, 移动j, 直到移动到str[j] == t为止, 然后把扫过的部分放到数组中
        int j = i;
        string item;
        while (str[j] != t) item += str[j ++];
        i = j;
        items.push_back(item);
    }
    return items;
}
```

