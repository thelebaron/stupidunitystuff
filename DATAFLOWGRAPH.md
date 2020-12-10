
# dataflowgraph + animation
learning notes - might be incorrect

# Hierarchy notes
GameObject with structure of 
CharacterRoot -- SkeletonRoot -- HipsJoint -- OtherJoints  
              -- CharacterMesh  
gets converted into
CharacterRoot -- SkeletonRoot -- HipsJoint -- OtherJoints  
CharacterRoot -- SkeletonRoot -- CharacterMesh  
              -- CharacterMesh  

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
