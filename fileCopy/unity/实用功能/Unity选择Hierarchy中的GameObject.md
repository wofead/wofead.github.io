# Unity中选中Hierarchy中的Gameobject

Unity是当鼠标在Hierarchy或者Project视图中选择一个或者多个Object，然后在右侧Inspector面板上就会显示所有属性。那么其实解决这个问题的方法就是使用脚本去选择一个Object就行。

```c#
[MenuItem("GameObject/AutoSelect",false,11)]	
	static void Start () 
	{
 
		GameObject go = GameObject.Find("Directional Light");
 
		EditorGUIUtility.PingObject(go);
		Selection.activeGameObject =  go;
 
	
		//也可以选择Project下的Object
		//Selection.activeObject  = AssetDatabase.LoadAssetAtPath<GameObject>("Assets/Cube.prefab");
 
	}
```

