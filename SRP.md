# stupid scriptable render pipeline/unity shader stuff
embarassing notes


transform.forward - to get forward from quaternion, multiply quaternion by whatever vector "forward" is in your coordinate system - most likely (0,0,1) 

var fwd = math.forward(rotations[i].Value);

structs values cant be assigned, whole new struct must be replaced - struct = new struct{ value = 21 };

# builtin -> urp api           
```cs
//#include "UnityCG.cginc"
#include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Core.hlsl"

//other core/useful 
#include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/SurfaceInput.hlsl"
#include "Packages/com.unity.render-pipelines.core/ShaderLibrary/Color.hlsl"
#include "Packages/com.unity.render-pipelines.core/ShaderLibrary/Common.hlsl"
```
