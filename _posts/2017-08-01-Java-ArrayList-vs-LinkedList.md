---
layout: post
title: Java ArrayList vs LinkedList
category: programming
---
When we are going to use a ```List```, we were taught to write the following the code:
```
List list = new ArrayList<>();
```
And to avoid another ```List``` implementation: ```LinkedList```, is this always true and why?

## JDK ArrayList internal
```ArrayList``` is an array based implementation, but array can only have fixed length, so what will happen is we try to add a new element? The solution is quite simple, just create a new array with longer length, and copy the elements from old array to the new array. We can check the JDK source code to have a look.

First, the ```add``` method:
```java
    public boolean add(E e) {
        ensureCapacityInternal(size + 1);
        elementData[size++] = e;
        return true;
    }
```
Then, ```ensureCapacityInternal``` will call ```grow```:
```java
    private void grow(int minCapacity) { //minCapacity = size + 1
        // This is the interesting part, the new array size is not just oldCapacity++, but oldCapacity + (oldCapacity >> 1)
        int oldCapacity = elementData.length;
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // minCapacity is usually close to size, so this is a win:
        elementData = Arrays.copyOf(elementData, newCapacity);
        //Notice that ```Arrays.copyOf``` is the core function and it has big performance influence on ArrayList!
    }
```
And then another question, how ```Arrays.copyOf``` is implemented?
First, from ```Arrays```:
```
    public static <T,U> T[] copyOf(U[] original, int newLength, Class<? extends T[]> newType) {
        @SuppressWarnings("unchecked")
        T[] copy = ((Object)newType == (Object)Object[].class)
            ? (T[]) new Object[newLength]
            : (T[]) Array.newInstance(newType.getComponentType(), newLength);
        System.arraycopy(original, 0, copy, 0,
                         Math.min(original.length, newLength)); //This line is the core!
        return copy;
    }
```
Then what is ```System.arraycopy```?
```java
    public static native void arraycopy(Object src,  int  srcPos,
                                        Object dest, int destPos,
                                        int length);
```
Sorry, guys, this is a JVM implemented method,  so we can't dive deeper, but I guess this method will finally invoke the memory copy system calls provided by OS.
Now, we come to our conclusion:
> The key to understand and implement ```ArrayList``` is ```System.arraycopy(Object srcArray, Object destArray...)```, and everytime when the array space is not enough, we need to create a new array and call ```System.arraycopy```. So if we know the final size, we should call ```ArrayList(int initialCapacity)``` to avoid unnecessary arraycopys.

## What about LinkedList?
First, go to the ```add(E e)``` method:
```java
    public boolean add(E e) {
        linkLast(e);
        return true;
    }
```
Then, go to the ```linkLast(e)```:
```java
    void linkLast(E e) {
        final Node<E> l = last;
        final Node<E> newNode = new Node<>(l, e, null);
        last = newNode;
        if (l == null)
            first = newNode;
        else
            l.next = newNode;
        size++;
        modCount++;
    }
```
We can see that the logic is quite simple campared to ```ArrayList```. It just new a new Node then add the Node to the last of the list.
## Add() mthod Benchmark time
We can already guess that the performance of the add method in LinkedList depends on ```new Node<>(l, e, null)```, and ArrayList depends on ```System.arraycopy()```, which one will win? That's do some simple tests.
```java
import java.util.ArrayList;
import java.util.LinkedList;
import java.util.List;

public class TestList {
    static int size = (int)1e4;
    static String e = "test element";

    public static void main(String[] args) {
        List<String> arrayListInit = new ArrayList<>(size);
        test("ArrayList with Init: ", arrayListInit);

        List<String> linkedList = new LinkedList<>();
        test("LinkedList: ", linkedList);

        List<String> arrayList = new ArrayList<>();
        test("ArrayList: ", arrayList);
    }

    public static void test(String message, List<String> list){
        long start = System.nanoTime();
        for(int i=0;i<size;i++){
            list.add(e);
        }
        double period = (System.nanoTime() - start)/1e9d;
        System.out.println(message + period);
    }
}
```
On my MacPro desktop, The output is:
```
ArrayList with Init: 0.001231222
LinkedList: 0.001673594
ArrayList:  0.001501807
```
If we change the size to **1e6**, then we get:
```
ArrayList with Init: 0.010108087
LinkedList: 0.028223336
ArrayList:  0.01934018
```
And change to **1e7**,
```
ArrayList with Init: 0.022215321
LinkedList: 5.321726898
ArrayList:  0.151722335
```
We can see that the add() method of ArrayList outperforms LinkedList, and why the performance gap wider when size grows? Because the number of arraycopy operations grow more slowly. The default capacity is 10:
```
private static final int DEFAULT_CAPACITY = 10;
```
And then the capacity grows by 1.5x times:
```
int newCapacity = oldCapacity + (oldCapacity >> 1);
```
We can also write some simple code to calculate how many array copy operations needed when the size grows:
```java
public class Capacity {
    public static void main(String[] args){
        int size = 1;
        while(size < (int)1e8){
            size = size * 10;

            int oldCapacity = 10;
            int count = 0;
            while(oldCapacity <= size){
                oldCapacity = oldCapacity + (oldCapacity >> 1);
                count ++;
            }
            System.out.println("reach size: " + size + ", need arraycopy count: " + (count - 1));
        }
    }
}
```
The output is:
```
reach size: 10, need arraycopy count: 0
reach size: 100, need arraycopy count: 5
reach size: 1000, need arraycopy count: 11
reach size: 10000, need arraycopy count: 17
reach size: 100000, need arraycopy count: 22
reach size: 1000000, need arraycopy count: 28
reach size: 10000000, need arraycopy count: 34
reach size: 100000000, need arraycopy count: 39
```
So, we can see that when size grow by 10x times, LinkedList will need to new 10x times new objects, but the number of array copy operations is just 6 or 5, in a constant manner.