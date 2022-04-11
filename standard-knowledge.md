### Create basic pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: name
  name: name
spec:
  containers:
  - image: busybox:latest
    name: name
    args:
      - sleep
      - "3600"
```

