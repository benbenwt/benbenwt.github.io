[TOC]
## 0901算法题
### 链表指定区间反转
```
import java.util.*;

/*
 * public class ListNode {
 *   int val;
 *   ListNode next = null;
 * }
 */

public class Solution {
    /**
     * 
     * @param head ListNode类 
     * @param m int整型 
     * @param n int整型 
     * @return ListNode类
     */
//     将输入的整个链表反转
    public void ReverseList(ListNode head){
        ListNode pre=null;
        ListNode cur=head;
      
        while(cur!=null){
            ListNode nextNode=cur.next;
            cur.next=pre;        
            pre=cur;
            cur=nextNode;
        }
    }

//     思路基本一致啊。
//     改成前后为null，提前存储4个连接点
    public ListNode reverseBetween (ListNode head, int m, int n) {
        // write code here
        ListNode vHead=new ListNode(-1);
        vHead.next=head;
        ListNode temp=vHead;
//         begin left right end
        for(int i=0;i<=m-2;i++)
        {
            temp=temp.next;
        }
        
        ListNode begin=temp;
        ListNode left=begin.next;
        
        for(int i=m-1;i<=n-1;i++)
        {
            temp=temp.next;
        }
        
        ListNode right=temp;
        ListNode end=right.next;
        
//         System.out.println("begin="+begin.val);
//         System.out.println("left="+left.val);
//         System.out.println("right="+right.val);
//         System.out.println("end="+end.val);
        
        begin.next=null;
        right.next=null;
        //         反转局部        

        ReverseList(left);
        // 连接反转部分的头和尾部
        begin.next=right;
        left.next=end;
        
        return vHead.next;

    }
}
```
## 二分查找-I
```
import java.util.*;


public class Solution {
    /**
     * 代码中的类名、方法名、参数名已经指定，请勿修改，直接返回方法规定的值即可
     *
     * 
     * @param nums int整型一维数组 
     * @param target int整型 
     * @return int整型
     */
    public int search (int[] nums, int target) {
        // write code here
        int left=0;
        int right=nums.length-1;
        int mid=(left+right)/2;
        while(left<=right){
            if(nums[mid]==target)
            {
                return mid;
            }
            if(nums[mid]>target){
//                 左边
                right=mid-1;
            }else{
                left=mid+1;
            }
            mid=(left+right)/2;
        }
        return -1;
    }
}
```