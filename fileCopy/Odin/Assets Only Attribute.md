# Asset Only Attribute

> Assets Only Attribute特性有两类
>
> * **Assets Only：**点击需要序列化的资源字段时，在出现的弹窗中只有Project的资源文件，不会出现Hierarchy（场景）的资源。
> * **SceneObjectOnly：**点击需要序列化的资源字段时，在出现弹窗中只有Hierarchy中的资源文件，不会出现Project的资源。
>
> **注意：**预制体等资源在Scene或者Project中都含有，出现的弹窗中也会都含有对应的资源。

```c#
using Sirenix.OdinInspector;
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class AssetsOnlyExample : MonoBehaviour
{
    [AssetsOnly]
    public List<GameObject> OnlyPrefabs;

    [AssetsOnly]
    public GameObject SomePrab;

    [AssetsOnly]
    public Material MaterialAsset;

    [AssetsOnly]
    public MeshRenderer SomeMeshRendererOnPrefab;

    [SceneObjectsOnly]
    public List<GameObject> OnlySceneObjects;

    [SceneObjectsOnly]
    public GameObject SomeSceneObject;

    [SceneObjectsOnly]
    public MeshRenderer SomeMeshRenderer;
}

```

