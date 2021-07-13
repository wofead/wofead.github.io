# git连接github每次都要输入密码怎么办

原因：在clone 项目的时候，使用了 https方式，而不是ssh方式。

默认clone 方式是：https。

切换到：shh 方式。

## 解决方法

1. 查看clone 地址：**git remote -v**
2. 移除https的方式，换成 ssh方式：**git remote rm origin**
3. 添加新的git方式：ssh方式，ssh方式地址的话，在github上，切换到ssh方式，然后复制地址。**git remote add origin** +git地址
4. 查看push方式是否修改成功：**git remote -v**
5. 重新push（提交一下）：**git push origin master**



