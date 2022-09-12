[TOC]
## 算法题0912
### DP41 【模板】01背包
>这是max最优值的动态规划问题，状态就是最优值。前后依赖，i依赖于i+1，共三种情况，1装不下2装得下但不装3装得下并装。
>背包：可用容量 累积价值  物品：容量 价值
>起点是，当可选物品只有一个时，且容量已知，那么该物品要么装不下，要么装得下直接装。

```
import java.util.*;
public class Main{
    public static void main(String [] args){
        Scanner scanner=new Scanner(System.in);
        int n,V;
        n=scanner.nextInt();
        V=scanner.nextInt();
        
        int [] volume=new int[n+1];
        int [] weight=new int[n+1];
        for(int i=1;i<=n;i++)
        {
            volume[i]=scanner.nextInt();
            weight[i]=scanner.nextInt();
        }
        
        int [] dp=new int[V+1];
//         将第一维省略了，因为此题只用求出max值，不用记录路径，且dp[i]只依赖于dp[i-1]所以省略了一维。
        for(int i=1;i<=n;i++){
            for(int j=V;j>=volume[i];j--)
            {
                dp[j]=dp[j]>dp[j-volume[i]]+weight[i]?dp[j]:dp[j-volume[i]]+weight[i];
            }
        }
        System.out.println(dp[V]);
        
//         正好装满，就是可用空间刚好等于vomume[i],或可用空间为0.其他情况都为不可达，表示无法装满。
        int [] dp1=new int[V+1];
        Arrays.fill(dp1,Integer.MIN_VALUE);
        dp1[0]=0;
        for(int i=1;i<=n;i++)
        {
            for(int j=V;j>=volume[i];j--)
            {
                dp1[j]=dp1[j]>dp1[j-volume[i]]+weight[i]?dp1[j]:dp1[j-volume[i]]+weight[i];
            }
        }
        if(dp1[V]<0)
        {
            dp1[V]=0;
        }
        System.out.println(dp1[V]);
    }
}
```

### DP5 多少个不同的二叉搜索树
>计数类型的动态规划，状态为可构成的二叉搜索树数量，前后依赖此状态。起点为1个节点，两个节点，三个节点。
>已知i，i+1与i的关系为：分解为左右子树的搜索树数量
```
import java.util.*;
public class Main{
    public static void main(String[] args){
        Scanner scanner=new Scanner(System.in);
        int n=scanner.nextInt();
//         
        int dp[]=new int[n+1];
        dp[0]=1;
        dp[1]=1;
        for(int i=2;i<=n;i++)
        {
            for(int root=1;root<=i;root++)
            {
                dp[i]+=dp[root-1]*dp[i-root];
            }
        }
        System.out.println(dp[n]);
    }
}
```
### DP3 跳台阶扩展问题
>可以跳任意阶数，所以就等于前边所有介的累加和，增加的1就是从0号台阶直接跳n级到达n号台阶的跳法。
```
import java.util.*;
public class Main{
    public static void main(String[] args)
    {
        Scanner scanner=new Scanner(System.in);
        int target=scanner.nextInt();
        
        if(target==1)
        {
            System.out.println(1);
            return;
        }
        if(target==2)
        {
            System.out.println(2);
              return;
        }
        int past=3;
        int ans=0;
        for(int i=3;i<=target;i++)
        {
            ans=past+1;
            past=past+ans;
        }
        System.out.println(ans);
    }
}
```

### DP4 最下花费爬楼梯 自输入输出
```
import java.util.*;

public class Main{
    public static void main(String[] args)
    {
        Scanner scanner=new Scanner(System.in);
        int n=scanner.nextInt();
        int [] cost=new int[n];
        for(int i=0;i<n;i++)
        {
            cost[i]=scanner.nextInt();
        }
        
        System.out.println(Main.minCostClimbingStairs(cost));
    }
    public static int minCostClimbingStairs (int[] cost) {
        // write code here
        if(cost.length==0 || cost.length==1)
        {
            return 0;
        }
        int a=0;
        int b=0;
        int ans=0;
        for(int i=2;i<=cost.length;i++)
        {
            ans=(b+cost[i-1])<(a+cost[i-2])?(b+cost[i-1]):(a+cost[i-2]);
            a=b;
            b=ans;
        }
        return ans;
    }
}
```
### DP1 斐波那契数列 自处理输入输出
```
import java.util.*;

public class Main {
    public static void main(String [] args){
        int n;
        Scanner scanner=new Scanner(System.in);
        n=scanner.nextInt();
        System.out.println(Main.Fibonacci(n));
    }
    public static int Fibonacci(int n) {
        //从0开始，第0项是0，第一项是1
        if(n <= 2)    
             return 1;
         int res = 0;
         int a = 1;
         int b = 1;
         //因n=2时也为1，初始化的时候把a=0，b=1
         for (int i = 3; i <= n; i++){
         //第三项开始是前两项的和,然后保留最新的两项，更新数据相加
             res = (a + b);
             a = b;
             b = res;
         }
        return res;
    }
}
```

### DP2爬台阶 自处理输入输出
```
import java.util.*;

public class Main{
    public static void main(String[] args){
        Scanner scanner=new Scanner(System.in);
        int n=scanner.nextInt();
        System.out.println(Main.jumpFloor(n));
    }
    public static int jumpFloor(int target) {
        if(target==0)
        {
            return 0;
        }
        if(target==1)
        {
            return 1;
        }
        if(target==2)
        {
            return 2;
        }
        int a=1;
        int b=2;
        int ans=0;
        for(int i=3;i<=target;i++)
        {
            ans=a+b;
            a=b;
            b=ans;
        }
        return ans;
    }
}
```
