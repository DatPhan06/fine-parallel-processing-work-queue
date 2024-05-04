# Kubernetes Parallel Job Processing with Redis Work Queue

## Overview

This project demonstrates how to use Kubernetes to process jobs in parallel from a Redis work queue. Each job processes tasks retrieved from a Redis queue. This setup showcases how Kubernetes can efficiently distribute and handle tasks in a distributed system.

## Prerequisites

Before you start, you need:

- A Kubernetes cluster (You can use Minikube or a managed Kubernetes service like Google Kubernetes Engine).
- `kubectl` command-line tool configured to communicate with your cluster.
- Docker installed on your local machine to build and push the Docker image.
- Access to a Docker registry (like Docker Hub) where you can push the Docker image.

## Setting Up the Environment

### Step 1: Start Redis

First, deploy Redis in your Kubernetes cluster which will be used as the task queue:

```
kubectl apply -f https://k8s.io/examples/application/job/redis/redis-pod.yaml
kubectl apply -f https://k8s.io/examples/application/job/redis/redis-service.yaml
```

### Step 2: Populate the Redis Queue

You need to populate the Redis queue with tasks. Start a temporary interactive pod to access Redis CLI:

```
kubectl run -i --tty temp --image redis --command "/bin/sh"

```

Once the shell is ready, connect to the Redis instance and add tasks:

```
redis-cli -h redis
rpush job2 "task1"
rpush job2 "task2"
... (add more tasks as needed) ...
lrange job2 0 -1
```

Exit the interactive session:

```
exit

```

### Step 3: Building and Pushing the Docker Image

Build the Docker image that includes your job application:

```
docker build -t yourusername/job-wq-2 .
docker push yourusername/job-wq-2
```

Replace 'yourusername' with your Docker Hub username.

### Step 4: Define and Run the Job

Create a job definition that uses the Docker image:

```
apiVersion: batch/v1
kind: Job
metadata:
  name: job-wq-2
spec:
  parallelism: 2
  template:
    spec:
      containers:
      - name: worker
        image: yourusername/job-wq-2:latest
        env:
        - name: REDIS_HOST
          value: "redis"
      restartPolicy: Never

```

Apply this configuration to your cluster:

```
kubectl apply -f job.yaml

```

### Step 5: Monitor and Manage the Job

Check the status of the job:

```
kubectl describe jobs/job-wq-2
```

View the logs of the pods to see the progress:

```
kubectl logs -f job-name=job-wq-2
```

## Cleaning Up

To clean up all resources used in this demo, delete the job and the Redis deployment:

```
kubectl delete job job-wq-2
kubectl delete pod redis
kubectl delete svc redis
```

## Conclusion

This demo illustrates how Kubernetes can be used to manage parallel processing of tasks using a Redis work queue. This setup can be adapted for various use cases where task distribution and management are critical.
