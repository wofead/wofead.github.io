# Set Raycast Target

> *前言：在开发中有时处于性能原因或者业务需求，会批量处理UI中的Raycast Target选项*
> *【Odin工具箱】这次集成Raycast Target批处理，不论实例物体(Hierarchy中)还是Asset资源（Project中），都可以更改并保存.*

##### 打开Odin工具箱

![img](https://aihailan.com/wp-content/uploads/2020/11/post-912-5fb80d601d634.gif)

##### 将需要处理的物体拖入即可

![img](https://aihailan.com/wp-content/uploads/2020/11/post-912-5fb80d60738a0.gif)
![img](https://aihailan.com/wp-content/uploads/2020/11/post-912-5fb80d6090c89.gif)

##### 示例完整代码

```cs
using Sirenix.OdinInspector;
using System.Collections.Generic;
using UnityEditor;
using UnityEngine;
using UnityEngine.UI;

[TypeInfoBox("批量选中或者取消对应UI上的Raycast")]
public class OneKeyChangeRaycastTarget : SerializedScriptableObject
{

    private bool selectRaycastComplete;
    private bool cancelRaycastComplete;

    [PreviewField(50)]
    [OnValueChanged("SelectRaycastListValueChangeCallBack")]
    [HorizontalGroup("Raycast")]
    [PropertySpace(10, 10)]
    [BoxGroup("Raycast/Left", false)]
    [LabelText("需要选中Raycast拖入进来")]
    public List<GameObject> SelectRaycastList = new List<GameObject>();

    public void SelectRaycastListValueChangeCallBack()
    {
        selectRaycastComplete = false;
    }

    [PreviewField(50)]
    [OnValueChanged("CancelRaycastListValueChangeCallBack")]
    [PropertySpace(10, 10)]
    [BoxGroup("Raycast/Right", false)]
    [LabelText("需要取消Raycast拖入进来")]
    public List<GameObject> CancelRaycastList = new List<GameObject>();

    public void CancelRaycastListValueChangeCallBack()
    {
        cancelRaycastComplete = false;
    }

    [HideIf("@selectRaycastComplete==true||SelectRaycastList.Count==0")]
    [BoxGroup("Raycast/Left")]
    [Button("勾选所有射线检测", ButtonSizes.Large, ButtonStyle.FoldoutButton)]
    public void SelectRaycast()
    {
        for (int i = 0; i < SelectRaycastList.Count; i++)
        {
            if (SelectRaycastList[i] == null)
            {
                continue;
            }
            Graphic[] graphicArray = SelectRaycastList[i].GetComponentsInChildren<Graphic>();
            for (int k = 0; k < graphicArray.Length; k++)
            {
                graphicArray[k].raycastTarget = true;
                EditorUtility.SetDirty(graphicArray[k]);
            }
            GameObject tempObject = SelectRaycastList[i];
            bool isPrefabInstance = PrefabUtility.IsPartOfPrefabInstance(tempObject);
            bool isPrefabAsset = PrefabUtility.IsPartOfPrefabAsset(tempObject);
            if (isPrefabAsset)
            {
                PrefabUtility.SavePrefabAsset(SelectRaycastList[i]);
            }
            if (isPrefabInstance)
            {
                PrefabUtility.ApplyPrefabInstance(SelectRaycastList[i], InteractionMode.UserAction);
            }
        }
        selectRaycastComplete = true;
        AssetDatabase.Refresh();
    }

    [HideIf("@cancelRaycastComplete==true||CancelRaycastList.Count==0")]
    [BoxGroup("Raycast/Right")]
    [Button("取消所有射线检测", ButtonSizes.Large, ButtonStyle.FoldoutButton)]
    public void CancelRaycast()
    {
        for (int i = 0; i < CancelRaycastList.Count; i++)
        {
            if (CancelRaycastList[i] == null)
            {
                continue;
            }
            Graphic[] graphicArray = CancelRaycastList[i].GetComponentsInChildren<Graphic>();
            for (int k = 0; k < graphicArray.Length; k++)
            {
                graphicArray[k].raycastTarget = false;
                EditorUtility.SetDirty(graphicArray[k]);
            }
            GameObject tempObject = CancelRaycastList[i];
            bool isPrefabInstance = PrefabUtility.IsPartOfPrefabInstance(tempObject);
            bool isPrefabAsset = PrefabUtility.IsPartOfPrefabAsset(tempObject);

            if (isPrefabAsset)
            {
                PrefabUtility.SavePrefabAsset(CancelRaycastList[i]);
            }
            if (isPrefabInstance)
            {
                PrefabUtility.ApplyPrefabInstance(CancelRaycastList[i], InteractionMode.UserAction);
            }
        }
        cancelRaycastComplete = true;
        AssetDatabase.Refresh();
    }
}
```