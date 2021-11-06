# stupid scriptable render pipeline/unity shader stuff
dumb notes
UnityObjectToClipPos > TransformWorldToHClip(v.vertex)

notes

# builtin -> urp api           
```cs
//#include "UnityCG.cginc"
#include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Core.hlsl"

//other core/useful 
#include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/SurfaceInput.hlsl"
#include "Packages/com.unity.render-pipelines.core/ShaderLibrary/Color.hlsl"
#include "Packages/com.unity.render-pipelines.core/ShaderLibrary/Common.hlsl"
```

unknown replacements
//#include "UnityLightingCommon.cginc"
//#include "AutoLight.cginc"
