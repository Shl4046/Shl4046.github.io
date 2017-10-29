---
layout: default
title:  "Merge_Sorted_Array"
category: "lcode"
---
>     {{ page.date | date: "%Y-%m-%d," }} {{ page.content | number_of_words | append: "chars"}}
>     {{ page.tags }}

Given two sorted integer arrays nums1 and nums2, merge nums2 into nums1 as one sorted array.

Note:
You may assume that nums1 has enough space (size that is greater or equal to m + n) to hold additional elements from nums2. The number of elements initialized in nums1 and nums2 are m and n respectively.

----------

``` java

class Solution {
    public void merge(int[] nums1, int m, int[] nums2, int n) {
        for (int i = 0;i<n;i++) {
            nums1[m] = nums2[i];
            m++;
        }
    }
}

```