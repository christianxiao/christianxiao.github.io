---
layout: post
title: "LC-Basic-quicksort-implement"
category: programming
---
Check out wikipedia
```
import java.util.Arrays;

public class Test2 {
    public static  void qs(int[] a, int low, int high){
        if(low >= high){
            return;
        }
        int k = p(a, low, high);
        qs(a,low,k);
        qs(a,k+1,high);
    }
    public static int p(int[] a, int low, int high){
        if(low >= high){
            return low;
        }
        int p = a[(low+high)/2];
        int l = low -1;
        int h = high + 1;
        while (l < h) {
            do {
                l += 1;
            } while (a[l] < p && l < h);
            do {
                h -= 1;
            } while (a[h] > p && h >= l);
            if(l<h){
                int temp = a[l];
                a[l] = a[h];
                a[h] = temp;
            }
        }

        return h;
    }


    public static void main(String[] args){
        int[] a = new int[]{3,2,1,6,8,4};
        qs(a, 0, a.length -1);
        System.out.println(Arrays.toString(a));
    }
}
```
