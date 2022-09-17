[TOC]
# 0914 算法题
## leetcode 179 最大数
>给定一个nums数组，请重新排列每个数的顺序，使得其排列组成一个最大的数
>主要熟悉冒泡排序、String的compareTo函数，它是从高位到低位逐位比较大小。
>还有处理00这种情况

```
import java.util.*;
import java.lang.*;
class Solution {
    public String largestNumber(int[] nums) {
        // 冒泡
        int left=0;
        int temp=0;
        for(int i=0;i<nums.length-1;i++)
        {
            for(int j=nums.length-1;j>left;j--)
            {
                // j 大于 j-1
                if(((""+nums[j]+nums[j-1]).compareTo(""+nums[j-1]+nums[j]))>0)
                {
                    temp=nums[j];
                    nums[j]=nums[j-1];
                    nums[j-1]=temp;
                }
            }
        }
        StringBuilder result=new StringBuilder("");
        for(int i=0;i<nums.length;i++)
        {
            result.append(nums[i]);
        }
        //00
        if(result.substring(0,1).equals("0"))
        {
            return "0";
        }
        return result.toString();
    }
}
```

## leetcode nums数组两个数字和为target
>1遍历数组，向后查找是否有与当前下标数和为target的。为了加速查找，可以先排序，在进行查找。时间复杂度为O(nlogn)
>2使用hash表存储nums[i],每次遍历到一个元素，检索hash表中是否有target-nums[i]，如果有则返回该匹配的结果，否则将其放入hash表中。时间复杂度为O(n),空间复杂度为O(n)。
>判断某个值存不存在，一般都有空间换时间的替代解法，使用hashMap，set等。本题是判断target-num[i]是否存在。

## 排序复习
### 冒泡
```
for(int i=0;i<nums.length;i++)
{
    int left=0;
    int temp=0;
    for(int j=nums.length-1;j>left;j--)
    {
        temp=nums[j];
        nums[j]=nums[j-1];
        nums[j-1]=temp;
    }
}
```
### 快速排序
```
#使用了nums[left]作为划分界限，并从nums[left+1]开始排序，最终再把nums[left]和nums[j]交换。

void quicksort(int[] nums,int left,int right)
{
    <!-- 划分 -->
    int i=left;
    int j=right+1;
    int temp=0;
    while(true)
    {
        while(nums[++i]<nums[left] & i<right);
        while(nums[--j]>nums[left]);
        if(i>=j)
        {
            break;
        }
        temp=nums[i];
        nums[i]=nums[j];
        nums[j]=temp;
    }
    temp=nums[left];
    nums[left]=nums[j];
    nums[j]=temp;

    quicksort(nums,left,j);
    quicksort(nums,j+1,right);
}
```

### 堆排序
```
#java 用优先队列实现
#默认是小根堆
PriorityQueue<Integer> queue = new PriorityQueue<>();

# 大根堆
PriorityQueue<Integer> queue = new PriorityQueue<>(new Comparator<Integer>() {
    @Override
    public int compare(Integer o1, Integer o2) {
        return o2.compareTo(o1);
    }
});

#简化写法
PriorityQueue<Integer> queue = new PriorityQueue<>((o1, o2)->o2.compareTo(o1));
Queue<Integer> queue = new PriorityQueue<>(Collections.reverseOrder());

queue.offer(12);
queue.poll();
```

## 牛客 DP18 滑雪
>动态规划 极值类型，求出得到极值的path。这题关键点是确定起点和计算方向，起点是最低的点，计算方向是按照高度增加。
```
import java.util.*;

//     
class HeightAxis{
    public int height;
    public int i;
    public int j;

    public  HeightAxis(int height,int i,int j){
        this.height=height;
        this.i=i;
        this.j=j;
    }
}

public class Main{
//     选出最长的路径，状态值为当前的max长度，依赖为取决于三个方向。起始点为当一个点的四周没有比他低的（排除来路），其max=0。如何取到所有滑道。
    public static void main(String[] args){
        Scanner scanner=new Scanner(System.in);
        int n=0;int m=0;
        n=scanner.nextInt();
        m=scanner.nextInt();
        PriorityQueue<HeightAxis> heightQueue=heightQueue=new PriorityQueue<HeightAxis>((o1,o2)->(o1.height-o2.height));;
        int [][] matrix=new int[n][m];
        for(int i=0;i<n;i++)
        {
            for(int j=0;j<m;j++)
            {
                matrix[i][j]=scanner.nextInt();
                HeightAxis h=new HeightAxis(matrix[i][j],i,j);
                heightQueue.offer(h);
            }
        }
        Main.answer(matrix,n,m,heightQueue);
    }

    public static void answer(int [][] matrix,int n,int m,PriorityQueue<HeightAxis> heightQueue){
        int [] [] dp=new int [n][m];
        int maxResult=Integer.MIN_VALUE;
        while(heightQueue.size()>0){    
                HeightAxis h=heightQueue.poll();
                int i=h.i;
                int j=h.j;
                int max=1;
                if(i-1>=0 && matrix[i][j]>matrix[i-1][j]){
                    max=(dp[i-1][j]+1)>max?(dp[i-1][j]+1):max;
                }
                 if(i+1<n && matrix[i][j]>matrix[i+1][j]){
                    max=(dp[i+1][j]+1)>max?(dp[i+1][j]+1):max;
                }
                 if(j-1>=0 && matrix[i][j]>matrix[i][j-1]){
                     max=(dp[i][j-1]+1)>max?(dp[i][j-1]+1):max;
                }
                 if(j+1<m && matrix[i][j]>matrix[i][j+1]){
                     max=(dp[i][j+1]+1)>max?(dp[i][j+1]+1):max;
                }
                dp[i][j]=max;  
                maxResult=max>maxResult?max:maxResult;
        }
        System.out.println(maxResult);
//     max
    }
}
```
