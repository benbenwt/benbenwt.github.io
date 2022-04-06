# 数据结构

## 队列

### 双指针题型

## 栈

# 算法

## 深度优先

>模板中的几个关键参数：
>
>path：表示从根节点到当前遍历节点的路径记录
>
>mark：用于处理重复情况，剪枝掉无用的分支。
>
>resultt：每当找到符合条件的节点路径，讲路径存入ret，ret就是最终的结果。

```
dfs{
for循环遍历该层可用的节点{
//剪枝条件  path,mark
//判断是否是结果，处理result  result
//判断是否进入下一层遍历  dfs

}

}
```

```
   public void dfs(List<Integer> path,int first,List<List<Integer>> result,int [] candidates,int target,int len)
    {
        System.out.print(':');
        System.out.println(first);
        for(int i=first;i<len;i++)
        {
            if(i>first&&candidates[i]==candidates[i-1])
            {
                continue;
            }
            
            System.out.print(i);
            System.out.print(':');
            System.out.println(candidates[i]);
            path.add(candidates[i]);

            int sum=0;
            for(int value:path){
                System.out.print(value);
                System.out.print(',');
                sum=sum+value;
            }
            System.out.println();
            if(sum==target)
            {  
                System.out.print("sum:");
                System.out.println(path.size());
                List<Integer> ls=new ArrayList<Integer>(path);
                result.add(ls);
            }
            
            if(sum<target)
            {
                dfs(path,i+1,result,candidates,target,len);
            }
            path.remove(path.size()-1);

            if(sum>target){
                break;
            }//剪枝不要放在两个path操作之间，否则可能没还原path.基于sum的优化放在最后，基于已知数据的放在前边。
        }

    }
```



```
void dfs(int depth,int *nums,int numsSize,int **ret,int *retSize,int *mark,int *path)
{
    for(int i=0;i<numsSize;i++)
    {
        //path,mark;
        if(mark[i]==0) continue;
        if(i>0&&nums[i]==nums[i-1]&&mark[i-1]==1) continue;
        
        mark[i]=0;
        path[depth]=nums[i];
        depth++;
        
        //result
        if(depth==numsSize)
        {
            ret[*retSize]=(int *)malloc(sizeof(int)*numsSize);
            for(int i=0;i<numsSize;i++)
            {
                ret[*retSize][i]=path[i];
            }
            *retSize=*retSize+1;
        }
        
        //dfs
        if(depth<numsSize) dfs(depth,nums,numsSize,ret,retSize,mark,path);
        depth--;
        mark[i]=1;
        
    }
}
```

## 动态规划

>确认计算的方向，以哪为起始点，初始值多少。

>改题中，从左向右计算，一行计算完下一行。第一行为初始值，就是第一行每一列的值。

```
for(int i=0;i<*AColSize;i++)
    {
        dp[0][i]=A[0][i];
         if((0==ASize-1)&&(result>dp[0][i]))
            {
                result=dp[0][i];
            }
    }
    for(int i=1;i<ASize;i++)
    {
        for(int j=0;j<*AColSize;j++)
        {
            if(j==0)
            {
                dp[i][j]=dp[i-1][j]>dp[i-1][j+1]?dp[i-1][j+1]+A[i][j]:dp[i-1][j]+A[i][j];
            }
            else if(j==*AColSize-1)
            {
                dp[i][j]=dp[i-1][j-1]>dp[i-1][j]?dp[i-1][j]+A[i][j]:dp[i-1][j-1]+A[i][j];
            }
            else{
                int min=dp[i-1][j-1]>dp[i-1][j]?dp[i-1][j]:dp[i-1][j-1];
                min=min>dp[i-1][j+1]?dp[i-1][j+1]:min;
                dp[i][j]=min+A[i][j];
            }
            if((i==ASize-1)&&(result>dp[i][j]))
            {
                result=dp[i][j];
            }
        }
    }
```

