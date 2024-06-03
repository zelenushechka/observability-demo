# observability-demo
Datadog deployment guide for EKS cluster on AWS cloud

Prerequinments:
- Docker
- Helm
- Kubectl

Create trial account at https://www.datadoghq.com/. It can be used for 14 days.

Create cluster, you can leave everything by default. Create node group, you can leave default settings

Login to you cluster: 
aws eks --profile reply_aws --region eu-central-1 update-kubeconfig --name datadog #make sure you are using your own region, profile and cluster name

helm repo add datadog https://helm.datadoghq.com
helm install datadog-operator datadog/datadog-operator
kubectl create secret generic datadog-secret --from-literal api-key=***(use your own api-key)
create file datadog-agent.yaml with content:
apiVersion: datadoghq.com/v2alpha1
kind: DatadogAgent
metadata:
  name: datadog
spec:
  global:
    site: datadoghq.eu #optional if your deployment fails
    credentials:
      apiSecret:
        secretName: datadog-secret
        keyName: api-key

apply: kubectl apply -f datadog-agent.yaml
￼
￼
￼
Let’s create simple pod to load CPU

apiVersion: v1
kind: Pod
metadata:
  name: cpu-load-test
  namespace: default
spec:
  containers:
  - name: cpu-load
    image: busybox
    command:
    - sh
    - -c
    - "while true; do :; done"
  restartPolicy: Never
￼
check metrics in datadog
￼
create namespace
kubectl create namespace external-user

Create rolebinding

apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: external-user-rolebinding
  namespace: external-user
subjects:
- kind: User
  name: external-user
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: external-user-role
  apiGroup: rbac.authorization.k8s.io

Create role, so user can manipulate pods

apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: foobar
  name: external-user-role
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list", "watch", "create", "delete"]

you can test if it works using kubectl auth can-i  command or you switch to user itself	
kubectl auth can-i get pods --namespace=external-user --as=external-user # Should return "yes"
kubectl auth can-i get pods --namespace=default --as=external-user # Should return "no"
kubectl auth can-i get secret --namespace=external-user --as=external-user  # Should return "no" 
