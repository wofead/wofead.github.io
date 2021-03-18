#### 移除最多的同行或者同列的石子

> n 块石头放置在二维平面中的一些整数坐标点上。每个坐标点上最多只能有一块石头。
>
> 如果一块石头的 同行或者同列 上有其他石头存在，那么就可以移除这块石头。
>
> 给你一个长度为 n 的数组 stones ，其中 stones[i] = [xi, yi] 表示第 i 块石头的位置，返回 可以移除的石子 的最大数量。
>
> - `1 <= stones.length <= 1000`
> - `0 <= xi, yi <= 104`
> - 不会有两块石头放在同一个坐标点上
>

## 解题思路

这个是一道图论的题，只要有两个点处于同一行或者同一列，那么删除其中一个点。

我们把同行或者同列的点连接起来，这样就构成了一个连通图，那么与点相连接的点必然是同行或者同列。

图建立起来之后，使用广度优先搜索或者深度优先搜索都可以删的只剩下一个。

还有一种方式就是如果图是环的环，从哪里看是都可以，如果不是环需要从一端到另一端。

1. 首先，我们构造图：只要两个点同行或同列，那么将两个点相连接
2. 这样，最后的结果图应该是很多个连通图组成的非连通图
3. 而对于任何连通图，我们都可以从一端开始移除直至只剩下一个点
4. 所以，我们只需要判断有多少个连通图，最后便至少剩余多少个点
5. 最后，用节点的数量 - 连通图的数列即为结果

## 解：

```c#
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace CodeExercise
{
    public class RemoveStones
    {
        public static int ErrorSolve(int[][] stones)
        {
            int len = stones.Length;
            int result = 0;
            Dictionary<int, int> surveColList = new Dictionary<int, int>();
            Dictionary<int, int> surveRowList = new Dictionary<int, int>();
            surveColList.Add(stones[0][0], stones[0][1]);
            surveRowList.Add(stones[0][1], stones[0][0]);
            HashSet<int> colSet = new HashSet<int>();
            HashSet<int> rowSet = new HashSet<int>();
            colSet.Add(stones[0][0]);
            rowSet.Add(stones[0][1]);
            for (int i = 1; i < len; i++)
            {
                if (surveColList.ContainsKey(stones[i][0]) && surveRowList.ContainsKey(stones[i][1]))
                {
                    result += 2;
                    surveRowList.Remove(surveColList[stones[i][0]]);
                    surveColList.Remove(surveRowList[stones[i][1]]);
                    surveColList.Remove(stones[i][0]);
                    surveRowList.Remove(stones[i][1]);
                    surveColList.Add(stones[i][0], stones[i][1]);
                    surveRowList.Add(stones[i][1], stones[i][0]);
                }
                else if (colSet.Contains(stones[i][0]) || rowSet.Contains(stones[i][1]))
                {
                    result += 1;
                }
                else
                {
                    surveColList.Add(stones[i][0], stones[i][1]);
                    surveRowList.Add(stones[i][1], stones[i][0]);
                }
                colSet.Add(stones[i][0]);
                rowSet.Add(stones[i][1]);
            }
            return result;
        }


        public static int Solve(int[][] stones)
        {
            UnionFind unionFind = new UnionFind();
            foreach (int[] stone in stones)
            {
                unionFind.Union(stone[0] + 10001, stone[1]);
            }
            return stones.Length - unionFind.GetCount();
        }
    }

    //并查集
    class UnionFind
    {
        private Dictionary<int, int> parent;
        private int count;
        public UnionFind()
        {
            parent = new Dictionary<int, int>();
            count = 0;
        }

        public int GetCount()
        {
            return count;
        }

        public int Find(int x)
        {
            if (!parent.ContainsKey(x))
            {
                parent.Add(x, x);
                count++;
            }
            if (x != parent[x])
            {
                parent[x] = Find(parent[x]);
            }
            return parent[x];
        }
		//stone[0] + 10001, stone[1]
        public void Union(int x, int y)
        {
            int rootX = Find(x);
            int rootY = Find(y);
            if (rootX == rootY)
            {
                return;
            }
            parent[rootX] = rootY;
            count--;
        }
    }

}



```



