---
layout: post
title: Java simple dead lock example
category: programming
---
This is just a simple example to demonstrate dead lock.

```

package com.test;

public class DeadLockTest {
    public static void main(String[] args){
        final Object a = new Object();
        final Object b = new Object();

        Thread ta = new Thread(){
            @Override
            public void run(){
                synchronized(a){
                    System.out.println("a");
                    synchronized (b){
                        System.out.println("a & b");
                    }
                }
            }
        };

        Thread tb = new Thread(){
            @Override
            public void run(){
                synchronized(b){
                    System.out.println("b");
                    synchronized (a){
                        System.out.println("b & a");
                    }
                }
            }
        };

        ta.start();
        tb.start();
    }
}
```
Output
```
a
b
```
Then hang forever. 
Note: this example can not cause deadlock every time, but it seems to be. +_+
