```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-container
        image: my-image:latest
        ports:
        - containerPort: 80
        volumeMounts:
        - name: my-pvc-mount
          mountPath: /data
        resources:
          limits:
            cpu: "500m"
            memory: "512Mi"
          requests:
            cpu: "250m"
            memory: "256Mi"
      volumes:
      - name: my-pvc-mount
        persistentVolumeClaim:
          claimName: my-pvc

---
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: my-app
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80

---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-ingress
  annotations:
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
spec:
  rules:
  - host: my-app.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: my-service
            port:
              number: 80
  tls:
  - hosts:
    - my-app.example.com
    secretName: my-tls-secret

---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

becomes

```starlark
Deployment(
    name = "my-deployment",
    replicas = 3,
    containers = [
        Container(
            name = "my-container",
            image = "my-image:latest",
            ports = [80],
            volumeMounts = [
                VolumeMount(
                    name = "my-pvc-mount",
                    mountPath = "/data"
                )
            ],
            resources = ResourceRequirements(
                limits = ResourceLimits(
                    cpu = "500m",
                    memory = "512Mi"
                ),
                requests = ResourceRequests(
                    cpu = "250m",
                    memory = "256Mi"
                )
            )
        )
    ],
    volumes = [
        Volume(
            name = "my-pvc-mount",
            persistentVolumeClaim = PersistentVolumeClaimVolumeSource(
                claimName = "my-pvc"
            )
        )
    ]
)

Service(
    name = "my-service",
    selector = "app: my-app",
    ports = [80]
)

Ingress(
    name = "my-ingress",
    annotations = {
        "nginx.ingress.kubernetes.io/ssl-redirect": "true"
    },
    rules = [
        IngressRule(
            host = "my-app.example.com",
            http = HTTPIngressRuleValue(
                paths = [
                    HTTPIngressPath(
                        path = "/",
                        backend = IngressBackend(
                            serviceName = "my-service",
                            servicePort = 80
                        )
                    )
                ]
            )
        )
    ],
    tls = [
        IngressTLS(
            hosts = ["my-app.example.com"],
            secretName = "my-tls-secret"
        )
    ]
)

PersistentVolumeClaim(
    name = "my-pvc",
    spec = PersistentVolumeClaimSpec(
        accessModes = ["ReadWriteOnce"],
        resources = ResourceRequests(
            requests = Storage("1Gi")
        )
    )
)
```
