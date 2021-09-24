# Hierarchy视图监听gameObject点击事件

```c#
[InitializeOnLoadMethod]
	static void Start () 
	{
	
		Selection.selectionChanged = delegate 
		{
			Debug.Log(Selection.activeObject.name);
		};
	
	}
```

下面的这两个代理事件大家都知道吧？其实都可以干这件事，但是不完美。因为每一帧都会调用一下，才能做判断
EditorApplication.hierarchyWindowItemOnGUI
EditorApplication.update

我觉得最好的办法，还是说当我选择某个gameObject的时候，由unity回调给我一个事件。所以我又找到了一个不完美的解决方法。在你需要监听点击的gameObject的脚本上添加如下代码。OnDrawGizmosSelected 就是选择的回调。但是它可能会回调多次，所以要进行一次判断保证它只执行一次。

**OnDrawGizmosSelected 得放在MonoBehaviour里。**

```c#
#if UNITY_EDITOR
    bool selected = false;
    void OnDrawGizmosSelected()
    {
        if (!selected)
        {
            selected = true;
            Debug.Log(gameObject.name);
        }
    }
#endif
```

