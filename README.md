# flux-flagger-demo

## 構築手順

### Cluster

#### kind

```bash
$ kind create cluster --name flux-flagger-demo --config setup/kind.yaml
```

#### EKS

```bash
$ eksctl create cluster -f eksctl_config.yaml
```

### Helm

```bash
$ kubectl apply -f setup/helm-rbac.yaml
$ helm init --service-account tiller
```

### Istio

```bash
$ helm install -n istio-init charts/istio-init --namespace=istio-system
$ helm install -n istio charts/istio --namespace=istio-system -f setup/values/istio.yaml
```

### Flagger

```bash
$ helm upgrade -i flagger flagger/flagger --namespace=istio-system -f setup/values/flagger.yaml
```

### Flux

```bash
$ helm repo add fluxcd https://fluxcd.github.io/flux
$ helm upgrade -i flux --namespace flux fluxcd/flux -f setup/values/flux.yaml
# 下記コマンドのpublic key をリポジトリのDeploy keysに登録する(Allow write accessにチェック)
$ fluxctl identity --k8s-fwd-ns flux
```

## 動作確認手順

1. デプロイ済みのサービスへアクセスする
1. イメージのバージョンを変更し、コミット/プッシュする
1. 1を実行し、一部のアクセスのみ最新バージョンへアクセスしていることを確認する
1. kialiで確認する

### kiali

```bash
$ kubectl apply -f setup/kiali-secret.yaml
$ kubectl -n istio-system port-forward $(kubectl -n istio-system get pod -l app=kiali -o jsonpath='{.items[0].metadata.name}') 20001:20001
```
