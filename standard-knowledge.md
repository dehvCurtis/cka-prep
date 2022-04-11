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
  - image: nginx:latest
    name: name
```

