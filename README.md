# remedialnotes
embarassing notes


transform.forward - to get forward from quaternion, multiply quaternion by whatever vector "forward" is in your coordinate system - most likely (0,0,1) 

var fwd = math.forward(rotations[i].Value);

structs values cant be assigned, whole new struct must be replaced - struct = new struct{ value = 21 };

# sin movement             
            var amplitude = 1;
            var frequency = 1;
            var scale = c1.Value;
            scale += amplitude*(math.sin(2*math.PI*frequency*time) - math.sin(2*Mathf.PI*frequency*(time - deltaTime)))*maths.up;
            c1.Value.y = scale.y;
            
# math for TransformPoint
old - transform.TransformPoint(Bezier.GetFirstDerivative(points[0], points[1], points[2], t)) - transform.position;
dots(untested) - float3 offsetPosition = new float3(3, 3, 3);
            quaternion offsetRotation = quaternion.EulerYXZ(50, 25, 100);
            RigidTransform transform = new RigidTransform(offsetRotation, offsetPosition);
            float3 someLocalPosition = new float3(1, 1, 1);
            float3 worldPosition = math.transform(transform, someLocalPosition);

# math for InverseTransformDirection
quaternion rotation = math.inverse(math.quaternion(localToWorld.Value));
float3 localVelocity = math.mul(rotation,velocity.Linear * 0.016f) * timeScale; 
  finally worked for me to be the equiv of 
             
Vector3 localVelocity = transform.InverseTransformDirection(m_FPController.Velocity * 0.016f) * Time.timeScale;

# math for TransformDirection
https://answers.unity.com/questions/356638/maths-behind-transformtransformdirection.html
Take `Vector3 B = transform.TransformDirection(Vector3.right);` as an example.

Says to apply your rotation to real-world right, to get my right. Rotations are quaternions, which apply using `*`, so the statement is really:

     Vector3 B = transform.rotation * Vector3.right;

Using matrixes (quaternions replace them,) the worldToLocal matrix upper-left 3x3 (no translation data) seems to work.
#quaternion Math to math
var rot = quaternion.LookRotationSafe(rayHit.SurfaceNormal, maths.up);
rot = Quaternion.FromToRotation(Vector3.up, rayHit.SurfaceNormal);
# constructing float4x4
        localToParent = new LocalToParent
        {
            Value = float4x4.TRS(new float3(body.m_SmoothPosition/* + cam.PositionOffset*/),
                quaternion.LookRotationSafe(math.forward(rotation.Value), math.up()),
                new float3(1.0f, 1.0f, 1.0f))
        };
# magnitude
You can use math.length\lengthsq, or write your own extension with naming magnitude\sqrmagnitude it's very simple
https://forum.unity.com/threads/unity-mathematics-available-on-github.526100/page-4#post-3996142

# ecs specific
queries
        private ComponentGroup group;
            group = GetComponentGroup(typeof(Position), typeof(Rigidbody), typeof(FindTarget));
            
            group = new EntityArchetypeQuery
            {
                Any = Array.Empty<ComponentType>(),
                None = Array.Empty<ComponentType>(),
                All = new ComponentType[] {typeof(EcsTestData)}
            };
            
            group = GetComponentGroup(new EntityArchetypeQuery()
            {
                All = new ComponentType[] { typeof(Ammo) },
                None = new ComponentType[] { typeof(AttackingTag) }
                
                All = new ComponentType[] { ComponentType.Create<Position>() },
                Any = new ComponentType[] { ComponentType.ReadOnly<ManPowerData>(), ComponentType.ReadOnly<EngineData>() },
                None = System.Array.Empty<ComponentType>()
            });
            
            
            NativeArray<ArchetypeChunk> chunks = group.CreateArchetypeChunkArray(Allocator.TempJob);
            var ecsTestData = GetArchetypeChunkComponentType<FindTarget>(true);

#ijob chunk iteration

        using Game.Components;
        using Unity.Burst;
        using Unity.Collections;
        using Unity.Entities;
        using Unity.Jobs;
        using Unity.Transforms;
        using UnityEngine;

        namespace Game.Systems.Tests
        {
            public class TargetChunkSystem : JobComponentSystem
            {

                private ComponentGroup _g;
                private ArchetypeChunkComponentType<Rotation> rotationType;
                private ArchetypeChunkComponentType<Position> positionType;

                protected override void OnCreateManager()
                {
                    // query
                    _g = GetComponentGroup( new EntityArchetypeQuery
                    {
                        All = new ComponentType[] { ComponentType.Create<TargetBuffer>(), ComponentType.ReadOnly<Search>(), ComponentType.Create<Position>(), ComponentType.ReadOnly<Rotation>(), ComponentType.ReadOnly<Faction>(),  },
                        None =  new ComponentType[] { ComponentType.ReadOnly<Target>(), ComponentType.ReadOnly<Dead>() }, 
                        //Any = Array.Empty<ComponentType>(),
                    } );
                }

                //[BurstCompile]
                private struct Job : IJobChunk
                {
                    public ArchetypeChunkComponentType<Position> PositionType;
                    public ArchetypeChunkComponentType<Rotation> RotationType;
                    public ArchetypeChunkComponentType<Faction> FactionType;
                    public ArchetypeChunkBufferType<TargetBuffer> TargetBufferChunkType;

                    public void Execute(ArchetypeChunk chunk, int chunkIndex, int firstEntityIndex)
                    {
                        //var instanceCount = chunk.Count;
                        var chunkPositions = chunk.GetNativeArray(PositionType);
                        var chunkRotations = chunk.GetNativeArray(RotationType);
                        var chunkFactions = chunk.GetNativeArray(FactionType);
                        var buffers = chunk.GetBufferAccessor(TargetBufferChunkType); //get buffers from chunk

                        for( int i = 0; i < chunk.Count; i++ )
                        {
                            chunkFactions[i] = new Faction
                            {
                                Value = Faction.FactionType.Friendly
                            };
                            chunkPositions[i] = new Position{ Value = Vector3.up};
                            //var buffer = buffers[i];
                            var targetbuf = new TargetBuffer
                            {
                                Value = new Target
                                {
                                    Entity = Entity.Null,
                                    Position = Vector3.up,
                                    Health = 3
                                }
                            };
                            buffers[i].Add(targetbuf);
                        }
                        //chunk.
                    }
                }

                protected override JobHandle OnUpdate(JobHandle deps)
                {
                    //var chunks = _g.CreateArchetypeChunkArray(Allocator.TempJob);

                    var job = new Job
                    {
                        PositionType = GetArchetypeChunkComponentType<Position>(),
                        RotationType = GetArchetypeChunkComponentType<Rotation>(),
                        FactionType = GetArchetypeChunkComponentType<Faction>(),
                        TargetBufferChunkType = GetArchetypeChunkBufferType<TargetBuffer>()
                    };
                    var handle = job.Schedule(_g, deps);


                    return handle;
                }


            }
        }


#chunk buffer access

        using Game.Components;
        using Unity.Burst;
        using Unity.Entities;
        using Unity.Jobs;
        using Unity.Transforms;

        namespace Game.Systems.Tests
        {
            public class TargetChunkSystem : JobComponentSystem
            {

                private ComponentGroup _g;
                protected override void OnCreateManager()
                {
                    // query
                    _g = GetComponentGroup( new EntityArchetypeQuery() 
                    {
                        All = new ComponentType[] { ComponentType.Create<TargetBuffer>(), ComponentType.ReadOnly<Search>(), ComponentType.ReadOnly<Position>(), ComponentType.ReadOnly<Rotation>(), ComponentType.ReadOnly<Faction>(),  },
                        None =  new ComponentType[] { ComponentType.ReadOnly<Target>(), ComponentType.ReadOnly<Dead>() }, 
                        //Any = Array.Empty<ComponentType>(),
                    } );
                }

                // Schedule Job
                protected override JobHandle OnUpdate(JobHandle deps) => new Job()
                {
                    TargetBufferChunkType = GetArchetypeChunkBufferType<TargetBuffer>() // -RW buffer

                }.Schedule(_g, deps);


                [BurstCompile]
                private struct Job : IJobChunk
                {
                    public ArchetypeChunkBufferType<TargetBuffer> TargetBufferChunkType;

                    public void Execute(ArchetypeChunk chunk, int chunkIndex, int firstEntityIndex)
                    {
                        var buffers = chunk.GetBufferAccessor(TargetBufferChunkType); //get buffers from chunk

                        for( int i = 0; i < buffers.Length; i++ )
                        {
                            var buffer = buffers[i];

                            buffer[0] = new TargetBuffer(); //write
                        }
                    }
                }
            }
        }



# jobhandle in an array
      var handleArray = new NativeArray<JobHandle>(100, Allocator.Persistent);
 
      // schedule all jobs
      for (Int32 i = 0; i < _packets.Length; ++i) {
        Job_IJob job;
        job.Packet = _packets[i];
        job.Entities = entitiesArrays[i];
        handleArray[i] = job.Schedule(handle);
      }
 
      // complete
      JobHandle.CompleteAll(handleArray);



# keep scene view active
            if (Application.isEditor)
            {
                m_fpinput.SetCursorLock(false);
                UnityEditor.SceneView.FocusWindowIfItsOpen(typeof(UnityEditor.SceneView));
            }

------------------- python --------------------------
https://www.pugetsystems.com/labs/hpc/The-Best-Way-to-Install-TensorFlow-with-GPU-Support-on-Windows-10-Without-Installing-CUDA-1187/#create-a-python-virtual-environment-for-tensorflow-using-conda

python --version
conda install python=3.6
