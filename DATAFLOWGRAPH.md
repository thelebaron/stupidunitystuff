
# dataflowgraph + animation
learning notes - might be incorrect

# Hierarchy notes

GameObject with structure of 
```
CharacterRoot -- SkeletonRoot -- HipsJoint -- OtherJoints  
              -- CharacterMesh  
// gets converted into
CharacterRoot -- SkeletonRoot -- HipsJoint -- OtherJoints  
CharacterRoot -- SkeletonRoot -- CharacterMesh  
              -- CharacterMesh  
```
The "CharacterMesh" entity under the hips appears to be the main mesh? Contains
while the "CharacterMesh" entity under the root isnt? 

This entity also has a SkinnedMeshToRigIndexMapping buffer. RigIndex index contains corresponding SkinMeshIndex(why isnt this 1:1 match given they are the same length?)
SkinnedMeshRenderer.bones contains the rig index + name


## change graph threading
```
nodeSet.RendererModel = NodeSet.RenderExecutionModel.MaximallyParallel;
```
options:    
  MaximallyParallel = Every node of execution will be launched in a separate job  
  SingleThreaded - All nodes are executed in a single job  
  Synchronous - All nodes are executed on the calling thread  
  Islands - Connected components in the graph will be executed in one job.  

## ClipConfiguration

ClipConfigurationMask:  
```
  NormalizedTime = 1, // make animation run at sensible rate  
  LoopTime = 2, // loop animation duration  
  LoopValues = 4, // loop animation(ie seamless loop)  
  CycleRootMotion = 8, // ?  
  DeltaRootMotion = 16, // ?  
  RootMotionFromVelocity = 32, // ?    
  BankPivot = 64, // ?    
```
MotionId: ?

## Systems interaction

Must register/deregister any system with ProcessDefaultAnimationGraph
```
// In OnCreate
// Increase the reference count on the graph system so it knows that we want to use it.
m_GraphSystem = World.GetOrCreateSystem<ProcessDefaultAnimationGraph>();
m_GraphSystem.AddRef();
m_GraphSystem.Set.RendererModel = NodeSet.RenderExecutionModel.Islands;

// In OnDestroy
// Clean up all our nodes in the graph
Entities.WithAll<SynchronizeMotionSample>().WithoutBurst().WithStructuralChanges().ForEach((Entity e, ref SynchronizeMotionGraphComponent graph) => {
    DestroyGraph(ref graph);
}).Run();

// Decrease the reference count on the graph system so it knows
// that we are done using it.
m_GraphSystem.RemoveRef();
base.OnDestroy();

```

## Get value from node

Old unused code snippet
```
  [UpdateAfter(typeof(RotateSimpleController))]
  [UpdateInGroup(typeof(DefaultAnimationSystemGroup))]
  public class GraphMainThreadUpdaterSystem : SystemBase
  {
      private ProcessDefaultAnimationGraph m_GraphSystem;

      protected override void OnCreate()
      {
          base.OnCreate();
          m_GraphSystem = World.GetOrCreateSystem<ProcessDefaultAnimationGraph>();
          m_GraphSystem.AddRef();
      }

      protected override void OnDestroy()
      {
          base.OnDestroy();
          m_GraphSystem.RemoveRef();
      }

      protected override void OnUpdate()
      {
          // Update graph if the animation component changed
          Entities
          //.WithChangeFilter<MyFirstClip_PlayClipComponent>()
          .WithName("UpdateVelocityGraph")
          .WithoutBurst()
          .ForEach((Entity e, ref PhysicsVelocity velocity, in SimpleControllerData simpleControllerData, in LocalToWorld localToWorld) =>
          {
              var node = m_GraphSystem.Set.GetDefinition(simpleControllerData.PhysicsNodeHandle);

              GraphValue<float3> nodeGV = m_GraphSystem.Set.CreateGraphValue(simpleControllerData.PhysicsNodeHandle, PhysicsVelocityNode.KernelPorts.OuputDelta);
              //node.SimulationPorts.Output;

              //Debug.Log(m_GraphSystem.Set.GetValueBlocking(nodeGV) * 30);

              var linear = m_GraphSystem.Set.GetValueBlocking(nodeGV) * 30;
              var result = math.mul(new quaternion(localToWorld.Value), linear);

              //node.Hello();
              //Debug.Log(x);
              velocity.Linear = result;


              m_GraphSystem.Set.ReleaseGraphValue(nodeGV);

          }).Run();



      }
  }
```


## Questions
* Forum for dataflowgraph?
* Will ProcessDefaultAnimationGraph be included in the final release? Why is it not a part of the animation package
* How to add/remove rootmotion
* How to transition by time/bool
* Are the nodes in the samples expected to be included in the official packages?
* Is creating DFG nodes considered to be expected/commonplace? Is DFG an internal tool to facillitate or intended for users to base game logic on
* Will DFG have a visual representation?
* Why are animation systems mainthreaded for graph interactions?
* What are the differences between all the rootmotion configuration options? Is configuration all you need to set?
* Animation events?
* How to set a node to automatically overwrite component data? Or is there a receive message function?
