### Integrating Kubernetes with 1Password for infrastructure secrets

I am going to follow This [blog](https://blog.bennycornelissen.nl/post/onepassword-on-kubernetes/)

I have created ```jack``` vault

To test the secret and deployment I have created below manifests


```
apiVersion: onepassword.com/v1
kind: OnePasswordItem
metadata:
  name: jack-password
spec:
  itemPath: "vaults/jack/items/jack-password"

```

Note: ```jack-password``` needs to be created beforehand in ```jack``` vault



To test the Deployment, use below manifest

```
apiVersion: apps/v1
kind: Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis
  annotations:
    operator.1password.io/item-path: "vaults/jack/items/redis-secret"
    operator.1password.io/item-name: "redis-secret"
  namespace: default
  labels:
    app: redis
spec:
  selector:
    matchLabels:
      app: redis
  template:
    metadata:
      labels:
        app: redis
    spec:
      containers:
      - name: redis
        image: redis:6
        imagePullPolicy: Always
        ports:
        - name: client
          containerPort: 6379
        env:
          - name: REDIS_PASS
            valueFrom:
              secretKeyRef:
                name: redis-secret
                key: password
        livenessProbe:
          exec:
            command:
            - sh
            - -c
            - 'redis-cli -h redis.redis.svc.cluster.local ping'

```

Note: redis-secret need to be created in ```jack``` vault


- Useful Commands

```
kubectl get secret redis-secret  -o go-template='
{{range $k,$v := .data}}{{printf "%s: " $k}}{{if not $v}}{{$v}}{{else}}{{$v | base64decode}}{{end}}{{"\n"}}{{end}}'

```

```
kubectl get secret jack-password  -o go-template='
{{range $k,$v := .data}}{{printf "%s: " $k}}{{if not $v}}{{$v}}{{else}}{{$v | base64decode}}{{end}}{{"\n"}}{{end}}'

```

Above commands will show the created secret data