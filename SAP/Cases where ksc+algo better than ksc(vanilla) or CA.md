
### <u>ScaleUp</u>
#### Case 1 (case-up-3)

```
(single zone workerpool)
PodA : 5Gb -> 20 Replicas
NG1 : m5.large -> 8Gb; NG1Max: 12
NG2 : m5.4xlarge -> 64Gb; NG2Max: 4

(no effect of pod order here, as only one pod type)
Result (optimal) -> 2 * NG2 (lower fragmentation)
Result of CA -> 2 * NG2
Result of using ksc(vanilla) -> 12 * NG1 + 1 * NG2
Result of using ksc+algo (lW = 1,lC = 1) -> 8 * NG1 + 1 * NG2 
Result of using ksc+algo (lW = 1,lC = 1.5) -> 11 * NG1 + 1 * NG2 
```

#### Case 2 (case-up-2)

```
(single zone workerpool)
PodA : 5Gb -> 10 Replica
PodB : 12Gb -> 1 Replica
NG1 : m5.large -> 8Gb; NG1Max: 12
NG2 : m5.xlarge -> 16Gb; NG2Max: 5
NG3 : m5.4xlarge -> 64Gb; NG3Max: 5

Result (optimal) ->  1 * NG3 + 1 * NG1
Result of CA -> 1 * NG3 + 1 * NG1
Result of using ksc(vanilla) -> 10 * NG1 + 1 * NG2
Result of using ksc+algo (lW = 1,lC = 1, pd = none) -> 1 * NG2 + 1 * NG3 (cannot reach optimal as it does not consider pod order.)
Result of using ksc+algo (lW = 1,lC = 1, pd = desc) -> 1 * NG1 + 1 * NG3 
Result of using ksc+algo (lW = 1,lC = 1.5,pd = none) -> 1 * NG2 + 1 * NG3 
Result of using ksc+algo (lW = 1,lC = 1.5,pd = desc) -> 1 * NG1 + 1 * NG3 
```

#### Case 3 (case-up-3)

```
(single zone workerpool)
PodA : 5Gb -> 11 Replicas
PodB : 12Gb -> 1 Replicas
NG1 : m5.large -> 8Gb; NG1Max: 12
NG2 : m5.4xlarge -> 64Gb; NG2Max: 4

Result (optimal) ->  1 * NG2 + 2 * NG1
Result of CA -> 1 * NG2 + 2 * NG1
Result of using ksc(vanilla) -> 11 * NG1 + 1 * NG2
Result of using ksc+algo (lW = 1,lC = 1, pd = none) -> 2 * NG2
Result of using ksc+algo (lW = 1,lC = 1, pd = desc) -> 2 * NG1 + 1 * NG2
Result of using ksc+algo (lW = 1,lC = 1.5,pd = none) -> 11 * NG1 + 1 * NG2
Result of using ksc+algo (lW = 1,lC = 1.5,pd = desc) ->  11 * NG1 + 1 * NG2
```

### Case 4 (case-up-4)

```
(single zone workerpool with each workerpool in different zone)
PodA: 5GB -> 10 replicas  
   TSC: topology-key = `zone`, maxSkew=1, whenUnsatisfiable=`DoNotSchedule`  
PodB: 12GB -> 2 replicas  
   TSC: topology-key = `node`, maxSkew = 1, whenUnsatisfiable=`DoNotSchedule`  
NG1 -> m5.Large -> Zone-A (8gb)  (Max : 12)
NG2 -> m5.xLarge -> Zone-B (16 gb)  (Max : 5)
NG3 -> m5.4xLarge -> Zone-C (64 gb) (Max : 5)


Result (optimal) ->  3 * NG1, 3 * NG2, 1 * NG3
Result of CA -> 
Result of using ksc(vanilla) -> 
Result of using ksc+algo (lW = 1,lC = 1, pd = none) -> 2 * NG2
Result of using ksc+algo (lW = 1,lC = 1, pd = desc) -> 3 * NG1 + 4 * NG2 + 1 * NG3
Result of using ksc+algo (lW = 1,lC = 1.5,pd = none) -> 11 * NG1 + 1 * NG2
Result of using ksc+algo (lW = 1,lC = 1.5,pd = desc) ->  11 * NG1 + 1 * NG2

```

### <u>ScaleDown</u>

### Case 1 (scenario-a)

```
PodA: 5GB -> 10 replicas
PodB: 12GB -> 1 replicas
NG1 -> m5.large -> 7
NG2 -> m5.xlarge -> 1

Initial pod to node distribution : 
Result (optimal) : 
Result of scaledown algo : 
```

### Case 2 (scenario-a)

```
PodA: 2GB -> 7 replicas
NG1 -> m5.large -> 7
NG2 -> m5.xlarge -> 1

Initial pod to node distribution : 
Result (optimal) : 
Result of scaledown algo : 
```

### Case 3 (case-up-4)

```
(single zone workerpool with each workerpool in different zone)
PodA: 5GB -> 10 replicas  
   TSC: topology-key = `zone`, maxSkew=1, whenUnsatisfiable=`DoNotSchedule`  
PodB: 12GB -> 2 replicas  
   TSC: topology-key = `node`, maxSkew = 1, whenUnsatisfiable=`DoNotSchedule`  
NG1 -> m5.Large -> Zone-A (8gb)  (Max : 12)
NG2 -> m5.xLarge -> Zone-B (16 gb)  (Max : 5)
NG3 -> m5.4xLarge -> Zone-C (64 gb) (Max : 5)

Initial pod to node distribution : 
Result (optimal) : 
Result of scaledown algo : 
```

