# 组合总和

> 给定一个数组 candidates 和一个目标数 target ，找出 candidates 中所有可以使数字和为 target 的组合。
>
> candidates 中的每个数字在每个组合中只能使用一次。
>
> 1. 所有数字（包括目标数）都是正整数。
> 2. 解集不能包含重复的组合。 
>

## 结题思路

1. 排序
2. 得到每个数字重复的个数
3. 遍历每个个数，index加个数
4. 判断结果

## 解：

```c#
public IList<IList<int>> Solve2(int[] candidates, int target)
        {
            IList<IList<int>> result = new List<IList<int>>();
            Array.Sort(candidates);
            Dictionary<int, int> valueDic = new Dictionary<int, int>();
            for (int i = 0; i < candidates.Length; i++)
            {
                int value = candidates[i];
                if (valueDic.ContainsKey(value))
                {
                    valueDic[value] = valueDic[value] + 1;
                }
                else
                {
                    valueDic[value] = 1;
                }
            }
            List<int> combine = new List<int>();
            IsEqualTarget(candidates, target, result, combine, 0, valueDic);
            return result;
        }

        public void IsEqualTarget(int[] candidates, int remind, IList<IList<int>> result, List<int> combine, int index, Dictionary<int, int> valueDic)
        {
            if (remind == 0)
            {
                result.Add(new List<int>(combine));
                return;
            }
            if (index == candidates.Length || remind < 0)
            {
                return;
            }
            else
            {
                int value = candidates[index];
                int count = remind / value;
                count = Math.Min(count, valueDic[value]);
                if (count == 0)
                {
                    return;
                }
                for (int i = count; i >= 0; i--)
                {
                    for (int j = 0; j < i; j++)
                    {
                        combine.Add(value);
                    }
                    IsEqualTarget(candidates, remind - i * value, result, combine, index + valueDic[value], valueDic);
                    for (int j = 0; j < count; j++)
                    {
                        combine.Remove(value);
                    }
                }
            }
        }
```



