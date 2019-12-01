---
layout: post
title: "LC-median-of-two-sorted-arrays"
category: programming
---
4. Median of Two Sorted Arrays
Hard

There are two sorted arrays nums1 and nums2 of size m and n respectively.

Find the median of the two sorted arrays. The overall run time complexity should be O(log (m+n)).

You may assume nums1 and nums2 cannot be both empty.

Example 1:

nums1 = [1, 3]
nums2 = [2]

The median is 2.0
Example 2:

nums1 = [1, 2]
nums2 = [3, 4]

The median is (2 + 3)/2 = 2.5

```
public class LcMedianOfTwoSortedArrays {
    public double findMedianSortedArrays(int[] nums1, int[] nums2) {
        return findMedianSortedArrays2(nums1, 0, nums1.length - 1,
        nums2, 0, nums2.length - 1);
    }

    public static double findMedianSortedArrays2(int[] nums1, int l1, int h1,
                                          int[] nums2, int l2, int h2) {
        if(h1-l1 < h2-l2){
            return findMedianSortedArrays2(nums2, l2, h2,
                    nums1, l1, h1);
        }

        if(h1 - l1 == 1){
            if(h2 - l2 < 0) {
                return (nums1[l1] + nums1[h1]) / 2d;
            }
        }
        if(h1 - l1 == 0){
            if(h2 - l2 == 0) {
                return (nums1[l1] + nums2[l2])/2d;
            }
            if(h2 - l2 < 0) {
                return nums1[l1];
            }
        }

        if(nums1[l1] <= nums2[l2]){
            l1 += 1;
        }else{
            l2 += 1;
            if(l2 > h2){
                h1 -= 1;
                return findMedianSortedArrays2(nums1, l1, h1,
                                                nums2, l2, h2);
            }
        }
        if(nums1[h1] >= nums2[h2]){
            h1 -= 1;
            if(h1 < l1){
                return findMedianSortedArrays2(nums1, l1, h1,
                        nums2, l2, h2);
            }
        }else{
            h2 -= 1;
            if(h2 < l2){
                return findMedianSortedArrays2(nums1, l1, h1,
                        nums2, l2, h2);
            }
        }
        //edge case


        return findMedianSortedArrays2(nums1, l1, h1,
                                        nums2, l2, h2);
    }

    public static void main(String... args){
        int[] nums1 = new int[]{1,2,3};
        int[] nums2 = new int[]{2,4,6};
        System.out.println(findMedianSortedArrays2(nums1, 0, nums1.length -1,
        nums2, 0, nums2.length-1));
    }
}
```
