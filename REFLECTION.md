
# REFLECTION 
test
```
// do stuff, make entity, give it data
var em     = World.DefaultGameObjectInjectionWorld.EntityManager;
var entity = em.CreateEntity();
em.AddComponentData(entity, new Translation());
em.AddComponentData(entity, new Rotation());
var componentTypes = em.GetComponentTypes(entity);
// loop through all componenttypes
for (int i = 0; i < componentTypes.Length; i++)
{
    ComponentType componentType = componentTypes[i];
    Type          type    = TypeManager.GetType(componentType.TypeIndex);
}
```

# reflection bit inside loop
```
// get type
var t = typeof(EntityManager);
// get method
var method = t.GetMethod("GetComponentBoxed",  System.Reflection.BindingFlags.NonPublic);
if (method == null)
{
    continue;
}
// invoke(and in this case return data)
var reflectionData = method.Invoke(em, new object[]{componentType});
Debug.Log(reflectionData);
// will log Translation, Rotation if iterating through all types

```
