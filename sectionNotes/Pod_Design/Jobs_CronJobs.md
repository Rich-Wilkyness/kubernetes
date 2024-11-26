# Docker vs Kubernetes (Single Processes)
- Multiple types of processes can be run in a container (e.g., a web server, a database, etc.).
    - A container can also run a single process like a calculation, script, or command.
        - **Docker Example**: `docker run ubuntu expr 3 + 2` will run the command in the container and then exit.
        - **Kubernetes Example**: In Kubernetes, we create a pod with the same command. The pod will run the command and then enter the `Completed` state. Depending on the restart policy, the pod may restart and run the command again until it reaches the specified number of completions.
            - This process is defined by `spec.restartPolicy`, which can be `Always` (default), `OnFailure`, or `Never`.
            - You can create a job as a Pod file, but it is better to specify the job in a Job file. for separation of concerns, reusability, and flexibility.

# Jobs
- Jobs are used to run pods in a controlled manner.

- Example
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: math-add-job
spec:
  completions: 3
  parallelism: 3
  template:
    spec:
      containers:
        - name: math-add
          image: ubuntu
          command: ['expr', '3', '+', '2']
      restartPolicy: Never
  backoffLimit: 4 
```
- `spec.completions`: The number of successful completions required for the job to be considered complete.
- `spec.parallelism`: The number of pods that should run at the same time.
    - If a pod fails, it will be retried until it reaches the specified number of completions. 
    - If only 1 completion is required, the job will not run multiple pods at the same time. It will only run the necessary number of pods to reach the completion, while staying within the parallelism limit.
- `spec.backoffLimit`: The number of retries before giving up.

- Commands: 
    - `k create -f job-definition.yaml --save-config`
    - `k get jobs`
    - `k delete job math-add-job`
        - Deleting the job will also result in the deletion of the pods created by the job.


# CronJobs
- CronJobs are used to run pods at a specific time or at regular intervals.
- First create a job file, then move the job.spec under `jobTemplate` in the cron job file.
- Commands:
    - `k create -f cronjob-definition.yaml --save-config`
    - `k get cronjobs`
    - `k delete cronjob reporting-cron-job`
        - Deleting the cron job will also result in the deletion of the pods created by the cron job.

- Example
```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: reporting-cron-job
spec:
  schedule: "*/1 * * * *"
  jobTemplate:
    spec:
      completions: 3
      parallelism: 3
      backoffLimit: 4
      template:
        spec:
          containers:
            - name: math-add
              image: ubuntu
              command: ['expr', '3', '+', '2']
          restartPolicy: Never
```

- Schedule: "*/1 * * * *"
            # |  | | | |
            # |  | | | +----- day of the week (0 - 7) (Sunday=0 or 7)
            # |  | | +------- month (1 - 12)
            # |  | +--------- day of the month (1 - 31)
            # |  +----------- hour (0 - 23)
            # +------------- min (0 - 59)
- This cron job will run every minute, every hour, every day, every month, every day of the week
- Other Examples:
    - `*/5 * * * *`: Every 5 minutes
    - `0 * * * *`: Every hour
    - `0 0 * * *`: Every day at midnight
    - `0 0 * * 0`: Every Sunday at midnight
    - `0 0 1 * *`: Every first day of the month at midnight
    - `0 0 1 1 *`: Every first day of January at midnight
