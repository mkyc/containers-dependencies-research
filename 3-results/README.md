# Results

During implementation of mocks I found that: 
 * `influences` section would be required
 * name of method `validate-config` (or later just `validate`) should in fact be `plan`
 * there is no need to implement method `get-state` in module container provider as state will be local and shared between modules. In fact some state related operations would be probably implemented on cli wrapper level.  
 * instead, there is need of `audit` method which would be extremely important to check if no manual changes were applied to remote infrastructure
 
# Required methods 

As already described there would be 5 main methods required to be implemented by module provider. Those are described in next sections. 

## Metadata

That is simple method to display static YAML/JSON  (or any kind of structured data) information about the module. In fact information from this method should be exactly the same to what is in repo file section about this module. Example output of `metadata` method might be: 

```
labels:
  version: 0.0.1
  name: Bare Metal Kafka
  short: BMK
  kind: stream-processor
  core-technology: apache-kafka
  provides-kafka: 2.5.1
  provides-zookeeper: 3.5.8
requires:
  strong:
    - - key: kind
        operator: eq
        values: [infrastructure]
      - key: provider,
        operator: in,
        values:
          - azure
          - aws
  weak:
    - - key: kind
        operator: eq
        values:
          - logs-storage
    - - key: kind
        operator: eq
        values:
          - monitoring
      - key: core-technology
        operator: eq
        values:
          - prometheus
influences:
  - - key: kind
      operator: eq
      values:
        - monitoring
```

## Init

`init` method main purpose is to jump start usage of module by generating (in smart way) configuration file using information in state. In example Makefile which is stored [here](../2-tests/1-runs/1-infrastructure-and-kafka/Makefile) you can test following scenario: 
 * `make clean`
 * `make init-and-apply-azure-infrastructure`
 * observe what is in `./shared/state.yml` file: 
   ```
   azi:
     status: applied
     size: 5
     provide-pubips: true
     nodes:
       - privateIP: 10.0.0.0
         publicIP: 213.1.1.0
         usedBy: unused
       - privateIP: 10.0.0.1
         publicIP: 213.1.1.1
         usedBy: unused
       - privateIP: 10.0.0.2
         publicIP: 213.1.1.2
         usedBy: unused
       - privateIP: 10.0.0.3
         publicIP: 213.1.1.3
         usedBy: unused
       - privateIP: 10.0.0.4
         publicIP: 213.1.1.4
         usedBy: unused
   ```
   it mocked that it created some infrastructure with VMs having some fake IPs.
 * change IP manually a bit to observe what I mean by "smart way"
   ```
   azi:
     status: applied
     size: 5
     provide-pubips: true
     nodes:
       - privateIP: 10.0.0.0
         publicIP: 213.1.1.0
         usedBy: unused
       - privateIP: 10.0.0.100 <---- here
         publicIP: 213.1.1.100 <---- and here
         usedBy: unused
       - privateIP: 10.0.0.2
         publicIP: 213.1.1.2
         usedBy: unused
       - privateIP: 10.0.0.3
         publicIP: 213.1.1.3
         usedBy: unused
       - privateIP: 10.0.0.4
         publicIP: 213.1.1.4
         usedBy: unused
   ```  
 * `make just-init-kafka`
 * observe what was generated in `./shared/bmk-config.yml`
   ```
   bmk:
     size: 3
     clusterNodes:
       - privateIP: 10.0.0.0
         publicIP: 213.1.1.0
       - privateIP: 10.0.0.100
         publicIP: 213.1.1.100
       - privateIP: 10.0.0.2
         publicIP: 213.1.1.2
   ```
   it used what it found in state file and generated config to actually work with given state. 
 * `make and-then-apply-kafka`
 * check it got applied to state file:
   ```
   azi:
     status: applied
     size: 5
     provide-pubips: true
     nodes:
       - privateIP: 10.0.0.0
         publicIP: 213.1.1.0
         usedBy: bmk
       - privateIP: 10.0.0.100
         publicIP: 213.1.1.100
         usedBy: bmk
       - privateIP: 10.0.0.2
         publicIP: 213.1.1.2
         usedBy: bmk
       - privateIP: 10.0.0.3
         publicIP: 213.1.1.3
         usedBy: unused
       - privateIP: 10.0.0.4
         publicIP: 213.1.1.4
         usedBy: unused
   bmk:
     status: applied
     size: 3
     clusterNodes:
       - privateIP: 10.0.0.0
         publicIP: 213.1.1.0
         state: created
       - privateIP: 10.0.0.100
         publicIP: 213.1.1.100
         state: created
       - privateIP: 10.0.0.2
         publicIP: 213.1.1.2
         state: created
   ```

So `init` method is not just about providing "default" config file, but to actually provide "meaningful" configuration file. What is significant here, is that it's very easily testable if that method generates desired state when given different example state files.  

## Plan

`plan` method is a method to: 
 * validate that config file has correct structure, 
 * get state file, extract module related piece and compare it to config to "calculate" if there are any changes required and if yes, than what are those. 
 
This method should be always started before apply by cli wrapper.

General reason to this method is that after we "smart initialized" config, we might have wanted to change some values some way, and then it has to be validated. Another scenario would be `influence` mechanism I [described in design part](../1-design/README.md) in `Influences` section. In that scenario it's easy to imagine that output of BMK module would produce proposed changes to `BareMetalMonitoring` module or even apply them to its config file. That looks obvious, that automatic "apply" operation on `BareMetalMonitoring` module is not desired option. So we want to suggest to the user "hey, I applied Kafka module, and usually it influences configuration of Monitoring module, so go ahead and do `plan` operation on it to check changes". Or we can even do automatic "plan" operation and show what are those changes. 

## Apply

`apply` is main "logic" module. 

## Audit

# Check that required methods are implemented