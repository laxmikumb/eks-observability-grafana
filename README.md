 README.md
├── auth-egress-restrict.yaml
├── auth-service
│   ├── deployment.yaml
│   └── service.yaml
├── cluster
│   └── minikube-start.sh
├── data-service
│   ├── deployment.yaml
│   └── service.yaml
├── debug
│   ├── auth-service-sa.yaml
│   ├── debug-auth.yaml
│   ├── debug-data.yaml
│   └── debug-sa.yaml
├── failure-simulation
│   └── failure-commands.txt
├── gateway
│   ├── deployment.yaml
│   ├── ingress.yaml
│   └── service.yaml
├── httpbin
│   ├── deployment.yaml
│   └── service.yaml
├── microservices
│   ├── auth-service
│   │   ├── deployment.yaml
│   │   └── service.yaml
│   ├── data-service
│   │   ├── deployment.yaml
│   │   └── service.yaml
│   └── gateway
│       ├── deployment.yaml
│       ├── ingress.yaml
│       └── service.yaml
├── minio
│   ├── minio-deployment.yaml
│   ├── minio-secret.yaml
│   └── minio-service.yaml
├── minio-iam
│   ├── debug-pods
│   │   ├── debug-auth.yaml
│   │   └── debug-data.yaml
│   ├── minio-deployment.yaml
│   ├── minio-secret.yaml
│   ├── minio-service.yaml
│   └── serviceaccount-data.yaml
├── namespace
│   └── namespace.yaml
├── namespaces
│   ├── app-namespace.yaml
│   ├── namespaces.yaml
│   └── system-namespace.yaml
├── network-policies
│   └── app-network-policy.yaml
├── networkpolicies
│   └── auth-egress.yaml
├── observability
│   ├── alerts
│   │   └── pod-restart-alert.yaml
│   ├── grafana
│   │   ├── alerts
│   │   └── dashboards
│   ├── grafana-dashboards
│   │   └── app-observability.json
│   ├── prometheus
│   └── prometheus-install.sh
├── rbac
│   ├── minio-access-binding.yaml
│   └── minio-access-role.yaml
├── security
│   ├── auth-deployment-fixed.yaml
│   └── networkingpolicy-auth-egress.yaml
└── serviceaccounts
    ├── auth-service-sa.yaml
    └── data-service-sa.yaml

architecture digram:

&nbsp;                  |──────────────┐
&nbsp;                   │   Browser    │
&nbsp;                   │ (User/Admin) │
&nbsp;                   └──────┬───────┘
&nbsp;                          │
&nbsp;                   Ingress (NGINX)
&nbsp;                          │
&nbsp;       ┌──────────────────┼──────────────────┐
&nbsp;       │                  │                  │
&nbsp; data-service        minio-service      grafana-service
&nbsp;       │                  │                  │
&nbsp;    Pod(s)            Pod(s)             Pod(s)
&nbsp;       │                  │                  │
&nbsp;       └──────────────┬───┴──────────────┘
&nbsp;                      │
&nbsp;               Kubernetes Cluster
&nbsp;                      │
&nbsp;       ┌──────────────┴──────────────┐
&nbsp;       │        Observability         │
&nbsp;       │  Prometheus + Alertmanager  │
&nbsp;       └─────────────────────────────┘

Design Decisions (Why This Way)

Microservices
Gateway simulates an API Gateway / ALB auth-service handles authentication data-service accesses object storage

IAM Simulation
Kubernetes ServiceAccount + Secret Only data-service can access MinIO auth-service is denied even if misconfigured

Security
NetworkPolicy enforces zero-trust Explicit allow: auth-service → data-service

All other egress blocked

Observability
kube-prometheus-stack provides:Node metrics
Pod metrics
Restart counts
Grafana dashboards imported (ID: 6417)







Observability Explanation (Important)

“Dashboards showed data even though I didn’t write queries”

This is expected behavior.

kube-prometheus-stack ships with:

Predefined metrics

Auto-discovered targets

Imported dashboards (ID: 6417) already contain queries

That’s why CPU, memory, pod status appeared automatically



Assumptions \& Known Issues

Assumptions

Local environment (Minikube)

No real AWS IAM (simulated with Kubernetes primitives)

Applications are mock containers

Known Issues

HTTP request metrics require app instrumentation

Minikube networking differs from real EKS

Falco / OPA not fully enforced (policy examples only)






