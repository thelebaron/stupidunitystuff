
# REFLECTION 

get type
```
Type.GetType("string name")
```

iterate via reflection
```
var componentTypes = em.GetComponentTypes(entity);
foreach (var componentType in componentTypes)
{
    var type        = componentType.GetManagedType();
    //var generic = type.GetGenericTypeDefinition();
    // attempt to get all fields inside struct and name them - not working
    foreach (var fieldInfo in type.GetFields(BindingFlags.Instance | BindingFlags.NonPublic | BindingFlags.Public))
    {
        Debug.Log(type.Name + fieldInfo.Name/* + fieldInfo.GetValue(componentType)*/); // last part doesnt work
    }
}

```


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
