# SceneObjectsOnlyAttribute

> *Scene Objects Only Attribute特性：用于对象属性，并将属性限制为场景对象，而不是项目资源。如果要确保对象是场景对象而不是项目资产，请使用此选项。*

![img](../image/SceneObjectsOnlyAttribute/post-668-5fb7dc315411e.gif)

```cs
using Sirenix.OdinInspector;
using System.Collections.Generic;
using UnityEngine;

public class SceneObjectsOnlyAttributeExample : MonoBehaviour
    {
    [Title("Assets only")]
    [AssetsOnly]
    public List<GameObject> OnlyPrefabs;

    [AssetsOnly]
    public GameObject SomePrefab;

    [AssetsOnly]
    public Material MaterialAsset;

    [AssetsOnly]
    public MeshRenderer SomeMeshRendererOnPrefab;

    [Title("Scene Objects only")]
    [SceneObjectsOnly]
    public List<GameObject> OnlySceneObjects;

    [SceneObjectsOnly]
    public GameObject SomeSceneObject;

    [SceneObjectsOnly]
    public MeshRenderer SomeMeshRenderer;
}
```