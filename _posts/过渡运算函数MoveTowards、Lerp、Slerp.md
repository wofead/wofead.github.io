# 过渡运算函数MoveTowards、Lerp、Slerp、Clamp、Smooth

MoveTowards、Lerp、Slerp这个三个函数一般在做数学过渡运算中经常遇到的，今天来具体的讲讲他们的用途和含义。

[toc]

## MoveTowards

Mathf、Vector2、Vector3等许多类都有这个方法，意思都差不多。以Vector3为例：

函数原型：

```c#
public static Vector3 MoveTowards(Vector3 current, Vector3 target, float maxDistanceDelta);
```

作用是将当前值current移向目标target。(对Vector3是沿两点间直线)

maxDistanceDelta就是每次移动的最大程度。

返回值是当current值加上maxDistanceDelta的值，如果这个值超过了target，返回的就是target的值。

例子：

```c#
//表示以每秒moveMax的速度从Current移动到Target。
//因为Current和Target距离是4，所以当moveMax 等于0.5f，用时8秒，moveMax等于2时，用时2秒。
float moveMax = 0.5f;
Vector3 current = new Vector3(0,0,0);
Vector3 target = new Vector3(0,0,4);
current = Vector3.MoveTowards(current, target, moveMax * Time.deltaTime);
```

## Lerp

Lerp表示线性插值，很多地方都用到了它。接下来我们以Vector3为例：

函数原型是：

```c#
public static Vector3 Lerp(Vector3 a, Vector3 b, float t);
```

它的作用就是按照计算a和b之间的插值。t的取值范围是[0,1]。

简单理解就是当t=0，返回值是a，当t=1，返回值是b。同理，t=0.1时是a到b的10%。t就代表了a到b的百分比。

线性插值图示：

例1：

```c#
//在time时间内移动物体
private IEnumerator MoveObject(Vector3 startPos, Vector3 endPos, float time)
{
	float dur = 0.0f;
	while(dur <= time)
	{
		dur += Time.deltaTime;
		transform.position = Vector3.Lerp(startPos, endPos, dur/time);
		yield return null;
	}
}
```

例2：

```c#
private IEnumerator MoveObjectSpeed(Vector3 startPos, Vector3 endPos, float speed)
{
	float startTime = Time.time;
    float length = Vector3.Distance(startPos, endPos);
    float frac = 0f;
    while(frac < 1.0f)
    {
        float dist = (Time.time - startTime) * speed;
        frac = dist / length;
        transform.position = Vector3.Lerp(startPos, endPos, frac);
        yield return null;
    }
}
```

## Slerp

球形插值在Vector3、Quaternion等类都有使用，一般多在Quaternion的旋转操作时使用。

* 对Vector3：

```c#
public static Vector3 Slerp(Vector3 a, Vector3 b, float t);
```

这里球形插值与线性插值不同的地方在于，它将Vectors视为方向而不再是点。返回的向量方向，它的角度是根据a和b的角度插值，而它的长度是根据a和b的长度插值。可以用其模拟太阳的升降变化等操作。

* 对于Quaternion：

```c#
public static Quaternion Slerp(Quaternion a, Quaternion b, float t);
```

计算a与b的球形插值。

Quaternion的Lerp与Slerp结果都是一样的，Lerp的效率会比Slerp高些，但是当旋转值a和b离得比较远时，Lerp的效果会非常差。

对于插值（Slerp和Lerp）运算中的t，要明白它代表的是百分比，直接对齐赋值时间如Time.deltaTime是没意义的，因为如果t为1的话，永远都不会到达目标值。

## Clamp

函数原型：

```c#
Mathf.Clamp(float value, float min, float max);	
```

在Mathf.Clamp中传入三个参数：value,min,max

限制value的值在min与max之间，小于min返回min，大于max返回max。

## Clamp01

函数原型：

```c#
Mathf.Clamp(float value);
```

限制value值在0到1之间。

## SmoothDamp

函数原型：

```c#
static function SmoothDamp(current:Vector3, target:Vector3, ref currentVelocity:Vector3, smoothTime:float=Mathf.Infinity, deltaTime:float = Time.deltaTime):Vector3
```

参数含义：

1. **current** 当前物体位置
2. target 目标物体位置
3. **ref currentVelocity** 当前速度，这个值由你每次调用这个函数时被修改
4. **smoothTime** 到达目标的时间，较小的值将快速到达目标
5. **maxSpeed** 所允许的最大速度，默认无穷大
6. **deltaTime** 自上次调用这个函数的时间，默认为Time.deltaTime

例子，相机跟随角色移动：

```c#
public class FollowTarget:MonoBehavior
{
	public Transform charctor;
	private float smothTime = 0.01f;
	private Vector3 currentVelocity = Vector3.zero;
	private Camera mainCamera;
	
	void Awake()
	{
		mainCamera = Camera.main;
	}
	void Start()
	{
		charactor = GameObject.Fins("Player").GetComponent<Transform>();
	}
	void Update()
	{
		tranform.position = Vector3.SmoothDamp(transform.position, charactor.transform + new Vector3(0,1,-10),ref currentVelocity, smoothTime);
	}
}
```

