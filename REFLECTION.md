
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


working code dump

```
var entities   = em.GetAllEntities();
var methodInfo = typeof(EntityManager).GetMethod("GetComponentData");
for (var i = 0; i < 15/*entities.Length*/; i++)
{
    var e     = entities[i];
    var types = em.GetComponentTypes(e);
    for (var j = 0; j != types.Length; j++)
    {
        var t = types[j].GetManagedType();
        // Ignore tag component for now
        if (TypeManager.GetTypeInfo(types[j].TypeIndex).IsZeroSized)
            continue;
        // Ignore buffer component for now
        if (typeof(IBufferElementData).IsAssignableFrom(t))
            continue;
        // Ignore shared component for now
        if (typeof(ISharedComponentData).IsAssignableFrom(t))
            continue;

        if (typeof(IComponentData).IsAssignableFrom(t))
        {
            //Debug.Log(t + " is icd");
            var genericMethodInfo = methodInfo?.MakeGenericMethod(t);
            var parameters        = new object[] {e};
            //Debug.Log(t.Name + JsonUtility.ToJson(genericMethodInfo?.Invoke(em, parameters)));

            var reflectedComponentData = genericMethodInfo?.Invoke(em, parameters);
            var cast                  = Cast(t, reflectedComponentData);

            Debug.Log("is same stuff? " + cast.Equals(reflectedComponentData));

            foreach (var field in t.GetFields(BindingFlags.Instance | BindingFlags.NonPublic | BindingFlags.Public))
            {
                Debug.Log(type.Name + field.Name + field.GetValue(reflectedComponentData)); // last part doesnt work
            }
            //Debug.Log(cast);

            var obj = Activator.CreateInstance(t); // what does this actually do?

        }

    }

    types.Dispose();
}

entities.Dispose();
```
