# Part 16: Commands

If a docker image specifies a command, it will be executed every time the container starts. However, this can be a problem if you want to make it configurable. For example, an Ubuntu docker image may have a command to sleep for 5 seconds. Every time the container starts, it will sleep for 5 seconds and then exits. 

```
...
CMD ["sleep", "5"]
docker run unbuntu-sleeper
```

We could overwrite this be providing a command to overwrite the defaults.

```
docker run ubuntu-sleeper sleep 5
```

But this doesn't look right; the ubuntu-sleeper should already know how to sleep. We only want to configure the amount of time to sleep, and not the command itself. To do this, we can specify an entrypoint that serves as the command.

```
ENTRYPOINT ["sleep"]
CMD ["5"]
```

Now if we want to overwrite the command, we can specify that as the argument.

```
docker run ubuntu-sleeper 5
```

However, it is still possible to overwride the ENTRYPOINT. We do this using the `--entrypoint` command-line option.

```
docker un ubuntu-sleeper --entrypoint awake 5
```

In Kubernetes, the corresponding pod definition would look something like this:

```
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: sleeper
  name: sleeper
spec:
  containers:
  - image: ubuntu
    name: sleeper
    command: ["sleep"]
    args: ["3"]
```


