# PingObject

在窗口Ping一个物体，就像在检视面板点击它。

```c#
// Pings the currently selected Object
//Ping当前选择的物体
@MenuItem("Examples/Ping Selected")
static function Ping() {
	if(!Selection.activeObject) {
		Debug.LogError("Select an object to ping");
		return;
	}
	EditorGUIUtility.PingObject(Selection.activeObject);
}
```

