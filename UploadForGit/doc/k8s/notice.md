hostNetwork: true

curl -X POST http:// -d ""


weave-net 不绑定cluster-admin会报serviceAccount:weave-net cannot list resources
绑定后会报404

 weave net：
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"

mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

应用权限：

source /etc/profile

将master作为node处理：

kubectl taint nodes --all node-role.kubernetes.io/master-

关闭交换区：

sudo swapoff -a

ingress-nginx：
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/static/mandatory.yaml


namespace token:
eyJhbGciOiJSUzI1NiIsImtpZCI6IjVmc0lFNjFvaVBzbVh4U1prRFVQaWlfUlpRX0p6eDU3MDE3RkJTdlAtekkifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJuYW1lc3BhY2UtY29udHJvbGxlci10b2tlbi1tc2ZjYyIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50Lm5hbWUiOiJuYW1lc3BhY2UtY29udHJvbGxlciIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50LnVpZCI6ImI4OWIyZGZmLWZkZjgtNDgyOC04NGY1LTQzOTgwN2I4OWJkZCIsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDprdWJlLXN5c3RlbTpuYW1lc3BhY2UtY29udHJvbGxlciJ9.ED38FvJ4x2h51ljECefoSwhmLP9dBRN0d7BfsARO9l88w7jS-mXBV2wOlrDi6VJACbs7q-vesUBDyhFtAsYuumfCS6uQnbanX-xpgv25RhUxPYyB1hDq_2YR68P5K1wOiAh-ODG7ydfqcaWy7jn5DPD--EU8Ktp18Uac8gfo9SKYGD9XAljaNday4Ehvk4-R3ChxgrbCYX03PoJNyvIEipqjiwfZg09qjt4VSt6YMfdTm3870sEzuCSyi_95XxGF4_6S9vR-3ULhUuJVTI0MVDO5bAm9AkTCCv7NPtTOXYhuj5yMFzuqql5Y5h-6hRMwSuycbxu7ARAAQ19gLEpCQg
