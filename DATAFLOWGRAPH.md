
# dfg

change graph threading
```
nodeSet.RendererModel = NodeSet.RenderExecutionModel.MaximallyParallel;
```
options:    
  MaximallyParallel = Every node of execution will be launched in a separate job  
  SingleThreaded - All nodes are executed in a single job  
  Synchronous - All nodes are executed on the calling thread  
  Islands - Connected components in the graph will be executed in one job.  


