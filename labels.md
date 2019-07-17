# Labels:
- labels are key/value pairs that can be attached to object
  - labels are like tag in Aws or other cloud providers using tag resources

- you can label your object, for instances your pod , following an organization structure 

  - key: env - value: dev/stating/qa/prod
  - key: department - value: engineeing/finance/marketing

- labels are not unique and multiple labels can be added to one node
- once labels are attached to an object, you can use filters to narrow down results
  - this is called label selectors
- using labels selectors, you can use matcing expression to match labels
  - for instance, a priticulat pod can only run on a node labeled with "env" equals "dev"
  - more complex matching: "env" in "dev" or "qa"

### NOde labels
- you can use labels to tag nodes
- once nodes are tagged, you can use label selectors to let podto run on specific nodes
- there are 2 steps required to run a pod on a specific set of nodes:
   - first  you tag the node
   - then you add a node selector to your pod configuraion
