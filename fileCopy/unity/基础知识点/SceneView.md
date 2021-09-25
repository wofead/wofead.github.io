# SceneView

[toc]

## 创建出来的物体放到scene场景的中心

```c#
Ray worldRay = SceneView.lastActiveSceneView.camera.ViewportPointToRay(new Vector3(0.5f, 0.5f, 1.0f));
RaycastHit hit;
if(Physics.Raycast(worldRay, out hit)){
    go.transform.position = hit.point;
}
else{
   	go.transform.position = Sceneview.lastActiveSceneView.camera.ViewPortToWorldPoint(new Vector3(0.5f, 0.5f, 1.0f));
}
```

Ray从屏幕中发射一个点去和其他scene的物体进行碰撞。

## 鼠标所在网格位置

 获取射线和网格之间的交点

```c#
[InitializeOnLoad]
public class RayHitRefrelection
{
    public static Type type_HandleUtility;
    protected static MethodInfo meth_IntersectRayMesh;

    static RayHitRefrelection()
    {
        var editorTypes = typeof(Editor).Assembly.GetTypes();

        type_HandleUtility = editorTypes.FirstOrDefault(t => t.Name == "HandleUtility");
        meth_IntersectRayMesh = type_HandleUtility.GetMethod("IntersectRayMesh", (BindingFlags.Static | BindingFlags.NonPublic));
    }

    public static bool IntersectRayMesh(Ray ray, MeshFilter meshFilter, out RaycastHit hit)
    {
        return IntersectRayMesh(ray, meshFilter.mesh, meshFilter.transform.localToWorldMatrix, out hit);
    }

    static object[] parameters = new object[4];
    public static bool IntersectRayMesh(Ray ray, Mesh mesh, Matrix4x4 matrix, out RaycastHit hit)
    {
        parameters[0] = ray;
        parameters[1] = mesh;
        parameters[2] = matrix;
        parameters[3] = null;
        bool result = (bool)meth_IntersectRayMesh.Invoke(null, parameters);
        hit = (RaycastHit)parameters[3];
        return result;
    }
}
```

获取鼠标的位置，并且将球的位置设置为交点：

```c#
private void OnSceneGUI(SceneView sceneView)
    {
        // 当前屏幕坐标，左上角是（0，0）右下角（camera.pixelWidth，camera.pixelHeight）
        Vector2 mousePosition = Event.current.mousePosition;

        // Retina 屏幕需要拉伸值
        float mult = EditorGUIUtility.pixelsPerPoint;
        // 转换成摄像机可接受的屏幕坐标，左下角是（0，0，0）右上角是（camera.pixelWidth，camera.pixelHeight，0）
        mousePosition.y = sceneView.camera.pixelHeight - mousePosition.y * mult;
        mousePosition.x *= mult;

        // 近平面往里一些，才能看得到摄像机里的位置
        Vector3 fakePoint = mousePosition;
        fakePoint.z = 20;
        Vector3 point = sceneView.camera.ScreenToWorldPoint(fakePoint);

        //Handles.SphereHandleCap(0, point, Quaternion.identity, 2, EventType.Repaint);

        // 刷新界面，才能让球一直跟随
        sceneView.Repaint();
        HandleUtility.Repaint();

        Ray ray = sceneView.camera.ScreenPointToRay(mousePosition);
        MeshFilter[] componentsInChildren = GameObject.FindObjectsOfType<MeshFilter>();
        float num = float.PositiveInfinity;
        foreach (MeshFilter filter in componentsInChildren)
        {
            Mesh sharedMesh = filter.sharedMesh;
            RaycastHit hit;
            if (sharedMesh && RayHitRefrelection.IntersectRayMesh(ray, sharedMesh, filter.transform.localToWorldMatrix, out hit) && hit.distance < num)
            {
                point = hit.point;
                num = hit.distance;
            }
        }
        ShereCapPos(point);
    }
private static Transform capSphere;

    private void ShereCapPos(Vector3 point)
    {
        if (capSphere == null)
        {
            GameObject go = GameObject.Find("[SphereCapPos]");
            if (go == null)
            {
                go = GameObject.CreatePrimitive(PrimitiveType.Sphere);
                go.name = "[SphereCapPos]";

                Collider collider = go.GetComponent<Collider>();
                DestroyImmediate(collider);

                Material mat = AssetDatabase.LoadMainAssetAtPath("Assets/Material/Green.mat") as Material;
                //mat.SetColor("_Color", Color.cyan);
                mat.hideFlags = HideFlags.HideAndDontSave;

                Renderer renderer = go.GetComponent<Renderer>();
                renderer.sharedMaterial = mat;
            }

            go.hideFlags = HideFlags.HideAndDontSave;
            capSphere = go.transform;
            capSphere.rotation = Quaternion.identity;
            capSphere.localScale = Vector3.one * 0.5f;
        }
        capSphere.position = point;
    }
```



## 同步相机位置为Scene场景视口位置或反之

将对象设置为sceneview相机的位置，或者将sceneview相机的位置和对象同步。

```c#
[MenuItem("Tools/CameraUtil/SetPositionRotationToSceneView")]
    public static void ToSceneView()
    {
        if (Selection.gameObjects.Length > 0)
        {
            var transform = SceneView.lastActiveSceneView.camera.transform;
            foreach (var go in Selection.gameObjects)
            {
                go.transform.SetPositionAndRotation(transform.position, transform.rotation);
            }
        }
    }

    [MenuItem("Tools/CameraUtil/SetSceneViewToSelectObject")]
    public static void ToSelectObjectView()
    {
        try
        {
            var cameraObj = Selection.activeObject;
            SceneView.lastActiveSceneView.AlignViewToObject((cameraObj as GameObject).transform);
        }
        catch { }
    }
```

