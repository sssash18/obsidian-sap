[[MCM-CA Overhaul POC]]
### Algo 1 : <u>Scaleup one machine at a time</u>

1. We decide to just scaleup one machine in one iteration.
2. Then for each kind of machine we try to fit max pods we can in that machine.
3. We calculate a score for each machine, where score is =:> waste(in %) + (unscheduled pods after scaling this machine)/(total unscheduled pods in pool).
4. We create virtual node for only that machine with least score and perform the simulation then.