# remedialnotes
embarassing notes

transform.forward - to get forward from quaternion, multiply quaternion by whatever vector "forward" is in your coordinate system - most likely (0,0,1)

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
