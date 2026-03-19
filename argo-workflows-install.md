# Argo Workflows (Central Server + Team Isolation)

This setup installs Argo Workflows with a single shared UI/API (argo-server) and uses SSO RBAC + namespace delegation so teams can only see and operate Workflows in their own namespace.

## Install Argo Workflows
Check github release
https://github.com/argoproj/argo-workflows/releases

Argo Server and Controller will be installed on Namespace argo.
```
kubectl create namespace argo
kubectl apply --server-side -n argo https://github.com/argoproj/argo-workflows/releases/download/v4.0.0/namespace-install.yaml
```

# Workflow will be run on Namespace data-science-workflows
```
kubectl create ns data-science-workflows
```
# SSO
mainfest/argo/argo-workflows-sso-secret.yaml
## 1. Create secret for connecting the oidc provider
```
apiVersion: v1
kind: Secret
metadata:
  name: argo-workflows-sso-clientsecret
stringData:
  client-secret: dGIsKYgfTb1rZS3wyXw8y8e8TRkVbfTu
```
mainfest/argo-workflows-secret.yaml
```
apiVersion: v1
kind: Secret
metadata:
  name: argo-workflows-sso-clientid
  namespace: argo
stringData:
  client-id: argo-workflows  
type: Opaque

```

## 2. Add SSO config to configmap name workflow-controller-configmap
manifest/argo/workflow-controller-configmap.yaml
```
  sso: |
    # This is the root URL of the OIDC provider (required).
    issuer: https://keycloak.rancher.local/realms/devops
    # This is name of the secret and the key in it that contain OIDC client
    # ID issued to the application by the provider (required).
    clientId:
      name: argo-workflows
      key: client-id
    # This is name of the secret and the key in it that contain OIDC client
    # secret issued to the application by the provider (required).
    clientSecret:
      name: argo-workflows-sso
      key: client-secret
    # This is the redirect URL supplied to the provider (required). It must
    # be in the form <argo-server-root-url>/oauth2/callback. It must be
    # browser-accessible.
    redirectUrl: https://argo-workflows.rancher.local/oauth2/callback 
    scopes:
     - groups
    rbac:
      enabled: true  
    rootCA: |
       -----BEGIN CERTIFICATE-----
       MIIE8jCCAtqgAwIBAgIQTHxTOnW+R+SDWYNmGii0rDANBgkqhkiG9w0BAQsFADAT
       MREwDwYDVQQDEwhrZXljbG9hazAeFw0yNTAxMTUxMjQ0MTNaFw0zMzAxMTUxMjQ0
       MTNaMBMxETAPBgNVBAMTCGtleWNsb2FrMIICIjANBgkqhkiG9w0BAQEFAAOCAg8A
       MIICCgKCAgEAq8XlusExsQJHGUeurqPmScVtu1hb7nVWDacwRISqrdZmRHfZ3PqC
       BBv8ucavcrjj8zUMfBmW3Pyj7zHZHaLCJ1KwY0KzA5y2LZsgdvJRPB0j1mmdrg1M
       zgEYqh5jwKCjQSSzZEOiOo4h0lrFFOnLzkwIyiQrXZBj1c+fTVsYTQYtuWuOtQa9
       AGo5Cl6Wx5xvqW8E+MmvHobcOeT9cmAfELiAoRzW7D+KTTnW34tHK5uoD4vicyi2
       Y0wO6khIgBBh6tdx/pyiosN79OGu46dXipZKX2HuzOVi1dpXS3D3S/XZb0Otc4lq
       gox7f9lL7NB0hEkHJ1FDVoezahgiwE0LQT0BgR9iEjH5fP9MylbpsqYPXM4KOI6f
       Usgse5onVw27JEK1HLaDbtSkzz+YcRd9yiNuCKQUAlNDkxeBOQabDY9QtadnSam7
       rWeSbSVJHN6VMKIu/rcWifINCcwz2Lc1Uxh9Kl7w0pr45VACirMV+uwOrsXsLU9i
       mGbZ4cmMo9VVKik47/hGHh3UsOoO6NH9CFk+d5KFoY4MuR7nRHnH/p/Yr9zZe3YT
       QvCLNorIU9iHYaBk8nsl1vLjvp4l5OFEMLE3sG7qTTkvgi0gjSYw8zVM6cc39mo+
       G84APQGiM3EEbGC3ve0Tb9XM2tA3bKaMq+ZvmWVxa7y2o3xUXdnnjNcCAwEAAaNC
       MEAwDgYDVR0PAQH/BAQDAgKkMA8GA1UdEwEB/wQFMAMBAf8wHQYDVR0OBBYEFOTz
       QMSNQO3xPOysqPo2e3rUEn8HMA0GCSqGSIb3DQEBCwUAA4ICAQA/crWhOKrtUSgP
       bnKCze//CH4TmTD/X+W+ms3In9o1YON+9uknniP/cqBDIAc8xKlgDRk/0GmqTCY8
       G3c5eqSVYaXIPgIDutplf1KJ9/EIMxL73JTNpQ1JDA8cG1/3CSFJOFMUIPWMUnmp
       He9YxBZ8ybg40r7UJU8MiQg1KVU0fSuB49d9pZJjO/gft1wPCFwMBR3LZn3F02Yz
       N5hHjB66uviyd6kEnWMac7TpVRMTPmajxPjCYVIN3azaPF1udOyEjF6msZNhrESL
       shGoK3uK2+NPX6sW9o60rBNuNhG9n+rJry7UpTRBZHb5tUnnEjmn8eildA7HeIX7
       c+O1I73MikVZK6D0P3WWCuWBSv/N4MoUWVm7NxRYHq9dfh/Oa4EDk+hKEbYujrw0
       uadTVnAY7hT4qRzNr2RroIJPItdpnA7R/UVgs0I+Dyfj+MTfQGl9+y+P0JMU0YY+
       es/9OKrcTWMrtulIC8+DxlqpR1q0dt9rda06vPpV1bWKUS2k4/zw/Sdzbv+ukgnV
       h5i7dC3KQnO2R3PJeqES2dyAGDdr1gNmJjw4se/5XEMqaQAab2xk0MhRY03nI4el
       D8ALvdRiGTKCKNPdWOYKmm8+lW+blww5caQEY3niwORZmzJoJLDJKpMR4wXPKk15
       ATmPf1pssq4alcp1upLaR4L7s/IysQ==
       -----END CERTIFICATE-----
```
## 3. Update argo server deployment
manifest/argo/argo-server-deployment.yaml
Edit argo server deployement, add --configmap and --auth-mode=sso
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: argo-server
  namespace: argo
spec:
  progressDeadlineSeconds: 600
  replicas: 1
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app: argo-server
  template:
    metadata:
      labels:
        app: argo-server
    spec:
      containers:
      - args:
        - server
        - --configmap
        - workflow-controller-configmap
        - --auth-mode=sso
# for managed namespace install        
#        - --namespaced
#        - --managed-namespace
#        - data-science-workflows
        image: quay.io/argoproj/argocli:v4.0.0
        imagePullPolicy: IfNotPresent
        name: argo-server
        ports:
        - containerPort: 2746
          name: web
          protocol: TCP
        readinessProbe:
          failureThreshold: 3
          httpGet:
            path: /
            port: 2746
            scheme: HTTPS
          initialDelaySeconds: 10
          periodSeconds: 20
          successThreshold: 1
          timeoutSeconds: 1
        resources: {}
        securityContext:
          allowPrivilegeEscalation: false
          capabilities:
            drop:
            - ALL
          privileged: false
          readOnlyRootFilesystem: true
          runAsNonRoot: true
          seccompProfile:
            type: RuntimeDefault
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
        volumeMounts:
        - mountPath: /tmp
          name: tmp
      dnsPolicy: ClusterFirst
      nodeSelector:
        kubernetes.io/os: linux
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext:
        runAsNonRoot: true
      serviceAccount: argo-server
      serviceAccountName: argo-server
      terminationGracePeriodSeconds: 30
      volumes:
      - emptyDir: {}
        name: tmp

```

# RBAC
Create K8S service account to match the AD/LDAP group 
Group:
- data-service
- devops
- admin
## 1. Grant service account argo-workflows-server can create secret
```
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: argo-workflows-server-secret-writer
  namespace: argo
rules:
- apiGroups: [""]
  resources: ["secrets"]
  verbs: ["create"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: argo-workflows-server-secret-writer
  namespace: argo
subjects:
- kind: ServiceAccount
  name: argo-workflows-server
  namespace: argo
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: argo-workflows-server-secret-writer

---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: argo-workflows-server-sa-reader
rules:
- apiGroups: [""]
  resources: ["serviceaccounts"]
  verbs: ["get","list","watch"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: argo-workflows-server-sa-reader
subjects:
- kind: ServiceAccount
  name: argo-workflows-server
  namespace: argo
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: argo-workflows-server-sa-reader


```
## 2. Create Service Account and token
### Create token name must be "serviceaccount-name".service-account-token, for eg: devops.service-account-token
manifest/argo/service-accounts.yaml
```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: sso-dsw
  namespace: data-science-workflows
  annotations:
    workflows.argoproj.io/rbac-rule: "'data-service' in groups"
    workflows.argoproj.io/rbac-rule-precedence: "10"
---
apiVersion: v1
kind: Secret
metadata:
  name: sso-dsw.service-account-token
  namespace: data-science-workflows
  annotations:
    kubernetes.io/service-account.name: sso-dsw
type: kubernetes.io/service-account-token

---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: sso-data-service
  namespace: data-service
  annotations:
    workflows.argoproj.io/rbac-rule: "'data-service' in groups"
    workflows.argoproj.io/rbac-rule-precedence: "10"

---
apiVersion: v1
kind: Secret
metadata:
  name: sso-data-service.service-account-token
  namespace: data-service
  annotations:
    kubernetes.io/service-account.name: sso-data-service
type: kubernetes.io/service-account-token


---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: sso-devops
  namespace: devops
  annotations:
    workflows.argoproj.io/rbac-rule: "'devops' in groups"
    workflows.argoproj.io/rbac-rule-precedence: "10"

---
apiVersion: v1
kind: Secret
metadata:
  name: sso-devops.service-account-token
  namespace: devops
  annotations:
    kubernetes.io/service-account.name: sso-devops
type: kubernetes.io/service-account-token


```
## 3. Create clusterrole for user adminster workflow on Namespace
cli
```
kubectl create rolebinding "cluserrole name" --clusterrole=admin --serviceaccount=argo:"service account" -n argo
```
example
```
kubectl create rolebinding argo-devops-workflows-admin --clusterrole=admin --serviceaccount=argo:devops -n argo
```
manifest/argo/argo-workflows-admin-clusterrole.yaml
Argo admin ClusterRole
```
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: argo-workflows-admin
rules:
  # Kubernetes Events API (newer group)
  - apiGroups: ["events.k8s.io"]
    resources: ["events"]
    verbs: ["get", "list", "watch"]

  - apiGroups:
    - argoproj.io
    resources:
    - workflows
    - workflowtemplates
    - cronworkflows
    - workfloweventbindings
    - workflowtasksets
    - workflowtaskresults
    verbs:
    - get
    - list
    - watch
    - create
    - update
    - patch
    - delete
  - apiGroups:
    - argoproj.io
    resources:
    - sensors
    - eventbus
    - eventsources
    - triggers
    verbs:
    - get
    - list
    - watch
    - create
    - update
    - patch
    - delete
  - apiGroups:
    - argoproj.io
    resources:
    - clusterworkflowtemplates
    verbs:
    - get
    - list
    - watch
    - create
    - update
    - patch
    - delete
  - apiGroups:
    - ""
    resources:
    - pods
    - pods/log
    - pods/exec
    - events
    - configmaps
    - secrets
    - serviceaccounts
    - services
    - persistentvolumeclaims
    verbs:
    - get
    - list
    - watch
    - create
    - update
    - patch
    - delete
  - apiGroups:
    - apps
    resources:
    - deployments
    - replicasets
    - statefulsets
    - daemonsets
    verbs:
    - get
    - list
    - watch
    - create
    - update
    - patch
    - delete
  - apiGroups:
    - batch
    resources:
    - jobs
    - cronjobs
    verbs:
    - get
    - list
    - watch
    - create
    - update
    - patch
    - delete

```
# 4. Create role binding  that grant user in group can admister workflow on namespace
manifest/argo/role-binding.yaml
```
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: sso-dsw-admin
  namespace: data-science-workflows
subjects:
  - kind: ServiceAccount
    name: sso-dsw
    namespace: argo
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: argo-workflows-admin
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: sso-data-service-admin
  namespace: data-service
subjects:
  - kind: ServiceAccount
    name: sso-data-service
    namespace: argo
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: argo-workflows-admin
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: sso-devops-admin
  namespace: devops
subjects:
  - kind: ServiceAccount
    name: sso-devops
    namespace: argo
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: argo-workflows-admin

```

# 5. Grant workflow template RBAC, Admin user only
manifest/argo/clusterworkflowtemplates-rbac.yaml
```
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: argo-clusterworkflowtemplates-admin
rules:
  - apiGroups: ["argoproj.io"]
    resources: ["clusterworkflowtemplates"]
    verbs: ["get","list","watch","create","update","patch","delete"]

---

apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: argo-clusterworkflowtemplates-admin-crb
subjects:
  - kind: ServiceAccount
    name: admin-user
    namespace: data-science-workflows
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: argo-clusterworkflowtemplates-admin

```

# 6. Optional: Grant Admin user adminster all namespace
manifest/argo/admin-user-rbac.yaml
```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: argo
  annotations:
    # The rule is an expression used to determine if this service account
    # should be used.
    # * `groups` - an array of the OIDC groups
    # * `iss` - the issuer ("argo-server")
    # * `sub` - the subject (typically the username)
    # Must evaluate to a boolean.
    # If you want an account to be the default to use, this rule can be "true".
    # Details of the expression language are available in
    # https://expr-lang.org/docs/language-definition.
    workflows.argoproj.io/rbac-rule: "'admin' in groups"
    # The precedence is used to determine which service account to use when
    # Precedence is an integer. It may be negative. If omitted, it defaults to "0".
    # Numerically higher values have higher precedence (not lower, which maybe
    # counter-intuitive to you).
    # If two rules match and have the same precedence, then which one used will
    # be arbitrary.
    workflows.argoproj.io/rbac-rule-precedence: "1"

---
apiVersion: v1
kind: Secret
metadata:
  name: admin-user.service-account-token
  namespace: argo
  annotations:
    kubernetes.io/service-account.name: admin-user
type: kubernetes.io/service-account-token

---

apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: argo-workflows-admin
  namespace: argo
subjects:
  # If you mean a Keycloak/OIDC group named "admin"
  - kind: ServiceAccount
    name: admin-user
    namespace: data-science-workflows

  # If you literally mean a Kubernetes user named "admin", use this instead:
  # - kind: User
  #   name: admin
  #   apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: argo-workflows-admin
  apiGroup: rbac.authorization.k8s.io

---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: argo-clusterworkflowtemplates-admin-crb
subjects:
  - kind: ServiceAccount
    name: admin-user
    namespace: argo
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: argo-clusterworkflowtemplates-admin


---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: sso-adminuser-data-service-admin
  namespace: data-service
subjects:
  - kind: ServiceAccount
    name: admin-user
    namespace: argo
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: argo-workflows-admin

---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: sso-adminuser-data-service-admin
  namespace: data-service
subjects:
  - kind: ServiceAccount
    name: admin-user
    namespace: argo
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: argo-workflows-admin
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: sso-admin-user-devops-admin
  namespace: devops
subjects:
  - kind: ServiceAccount
    name: admin-user
    namespace: argo
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: argo-workflows-admin



```

# 7. Install Argo Events
```
kubectl create namespace argo-events
kubectl apply -f https://raw.githubusercontent.com/argoproj/argo-events/stable/manifests/install.yaml
```

---
# Accessing Argo Workflows
## Certificate
manifest/route/argo-workflows-tls.yaml
```
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: argo-workflows-tls
  namespace: argo
spec:
  dnsNames:
  - argo-workflows.rancher.local
  issuerRef:
    group: cert-manager.io
    kind: ClusterIssuer
    name: selfsigned-cluster-issuer
  secretName: argo-workflows-tls
  usages:
  - digital signature
  - key encipherment

```

## Secret Referent Grant
manifest/route/argo-secret-ref-grant.yaml
```
apiVersion: gateway.networking.k8s.io/v1beta1
kind: ReferenceGrant
metadata:
  name: allow-gw-to-read-secret
  namespace: argo
spec:
  from:
  - group: gateway.networking.k8s.io
    kind: Gateway
    namespace: envoy-gateway-system
  to:
  - group: ""
    kind: Secret
    name: argo-workflows-tls

```

## Expose (HTTPRoute)
manifest/route/argo-workflows-httproute.yaml
```
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: argo-workflows-httproute
  namespace: argo
spec:
  hostnames:
  - argo-workflows.rancher.local
  parentRefs:
  - group: gateway.networking.k8s.io
    kind: Gateway
    name: rke2-gateway
    namespace: envoy-gateway-system
    sectionName: https-argo-workflows
  rules:
  - backendRefs:
    - group: gateway.envoyproxy.io
      kind: Backend
      name: argo-workflows-tls-backend
      weight: 1
    matches:
    - path:
        type: PathPrefix
        value: /

```
## Backend
manifest/route/argo-backend.yaml
```
apiVersion: gateway.envoyproxy.io/v1alpha1
kind: Backend
metadata:
  name: argo-workflows-tls-backend
  namespace: argo
spec:
  endpoints:
  - fqdn:
      hostname: argo-server.argo.svc.cluster.local
      port: 2746
  tls:
    insecureSkipVerify: true
  type: Endpoints

```

## Backend Trafficy Policy
/manifest/route/argo-ws-backend-traffic-policy.yaml
```
apiVersion: gateway.envoyproxy.io/v1alpha1
kind: BackendTrafficPolicy
metadata:
  name: argo-server-ws-backend-policy
  namespace: argo
spec:
  targetRef:
    group: gateway.networking.k8s.io
    kind: HTTPRoute
    name: argo-workflows-httproute

  timeout:
    http:
    # allow long-lived stream used by /api/v1/workflow-events
      requestTimeout: 0s
      connectionIdleTimeout: 0s
    # optional
    # connectTimeout: 10s

```

## Gateway
manifest/route/gateway.yaml
```
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: rke2-gateway
  namespace: envoy-gateway-system
spec:
  gatewayClassName: eg
  listeners:
  - allowedRoutes:
      namespaces:
        from: All
    name: http
    port: 80
    protocol: HTTP
  - allowedRoutes:
      namespaces:
        from: All
    hostname: argo-workflows.rancher.local
    name: https-argo-workflows
    port: 443
    protocol: HTTPS
    tls:
      certificateRefs:
      - group: ""
        kind: Secret
        name: argo-workflows-tls
        namespace: argo
      mode: Terminate


```

## Keycloak Settings 
<img src="images/keycloak-argoworkflows-client-1.png" alt="Keycloak Argo Workflows Client" width="700" />
---
<img src="images/keycloak-argoworkflows-client-2.png" alt="Keycloak Argo Workflows Client" width="700" />
---
<img src="images/keycloak-argoworkflows-client-3.png" alt="Keycloak Argo Workflows Client" width="700" />
