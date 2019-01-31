# remedialnotes
embarassing notes

transform.forward - to get forward from quaternion, multiply quaternion by whatever vector "forward" is in your coordinate system - most likely (0,0,1) 

var fwd = math.forward(rotations[i].Value);

structs values cant be assigned, whole new struct must be replaced - struct = new struct{ value = 21 };


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



------------------- python --------------------------
https://www.pugetsystems.com/labs/hpc/The-Best-Way-to-Install-TensorFlow-with-GPU-Support-on-Windows-10-Without-Installing-CUDA-1187/#create-a-python-virtual-environment-for-tensorflow-using-conda

python --version
conda install python=3.6
