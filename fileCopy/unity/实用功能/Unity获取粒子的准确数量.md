# 获取粒子的准确数量

```c#
using System.Reflection;
using UnityEngine;
 
public class Test : MonoBehaviour 
{
    private ParticleSystem [] m_ParticleSystems;
    private MethodInfo m_CalculateEffectUIDataMethod;
    private int m_ParticleCount = 0;
 
    void Start()
    {
        m_ParticleSystems = GetComponentsInChildren<ParticleSystem>();
        m_CalculateEffectUIDataMethod = typeof(ParticleSystem).GetMethod("CalculateEffectUIData", BindingFlags.Instance | BindingFlags.NonPublic);
    }
 
 
 
    private void Update()
    {
        m_ParticleCount = 0;
        foreach (var ps in m_ParticleSystems)
        {
            int count = 0;
            object[] invokeArgs = new object[] { count, 0.0f, Mathf.Infinity };
            m_CalculateEffectUIDataMethod.Invoke(ps, invokeArgs);
            count  = (int)invokeArgs[0];
            m_ParticleCount += count;
        }
    }
 
    private void OnGUI()
    {
        GUILayout.Label(string.Format("<size=50>m_ParticleCount : {0}</size>", m_ParticleCount));
    }
}
```

