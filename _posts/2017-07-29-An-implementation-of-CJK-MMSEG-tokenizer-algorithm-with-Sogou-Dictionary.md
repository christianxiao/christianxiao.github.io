---
layout: post
title: An implementation of CJK MMSEG tokenizer algorithm with Sogou Dictionary.
category: programming
---


An implementation of CJK [MMSEG](http://technology.chtsai.org/mmseg/) tokenizer algorithm with Sogou Chinese Dictionary.

Implementation Details:
* Word dictionary is based on [Patricia Trie](https://github.com/rkapsi/patricia-trie), so the dictionary is efficient when dealing with CJK languages and updating word dictionary dinamically is also possible.
* For English and special characters, [unicode uax #29](http://unicode.org/reports/tr29/) is used, and the implementation is JDK's java.text package. So given a piece of text, first we use [unicode uax #29](http://unicode.org/reports/tr29/) to tokenize to get English words, special characters and CJK sentences, then we use [MMSEG](http://technology.chtsai.org/mmseg/) to tokenize CJK words based on Sogou Chineses dictionary.

# How to use:
Cjk-mmseg is hosted on **Jcenter**:

Gradle:
```
compile 'com.profullstack:cjk-mmseg:0.0.1'
```
Maven:
```
<dependency>
  <groupId>com.profullstack</groupId>
  <artifactId>cjk-mmseg</artifactId>
  <version>0.0.1</version>
  <type>pom</type>
</dependency>
```
Then check the test code in CjkMmsegTest.java:
```java
        CjkMmseg seg = new CjkMmseg();
        String s = "My email address is christian.xiao@outlook.com。我的用户名是christian,我的邮箱是christian.xiao@outlook.com.";
        Reader r = new StringReader(s);
        seg.setReader(r);
        Word w;
        while((w = seg.nextWord()) != null){
            System.out.println(w);
        }
        seg.close();
```
Output:
```java
text: My startOffset: 0 endOffset: 1
text: email startOffset: 3 endOffset: 7
text: address startOffset: 9 endOffset: 15
text: is startOffset: 17 endOffset: 18
text: christian.xiao startOffset: 20 endOffset: 33
text: outlook.com startOffset: 35 endOffset: 45
text: 我的 startOffset: 47 endOffset: 48
text: 用户名 startOffset: 49 endOffset: 51
text: 是 startOffset: 52 endOffset: 52
text: christian startOffset: 53 endOffset: 61
text: 我的邮箱 startOffset: 63 endOffset: 66
text: 是 startOffset: 67 endOffset: 67
text: christian.xiao startOffset: 68 endOffset: 81
text: outlook.com startOffset: 83 endOffset: 93
```
