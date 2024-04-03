# ECHO CPU Scheduler

**Enhanced CPU Handling Orchestrator**

It is a CPU processes scheduler patch for Linux kernel.

This scheduler includes the following features: -

- Highly multitasking handling with max 35us quota.
- All tasks in a CPU have a shared quota = 35us in which every task runs (35us / # of tasks)
- Minimum slice for a running a task is 7us unless waked up task that must run before the current task then it preempts it.
- Calculate the estimation of tasks - SRTF (Shortest Remaining Task Next). This uses moving average to calculate virtual runtime.
- Next task is picked with the smallest estimated virtual runtime.
- Load balancer as in TT scheduler with tiny changes. CPU0 is responsible of moving tasks among other CPUs. Also, the candidate
balancer is enabled by default.

## Comparison with other schedulers

https://github.com/hamadmarri/benchmarks



## Policy
The policy is a mix of SRTF and RR (Round Robin) where virtual runtime calculation is
ported from CFS (it calculates the burst adjusted based on the priority of the task). Each round the tasks will run starting from
the least estimated vruntime and each task will run `shared_quota/#tasks` ex. `35us / 3 = ~11.7us`

If a wake up task has smaller estimated vruntime then it will preempt the current task and run. Every time the task consumes its
quota it will be placed in a second queue unless it is the only task that is running. After finishing the round, all tasks are placed
in the second queue. The scheduler switches the queue head from q1 to q2, and q2 become q1 and vise versa.


## Defaults and Sysctls
- The default HZ for ECHO is 625HZ - ticks every 1.6ms. No need to increase it since the HighRes clock handles the task preemption in 35us max.
- `kernel.sched_bs_shared_quota` by default is 35000 (35us) can be tuned with sysctl
ex. `sysctl kernel.sched_bs_shared_quota=4800000` larger values saves CPU caches but reduces interactivity and multitasking.
- There are kernel configurations that must be disabled:
	- CONFIG_FAIR_GROUP_SCHED
	- CONFIG_SCHED_AUTOGROUP
	- CONFIG_SCHED_CORE


## Telegram Group

https://t.me/tt_sched

Hamad
