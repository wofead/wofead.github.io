# EditorOnly

Unity自带了一个EditorOnly的Tag。意思就是标记过这个游戏对象，只在Editor下生效不会被最终打进包里。这个功能其实很必要，但是可能被很多团队都遗忘了。

例如：美术做的场景需要一些摄像机、角色、特效来进行辅助参照。这些东西单纯只是用来参照的，并不希望打进游戏包中。但是开发阶段又不能再场景中删除，因为场景修改后美术还是希望有参照物进行预览。

这一类东西，就非常适合标记成EditorOnly，但是这个Tag有个致命的缺陷-“无法预览”  因为场景的东西非常多，我们必须要很清楚的看到，那些是标记了EditorOnly的游戏对象，总不能手动的一个个找吧。

```c#
#if UNITY_EDITOR
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEditor;
 
public class EditorOnly : MonoBehaviour {
 
	[HideInInspector]
	public string tag = "Untagged";
	void OnDrawGizmos() 
	{
		foreach (GameObject go in GameObject.FindGameObjectsWithTag(tag)) {
			UnityEditor.Handles.Label(go.transform.position, tag);
		}
	}
}
 
[CustomEditor(typeof(EditorOnly))]
public class EditorOnlyEditor:Editor{
	public override void OnInspectorGUI ()
	{
		base.OnInspectorGUI ();
		EditorOnly gizmos = target as EditorOnly;
		EditorGUI.BeginChangeCheck ();
		gizmos.tag = EditorGUILayout.TagField("Tag for Objects:", gizmos.tag);
		if (EditorGUI.EndChangeCheck ()) {
			EditorUtility.SetDirty (gizmos);
		}
	}
}
#endif
```

如下图所示，选择tag以后就可以直接在SceneView视图中查看到那些是当前选中的Tag了。



这是自定义的Gizmos功能。如果想再SceneView中整体隐藏，如下图所示，勾掉它即可。

