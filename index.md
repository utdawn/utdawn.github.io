---
layout: post
title: '一些奇奇怪怪的题'
date: 2022-07-23
author: utdawn
# cover: 'http://on2171g4d.bkt.clouddn.com/jekyll-banner.png'
tags: leetcode
---

#### LC 373 [查找和最小的 K 对数字](https://leetcode.cn/problems/find-k-pairs-with-smallest-sums/)（优先队列+去重）

```c++
class Solution {
public:
    struct node {
        int i, j;
        int x, y;
        bool operator < (const node & a) const {
		    return x + y > a.x + a.y;
	    }
    };
    vector<vector<int>> kSmallestPairs(vector<int>& nums1, vector<int>& nums2, int k) {
        vector<vector<int>> ans;
        priority_queue<node> q;
        for(int i = 0; i < nums1.size() && i < k; i++) {
            node tmp;
            tmp.i = i; tmp.j = 0;
            tmp.x = nums1[i]; tmp.y = nums2[0];
            q.push(tmp);
        }
        while(k-- && !q.empty()) {
            node tmp = q.top();
            q.pop();
            ans.push_back({tmp.x, tmp.y});
            if(tmp.j < nums2.size() - 1) {
                tmp.y = nums2[++tmp.j];
                q.push(tmp);
            }
        }
        return ans;
    }
};
```



#### LC 4 [寻找两个正序数组的中位数](https://leetcode.cn/problems/median-of-two-sorted-arrays/)（https://blog.csdn.net/weixin_53286472/article/details/124025466）

```c++
class Solution {

    public double findMedianSortedArrays(int[] nums1, int[] nums2) {
        int m = nums1.length;
        int n = nums2.length;
        int left = (m+n+1)>>1;
        int right = (m+n+2)>>1;
        return (findkth(nums1,0,nums2,0,left)+findkth(nums1,0,nums2,0,right))/2.0;//注意返回d类型为double，所以要除以2.0
    }
    
    /**
     * 在给定的数组中查找第K个数
     * @param nums1 第一个数组
     * @param i 第一个数组的起始位置
     * @param nums2 第二个数组
     * @param j 第二个数组的起始位置
     * @param k 查找的第K个数
     * @return
     */
    public int findkth(int[] nums1, int i ,int[] nums2, int j ,int k){
        if (i>=nums1.length) return nums2[j+k-1];//注意落到下标上是：数组起始位置+k-1
        if (j>=nums2.length) return nums1[i+k-1];
        if (k==1) return Math.min(nums1[i],nums2[j]);
        int tmp1 = i+k/2-1<nums1.length?nums1[i+k/2-1]:Integer.MAX_VALUE;//i或j超出范围则直接赋为最大值，方便后续比较
        int tmp2 = j+k/2-1<nums2.length?nums2[j+k/2-1]:Integer.MAX_VALUE;
        return tmp1<tmp2 ? findkth(nums1,i+k/2,nums2,j,k-k/2):findkth(nums1,i,nums2,j+k/2,k-k/2);
    }
    
}


class Solution {
public:
    int getKthElement(const vector<int>& nums1, const vector<int>& nums2, int k) {
        /* 主要思路：要找到第 k (k>1) 小的元素，那么就取 pivot1 = nums1[k/2-1] 和 pivot2 = nums2[k/2-1] 进行比较
         * 这里的 "/" 表示整除
         * nums1 中小于等于 pivot1 的元素有 nums1[0 .. k/2-2] 共计 k/2-1 个
         * nums2 中小于等于 pivot2 的元素有 nums2[0 .. k/2-2] 共计 k/2-1 个
         * 取 pivot = min(pivot1, pivot2)，两个数组中小于等于 pivot 的元素共计不会超过 (k/2-1) + (k/2-1) <= k-2 个
         * 这样 pivot 本身最大也只能是第 k-1 小的元素
         * 如果 pivot = pivot1，那么 nums1[0 .. k/2-1] 都不可能是第 k 小的元素。把这些元素全部 "删除"，剩下的作为新的 nums1 数组
         * 如果 pivot = pivot2，那么 nums2[0 .. k/2-1] 都不可能是第 k 小的元素。把这些元素全部 "删除"，剩下的作为新的 nums2 数组
         * 由于我们 "删除" 了一些元素（这些元素都比第 k 小的元素要小），因此需要修改 k 的值，减去删除的数的个数
         */

        int m = nums1.size();
        int n = nums2.size();
        int index1 = 0, index2 = 0;

        while (true) {
            // 边界情况
            if (index1 == m) {
                return nums2[index2 + k - 1];
            }
            if (index2 == n) {
                return nums1[index1 + k - 1];
            }
            if (k == 1) {
                return min(nums1[index1], nums2[index2]);
            }

            // 正常情况
            int newIndex1 = min(index1 + k / 2 - 1, m - 1);
            int newIndex2 = min(index2 + k / 2 - 1, n - 1);
            int pivot1 = nums1[newIndex1];
            int pivot2 = nums2[newIndex2];
            if (pivot1 <= pivot2) {
                k -= newIndex1 - index1 + 1;
                index1 = newIndex1 + 1;
            }
            else {
                k -= newIndex2 - index2 + 1;
                index2 = newIndex2 + 1;
            }
        }
    }

    double findMedianSortedArrays(vector<int>& nums1, vector<int>& nums2) {
        int totalLength = nums1.size() + nums2.size();
        if (totalLength % 2 == 1) {
            return getKthElement(nums1, nums2, (totalLength + 1) / 2);
        }
        else {
            return (getKthElement(nums1, nums2, totalLength / 2) + getKthElement(nums1, nums2, totalLength / 2 + 1)) / 2.0;
        }
    }
};
```



#### 快排第K大

```c++
class Solution {
private:
    int res=0;
    int pos;
public:
    int findKthLargest(vector<int>& nums, int k) {
        int length=nums.size();
        pos=length-k;
        quicksort(nums,0,length-1);
        return res;
    }

    void quicksort(vector<int>& nums,int low,int high){
        if(low<=high){
            int pivot=partition(nums,low,high);
            if(pivot==pos){
                res=nums[pivot];
                return;
            }
            else if(pivot<pos)
                quicksort(nums,pivot+1,high);
            else
                quicksort(nums,low,pivot-1);
        }
    }

    int partition(vector<int>& nums,int low,int high){
        int pivot=nums[low];
        while(low<high){
            while(nums[high]>=pivot&&high>low)
                high--;
            nums[low]=nums[high];
            while(nums[low]<=pivot&&low<high)
                low++;
            nums[high]=nums[low];
        }
        nums[low]=pivot;
        return low;
    }
};
```



#### 排序链表（链表的合并，反转）

```C++
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode() : val(0), next(nullptr) {}
 *     ListNode(int x) : val(x), next(nullptr) {}
 *     ListNode(int x, ListNode *next) : val(x), next(next) {}
 * };
 */
class Solution {
public:

    ListNode* mergeList(ListNode* a, ListNode* b) {
        ListNode* preHead = new ListNode(-100005);
        ListNode* p = preHead;
        ListNode* p1 = a;
        ListNode* p2 = b;
        while(p1 != NULL && p2 != NULL) {
            if(p1 -> val > p2 -> val) {
                p -> next = p2;
                p2 = p2 -> next;
            }else {
                p -> next = p1;
                p1 = p1 -> next;
            }
            p = p -> next;
        } 
        if(p1 != NULL) {
            p -> next = p1;
        }else {
            p -> next = p2;
        }
        return preHead -> next;
    }

    ListNode* sortList(ListNode* head) {
        int len = 0;
        if(head == NULL)
            return NULL;
        ListNode* p = head;
        while(p != NULL) {
            len++;
            p = p -> next;
        }
        ListNode* preHead = new ListNode(-100005, head);
        for(int i = 1; i < len; i = i * 2) {
            ListNode* curr = preHead->next;
            ListNode* pre = preHead;
            while(curr != NULL) {
                ListNode* head1 = curr;
                for(int j = 1; j < i && curr->next != NULL; j++) {
                    curr = curr -> next;
                }
                ListNode* head2 = curr-> next;
                curr -> next = NULL;
                curr = head2;
                for(int j = 1; j < i && curr != NULL && curr->next != NULL; j++) {
                    curr = curr -> next;
                }
                ListNode* t = NULL;
                if(curr != NULL) {
                    t = curr -> next;
                    curr -> next = NULL;
                    // curr = t;
                }

                ListNode* merge = mergeList(head1, head2);
                pre -> next = merge;
                while(pre->next != NULL)
                    pre = pre -> next;
                curr  = t;
            }
                
        }
        return preHead -> next;
    }
};
```



#### 树状数组求逆序对

```c++
class BIT {
private:
    vector<int> tree;
    int n;

public:
    BIT(int _n): n(_n), tree(_n + 1) {}

    static int lowbit(int x) {
        return x & (-x);
    }

    int query(int x) {
        int ret = 0;
        while (x) {
            ret += tree[x];
            x -= lowbit(x);
        }
        return ret;
    }

    void update(int x) {
        while (x <= n) {
            ++tree[x];
            x += lowbit(x);
        }
    }
};

class Solution {
public:
    int reversePairs(vector<int>& nums) {
        int n = nums.size();
        vector<int> tmp = nums;
        // 离散化
        sort(tmp.begin(), tmp.end());
        for (int& num: nums) {
            num = lower_bound(tmp.begin(), tmp.end(), num) - tmp.begin() + 1;
        }
        // 树状数组统计逆序对
        BIT bit(n);
        int ans = 0;
        for (int i = n - 1; i >= 0; --i) {
            ans += bit.query(nums[i] - 1);
            bit.update(nums[i]);
        }
        return ans;
    }
};
```



#### KMP（字符串匹配）

```python
class Solution:
    # 根据匹配串获取next数组
    def getNext(self, p: str):
        # 加上空格是为了下标从1开始
        m = len(p)
        p = " " + p
        # i从下标2开始
        i, j = 2, 0
        next = [0] * (m + 1)
        while i <= m:
            while j and p[i] != p[j + 1]: j = next[j]
            if p[i] == p[j + 1]: j += 1
            next[i] = j
            i += 1
        return next
 
 
if __name__ == '__main__':
    s, p = "abcdabcaabcabc", "abcabc"
    n, m = len(s), len(p)
    i, j = 1, 0
    next = Solution().getNext(p)
    print(next)
    s = " " + s
    p = " " + p
    while i <= n:
        if j and s[i] != p[j + 1]: j = next[j]
        if s[i] == p[j + 1]: j += 1
        # 主串与模式串匹配成功
        if j == m:
            print(i - m)
            # 回到上一次匹配的位置
            j = next[j]
        i += 1
```

