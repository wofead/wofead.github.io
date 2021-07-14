# ValueDropdownAttribute

> Value Dropdown Attribute特性用于任何属性，并使用可配置选项创建下拉列表。使用此选项可为用户提供一组特定的选项供您选择。
> 也就是创建一些特殊的下拉条
>
> 这个里面的属性就有点多了，达到了16个！！！
> 下面笔者逐个讲解

##### MemberName,也是唯一一个有参构造函数需要的属性，有两种形式的Drop下拉条，一种是直接数值的，另一种是Key-Value形式的

![img](https://aihailan.com/wp-content/uploads/2020/11/post-710-5fb7dd6530e58.gif)

```cs
    /*【MemberName】*/
    [PropertySpace(40, 0)]
    [ValueDropdown("TextureSizes")]
    public int SomeSize1;
    private static int[] TextureSizes = new int[] { 32, 64, 128, 256, 512, 1024, 2048, 4096 };

    [ValueDropdown("FriendlyTextureSizes")]
    public int SomeSize2;
    private static IEnumerable FriendlyTextureSizes = new ValueDropdownList<int>()
    {
      { "Small", 256 },
      { "Medium", 512 },
      { "Large", 1024 },
    };
```

