[TOC]

## BFS基本思想

虽然题目千变万化，但是考察点永远是那几个。本题给出了一个场景：求 每个 1 到 0 的最短的曼哈顿距离。其中曼哈顿距离就是只能沿着横、竖到达另外一个点走的步数。

在一个图中，能从一个点出发求这种最短距离的方法很容易想到就是 BFS，BFS 的名称是广度优先遍历，即把周围这一圈搜索完成之后，再搜索下一圈，是慢慢扩大搜索范围的。

图左边是BFS，按照层进行搜索；图右边是 DFS，先一路走到底，然后再回头搜索。

##  ![img](https://pic.leetcode-cn.com/75fc42a2cfacf6e41a86b34b1861d2cdcd2965b20d8ebc0a6dcc41bb1fbcea31-BFS-and-DFS-Algorithms.png) BFS模板

BFS 使用队列，把每个还没有搜索到的点依次放入队列，然后再弹出队列的头部元素当做当前遍历点。BFS 总共有两个模板：

1、如果不需要确定当前遍历到了哪一层，BFS 模板如下。

```cpp
while queue 不空：
    cur = queue.front();
    queue.pop();
    for 节点 in cur的所有相邻节点：
        if 该节点有效且未访问过：
            queue.push(该节点)
```

2、如果要确定当前遍历到了哪一层，BFS 模板如下。
这里增加了 level 表示当前遍历到二叉树中的哪一层了，也可以理解为在一个图中，现在已经走了多少步了。size 表示在当前遍历层有多少个元素，也就是队列中的元素数，我们把这些元素一次性遍历完，即把当前层的所有元素都向外走了一步。

```cpp
level = 0
while queue 不空：
    size = queue.size()
    while (size --) {
        cur = queue.front();
        queue.pop();
        for 节点 in cur的所有相邻节点：
            if 该节点有效且未被访问过：
                queue.push(该节点)
    }
    level ++;

```



## 力扣例题详解

#### 1、01矩阵 --中等

给定一个由 0 和 1 组成的矩阵，找出每个元素到最近的 0 的距离。

两个相邻元素间的距离为 1 。

**示例 1:** 

```
输入:

0 0 0
0 1 0
0 0 0
输出:

0 0 0
0 1 0
0 0 0
```

**示例 2:** 

```
输入:

0 0 0
0 1 0
1 1 1
输出:

0 0 0
0 1 0
1 2 1
```

**注意:**

1. 给定矩阵的元素个数不超过 10000。
2. 给定矩阵中至少有一个元素是 0。
3. 矩阵中的元素只在四个方向上相邻: 上、下、左、右。



```cpp
class Solution {
public:
    vector<vector<int>> updateMatrix(vector<vector<int>>& matrix) {
        int numr = matrix.size();
        int numc = matrix[0].size();
        vector<pair<int,int>> around = {{0,1},{0,-1},{-1,0},{1,0}};
        vector<vector<int>> result(numr, vector<int>(numc, INT_MAX));
        queue<pair<int,int>> que;
        for(int i = 0; i < numr; i++){
            for(int j = 0; j < numc; j++){
                if(matrix[i][j] == 0){
                    result[i][j] = 0;
                    que.push({i, j});
                }
            }
        }
        while(!que.empty()){
            auto temp = que.front();
            que.pop();
            for(int i = 0; i < 4; i++){  //上下左右四个方向依次遍历
                int x = temp.first + around[i].first;
                int y = temp.second + around[i].second;
                if(x >= 0 && x < numr && y >= 0 && y < numc){
                    if(result[x][y] > result[temp.first][temp.second] + 1){
                        result[x][y] = result[temp.first][temp.second] + 1;
                        que.push({x, y});
                    }
                }

            }
        }
        return result;
    }
};
```



#### 2、地图分析  --中等

你现在手里有一份大小为 N x N 的「地图」（网格） grid，上面的每个「区域」（单元格）都用 0 和 1 标记好了。其中 0 代表海洋，1 代表陆地，请你找出一个海洋区域，这个海洋区域到离它最近的陆地区域的距离是最大的。

我们这里说的距离是「曼哈顿距离」（ Manhattan Distance）：(x0, y0) 和 (x1, y1) 这两个区域之间的距离是 |x0 - x1| + |y0 - y1| 。

如果我们的地图上只有陆地或者海洋，请返回 -1。

**示例 1：**

 ![img](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2019/08/17/1336_ex1.jpeg) 

```
输入：[[1,0,1],[0,0,0],[1,0,1]]
输出：2
解释： 
海洋区域 (1, 1) 和所有陆地区域之间的距离都达到最大，最大距离为 2。
```

**示例 2：**

 ![img](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2019/08/17/1336_ex2.jpeg) 

```
输入：[[1,0,0],[0,0,0],[0,0,0]]
输出：4
解释： 
海洋区域 (2, 2) 和所有陆地区域之间的距离都达到最大，最大距离为 4。
```

```
提示：

1 <= grid.length == grid[0].length <= 100
grid[i][j] 不是 0 就是 1
```

```cpp
class Solution {
public:
    int maxDistance(vector<vector<int>>& grid) {
        const int M = grid.size();
        const int N = grid[0].size();
        // 使用deque作为队列
        deque<pair<int, int>> deq;
        for (int i = 0; i < M; ++i) {
            for (int j = 0; j < N; ++j) {
                if (grid[i][j] == 1) {
                    // 将所有陆地都放入队列中
                    deq.push_back({i, j});
                }
            }
        }
        // 如果没有陆地或者海洋，返回-1
        if (deq.size() == 0 || deq.size() == M * N) {
            return -1;
        }
        // 由于BFS的第一层遍历是从陆地开始，因此遍历完第一层之后distance应该是0
        int distance = -1;
        // 对队列的元素进行遍历
        while (deq.size() != 0) {
            // 新遍历了一层
            distance ++;
            // 当前层的元素有多少，在该轮中一次性遍历完当前层
            int size = deq.size();
            while (size --) {
                // BFS遍历的当前元素永远是队列的开头元素
                auto cur = deq.front(); deq.pop_front();
                // 对当前元素的各个方向进行搜索
                for (auto& d : directions) {
                    int x = cur.first + d[0];
                    int y = cur.second + d[1];
                    // 如果搜索到的新坐标超出范围/陆地/已经遍历过，则不搜索了
                    if (x < 0 || x >= M || y < 0 || y >= N ||
                        grid[x][y] != 0) {
                        continue;
                    }
                    // 把grid中搜索过的元素设置为2
                    grid[x][y] = 2;
                    // 放入队列中
                    deq.push_back({x, y});
                }
            }
        }
        // 最终走了多少层才把海洋遍历完
        return distance;
    }
private:
    vector<vector<int>> directions = {{-1, 0}, {1, 0}, {0, 1}, {0, -1}};
};

```



#### 3、完全平方数 --中等

给定正整数 n，找到若干个完全平方数（比如 1, 4, 9, 16, ...）使得它们的和等于 n。你需要让组成和的完全平方数的个数最少。

**示例 1:**

```
输入: n = 12
输出: 3 
解释: 12 = 4 + 4 + 4.
```

**示例 2:**

```
输入: n = 13
输出: 2
解释: 13 = 4 + 9.
```

**BFS方法求解**

```cpp
class Solution {
public:
    int numSquares(int n) {
        queue<int> que;
        vector<bool> visit(n+1,true); //记录已经存在的节点，实现hash的功能
        //visit[n] = true;

        que.push(n);
        int minLev = 0;
        while(!que.empty())
        {
            minLev++;
            int size = que.size();
           
           while(size--)
           {
                int cur = que.front();
                que.pop();
                for(int j = 1;j*j <= cur;j++)
                {
                    int temp  =  cur - j*j;
                    if(temp == 0)
                        return minLev;
                    else if(temp < 0)
                        break;
                    
                    if(visit[temp])
                    {
                        visit[temp] = false;
                        que.push(temp);
                    }
                }
            }
        }
        return minLev;
    }
};
```





**动态规划方法求解**

```cpp
class Solution {
public:
    int numSquares(int n) {
        int maxIndex = (int)sqrt(n);

        vector<int> dp(n+1,INT_MAX);
        vector<int> square;

        for(int i = 1;i<=maxIndex;i++)
            square.push_back(i*i);
        
        dp[0] = 0;
        for(int i = 1;i<=n;i++)
        {
            for(int j = 0;j < square.size();j++)
            {
                if(i < square[j])
                    break;
                dp[i] = min(dp[i],dp[i - square[j]] + 1);
            }
        }
        return dp[n];
    }
};
```



#### 4、从上到下打印二叉树   --简单

从上到下按层打印二叉树，同一层的节点按从左到右的顺序打印，每一层打印到一行。

**例如:**
给定二叉树: [3,9,20,null,null,15,7],

```
    3
   / \
  9  20
    /  \
   15   7
```

返回其层次遍历结果：

```
[
  [3],
  [9,20],
  [15,7]
]
```

**提示：**

节点总数 <= 1000

```cpp
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode(int x) : val(x), left(NULL), right(NULL) {}
 * };
 */
class Solution {
public:
    vector<vector<int>> levelOrder(TreeNode* root) {
        if(!root)
            return {};
        vector<vector<int>> result;
        vector<int> temp;
        int level = 0;
        queue<TreeNode*> que;

        que.push(root);

        while(!que.empty())
        {
            int size = que.size();
            while(size--)
            {
                TreeNode* pCur = que.front();
                que.pop();
                temp.push_back(pCur->val);

                if(pCur->left)
                    que.push(pCur->left);
                if(pCur->right)
                    que.push(pCur->right);
            }
            result.push_back(temp);
            temp.clear();
        }
        return result;
    }
};
```



#### 5、跳跃游戏[III] --中等

这里有一个非负整数数组 arr，你最开始位于该数组的起始下标 start 处。当你位于下标 i 处时，你可以跳到 i + arr[i] 或者 i - arr[i]。

请你判断自己是否能够跳到对应元素值为 0 的 任意 下标处。

注意，不管是什么情况下，你都无法跳到数组之外。

**示例 1：**

```
输入：arr = [4,2,3,0,3,1,2], start = 5
输出：true
解释：
到达值为 0 的下标 3 有以下可能方案： 
下标 5 -> 下标 4 -> 下标 1 -> 下标 3 
下标 5 -> 下标 6 -> 下标 4 -> 下标 1 -> 下标 3 
```

**示例 2：**

```
输入：arr = [4,2,3,0,3,1,2], start = 0
输出：true 
解释：
到达值为 0 的下标 3 有以下可能方案： 
下标 0 -> 下标 4 -> 下标 1 -> 下标 3
```

**示例 3：**

```
输入：arr = [3,0,2,1,2], start = 2
输出：false
解释：无法到达值为 0 的下标 1 处。
```

**提示：**

```
1 <= arr.length <= 5 * 10^4
0 <= arr[i] < arr.length
0 <= start < arr.length
```



```cpp
class Solution {
public:
    bool canReach(vector<int>& arr, int start) {
       if(arr.empty())
            return false;
        int len = arr.size();

        queue<int> que;
        vector<bool> visit(len,true); //避免重复访问

        if(start >= len)
            return false;
        que.push(start);
        visit[start] = false;
        while(!que.empty())
        {
            int size = que.size();

            while(size--)
            {
                int cur = que.front();
                que.pop();

                if(arr[cur] == 0)
                    return true;
                if(cur + arr[cur]<len && visit[cur + arr[cur]])
                {
                    que.push(cur + arr[cur]);
                    visit[cur + arr[cur]] = false;
                }
                if(cur - arr[cur] >=0 && visit[cur - arr[cur]])
                {
                    que.push(cur - arr[cur]);
                    visit[cur - arr[cur]] = false;
                }
            }
        }
         return false;
    }
};
```

