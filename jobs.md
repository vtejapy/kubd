# job 
- each job creates one or more pods
- Ensure they are successfully terminated
- job controller restarts or re-scheduled if a pod node fails during execution
-  can run multiple pods in parallel 
-  can scale up using kubectl scale command

***useCases***:
- one time initilization of resource such as databases
-  multiple worker to process message in queue

