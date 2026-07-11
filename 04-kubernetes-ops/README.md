# 04. Kubernetes 運用入門（壊して直す）

**わざと壊して、Events / describe / logs で直す。** 無料の Minikube だけで完結します。

## 前提

- [03-kubernetes](../03-kubernetes/) 完了
- `minikube start` 済み、`kubectl get nodes` で Ready

## 学習順

1. [01-troubleshooting](./01-troubleshooting/)
2. [02-probes-resources](./02-probes-resources/)
3. [03-storage-rbac](./03-storage-rbac/)

マニフェスト: [manifests/](./manifests/)

## 切り分けの基本手順（暗記）

```text
1. kubectl get pods
2. kubectl describe pod <name>   … Events
3. kubectl logs <name>
4. kubectl get events --sort-by='.lastTimestamp'
5. Service / ラベル / ポート
```

## NetworkPolicy（概要のみ・実機必須ではない）

**NetworkPolicy = Pod 間のファイアウォール。**  
Minikube は CNI 差があるため、この章では概念理解で DoD 達成可。

## 完了条件（DoD）

- [ ] CrashLoop / ImagePullBackOff / Service 未接続を区別して直せる
- [ ] Probe と requests/limits の目的を説明できる
- [ ] PVC と RBAC（can-i）を一度は触った

次: [05-openshift](../05-openshift/)
