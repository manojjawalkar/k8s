# Jobs
- Jobs are defined as some work that you need to do once in a lifecycle. The aim of that pod is to do the task and then shut down forever.
- But by default k8s restarts a pod if it finds it to be shut down
- This is defined using a parameter called `restartPolicy: Always` in teh pods definition file.
- We need to change the default policy from Always to Never if we want it to not restart.

Below is the example of the pod that is set to restart always. You can see under RESTARTS column how many times k8s attempted to restart it

```
controlplane $ k get pods
NAME             READY   STATUS      RESTARTS       AGE
dataprocessor    0/1     Completed   5 (100s ago)   3m20s


controlplane $ k get pods
NAME             READY   STATUS             RESTARTS      AGE
dataprocessor    0/1     CrashLoopBackOff   5 (22s ago)   3m32s
```

- To override the default behaviour, edit the pod and change the policy
```
controlplane $ k edit pods dataprocessor
# change the value to Never on restartPolicy
# save the change, it won't allow you to save it. press ESC+q!
# a new file with the change it stored in /tmp directory
# To delete the original pod and replace it with the new one, use the below command

controlplane $ k replace --force -f /tmp/kubectl-edit-xxxxx.yaml
# in my case it was

controlplane $ k replace --force -f /tmp/kubectl-edit-1761627100.yaml
```

## Why Jobs
There can be multiple jobs that we need to run and make sure each of them are completed successfully. To monitor this, we make use of jobs. Job is similar to replicas but unlike replicas, the job is use to run set of pods to perform given task to completion

## Creating jobs
The job definition file looks similar to replica definition. The kind here is job and the apiVersion is batch/v1. Under the template section, you copy paste the pods definition as shown below
```
apiVersion: batch/v1
kind: job
metadata:
	name: dataprocessor
spec:
	template:
		spec:
			containers:
				- name: math-add
				image: ubuntu
				command: ['expr','5','+','4']
			restartPolicy: Never
```

```
controlplane $ k get jobs
NAME            COMPLETIONS   DURATION   AGE
dataprocessor   0/1           4s         4s



controlplane $ k get pods
NAME                  READY   STATUS      RESTARTS   AGE
dataprocessor-plth4   0/1     Completed   0          9m28s
```

## Checking output
```
controlplane $ k logs dataprocessor-plth4
9
```

### Imperative command
```
k create job throw-dice-job --image=kodekloud/throw-dice 
```



# Completions and Parallelism
- In above example it is just one pod that is created. What if we want multiple pods to be created. For this we have a property called completions
- We set completions under the spec of the job to whatever number of pods we want to run
```
apiVersion: batch/v1
kind: Job
metadata:
	name: dataprocessor
spec:
	completions: 3
	template:
		spec:
			...
```

- The pods are created sequentially. That means second pod is created only after the first one is created and exited sucessfully. Similarly the next
-  What if **pod fails**?
-  If the pod fails, it creates another one until there are 3 successful completions (since we specified `completions: 3`) in our spec.
- To create pods in parallel, add a property called parallelism
```
apiVersion: batch/v1
kind: Job
metadata:
	name: dataprocessor
spec:
	completions: 5
	parallelism: 2
	template:
		spec:
			...
```

- Here it will create 2 pods parallely. so 2 + 2 + 1. It is smart enough to create only one in the last go so that it matches the completions count of 5
```
apiVersion: batch/v1
kind: Job
metadata:
  name: dataprocessol
spec:
  completions: 11
  parallelism: 2
  template:
    spec:
      containers:
        - name: mathadd
          image: kodekloud/random-error
          command: ['expr','5','+','4']
      restartPolicy: Never
~                               
```

```
controlplane $ k create -f job4.yaml 
job.batch/dataprocessol created
controlplane $ k get jobs
NAME            COMPLETIONS   DURATION   AGE
dataprocesso3   3/3           15s        5m55s
dataprocessol   2/11          7s         7s
dataprocessor   1/1           14s        7m14s
controlplane $ k get jobs --watch
NAME            COMPLETIONS   DURATION   AGE
dataprocesso3   3/3           15s        6m
dataprocessol   4/11          12s        12s
dataprocessor   1/1           14s        7m19s
dataprocessol   4/11          13s        13s
dataprocessol   5/11          13s        13s
dataprocessol   5/11          14s        14s
dataprocessol   5/11          16s        16s
dataprocessol   6/11          16s        16s
dataprocessol   6/11          18s        18s
dataprocessol   7/11          18s        18s
dataprocessol   7/11          21s        21s
dataprocessol   8/11          21s        21s
dataprocessol   8/11          22s        22s
dataprocessol   9/11          22s        22s
dataprocessol   9/11          26s        26s
dataprocessol   10/11         26s        26s
dataprocessol   10/11         27s        27s
dataprocessol   11/11         27s        27s
```


# Cron jobs
- Cron jobs are same as jobs only thing they can be run on a specific time. 
- A regular job runs as soon as the pod is created. In Cron jobs we have the control over when we want to run the job.
- This is achieved using `schedule` property
- Schedule is similar to linux cron jobs
- Schedule is specified under spec section and below that is jobTemplate which contains spec section of a job.
```
apiVersion: batch/v1
kind: CronJob
metadata:
  name: reportingcronjob

spec:
  schedule: "*/1 * * * *"
  jobTemplate:
    spec:
      completions: 3
      parallelism: 3
      template:
        spec:
          containers:
            - name: math-add
              image: ubuntu
              command: ['expr','5','+','4']
          restartPolicy: Never
```

```
controlplane $ k get cronjobs
NAME               SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
reportingcronjob   */1 * * * *   False     0        <none>          6s
```

Check the `Pods statuses` value inside `k describe job <job_name>` to see if we got any pods in the completed state

Total attemps to complete the job is calculated as sum of failed and succedded
```
controlplane $ k describe jobs reportingcronjob-284521 | grep Pods
Pods Statuses:    0 Active (0 Ready) / 3 Succeeded / 0 Failed
```