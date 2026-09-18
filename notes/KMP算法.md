---
created_at: 2026-03-20
updated_at: 2026-09-18
tags:
    - 计算机基础
archived: true
---

[数据结构](./数据结构.md) / 4. 字符串与特殊矩阵压缩 / 4.1 字符串匹配

# KMP算法

基于模式串确定next数组  
利用next数组完成字符串匹配  
在匹配过程中发生字符不匹配情况时, next数组用于帮助确定下一次匹配的位置

1. 遍历模式串, 获取每个字符前面的内容, 根据前, 后缀相同的最大长度填写对应next中的值
2. 将不匹配的模式串的下标所对应的next数组中的数字提取出来记为a
3. 让模式串下标为a的元素对应字符串原本失配的位置开始下一轮匹配

```C
#include <stdio.h>
#include <string.h>

void getNext(char* pattern, int* next){
    int m = strlen(pattern);
    int i = 0;
    int j = -1;
    next[0] = -1;
    while(i < m){
        if(j == -1 || pattern[i] == pattern[j]){
            i++;
            j++;
            next[i] = j;
        }
        else{
            j = next[j];
        }
    }
}

int kmp(char* str, char* pattern){
    int i = 0;
    int j = 0;
    int next[100];
    getNext(pattern, next);
    int n = strlen(str);
    int m = strlen(pattern);

    while(i < n && j < m){
        if(j == -1 || str[i] == pattern[j]){
            i++;
            j++;
        }
        else{
            j = next[j];
        }
    }

    if(j == m){
        return i - j;
    }
    else{
        return -1;
    }
}

int main(){
    char* str = "abaabaabacacaabaabcc";
    char* pattern = "abaabc";
    printf("%d\n", kmp(str, pattern));
    return 0;
}
```

## 相关笔记

- 上一篇:[朴素匹配](./朴素匹配.md)
- 下一篇:[稀疏矩阵](./稀疏矩阵.md)
- 返回索引:[数据结构](./数据结构.md)
