# Argo Project install kustomization

**argocd 商標圖示來自於 [官方 github](https://github.com/argoproj/argo-cd/blob/master/docs/assets/argo.png)**

:::warning
由於我這只是我測試 k8s / argocd 與相關服務使用的一個設定，因此在更新版本時都是不考慮 argocd 版本更換時的各種 api 遷移問題，我通常都是直接重新安裝 argocd。如果有遷移需求的話，這一份 repo 僅供參考。
:::

## repo 說明

本 repo 可用於快速安裝 argocd 至 k8s 中，並且進行自我管理，以便未來的版本更新。

這份 repo 中所提供的 kustomization patch 資料室為了替 argocd 的所有資源加上以下兩個 metadata
- app
- version

這兩個 metadata 主要是提供給早期版本的 istio 中所附帶的 kiali, Prometheus 等服務繪製網路流量圖使用，後續是否需要使用這兩個資訊用於流量管理則是要看組織是否有相關需求，這邊我依循我自己早期建立的規則，會持續保留相關標籤的建立。

## CICD Pipeline

![](./CI-CD%20Pipeline.jpg)

## Cluster Requested

- istio

## package version

| package name  | version | update date |
|---------------|---------|-------------|
| argocd        | 3.0.12  | 2025-08-01  |
| argo-rollouts | 1.8.3   | 2025-08-01  |

## ArgoCD Patch For Istio

參考以下三個連結

* [說明文件](https://loadbalancing.se/2021/03/22/argocd-behind-istio-on-rancher/)
* [官方 issues](https://github.com/argoproj/argo-cd/issues/2784)
* [第三方的主要 GitRepo](https://github.com/epacke/argo-istio)

此定義檔主要增加 Kubernetes 的 Label 設定 (app, version)，讓 Istio 可正常存取
為了方便以後的更新，有特別將各種不同類型的 patch 設定檔切割出來

### 官方說明

目前 (2025/08/01 - v3.0.12) 版本可以看到官方說明文件上已有 istio 相關的 ingress 設定說明

[argocd - ingress configuration](https://argo-cd.readthedocs.io/en/stable/operator-manual/ingress/#istio)

## 安裝檔更新

1. download install yaml (option)
    * 指定版本
        ```bash=
        curl -sSLk https://raw.githubusercontent.com/argoproj/argo-cd/v3.0.12/manifests/install.yaml -o install.yaml
        curl -sSLk https://raw.githubusercontent.com/argoproj/argo-cd/v3.0.12/manifests/ha/install.yaml -o install-ha.yaml
        curl -sSLk https://github.com/argoproj/argo-rollouts/releases/download/v1.8.3/install.yaml -o install.yaml
        curl -sSLk https://github.com/argoproj/argo-rollouts/releases/download/v1.8.3/dashboard-install.yaml -o dashboard-install.yaml
        ```
1. 建立 namespace
    * 純建立
        ```bash=
        kubectl create namespace argocd
        ```
    * 含設定 istio 掛車
        ```bash=
        kubectl create namespace argocd
        kubectl label namespace argocd istio-injection=enabled --overwrite

        # 如果安裝 istio 的時候是特別設定版本的話 (通常出現在有使用金絲雀部署來更新 istio 的狀況)
        # 如果已經有設定掛車的話，可以用這方式刪掉舊的同時設定新的
        #kubectl label namespace argocd istio-injection- istio.io/rev=1-13-3
        kubectl label namespace argocd istio.io/rev=1-13-3
        ```

## When Upgrade

* 升級後不會更新設定檔，因此原始的密碼與設定都相同
* 可以先用以下命令檢查差異 (cmd 位置要先到 argocd kustomiztion 的位置)
    ```bash=
    # 查看差異
    kubectl diff -k ./

    # 將差異資料輸出到特定檔案
    kubectl diff -k ./ > different-data.txt

    # 測試執行並輸出結果
    kubectl apply -k ./ --dry-run=client > dry-run.txt

    ```

## Install

```
kubectl apply -k ./
```

## link to prometheus

從 Argo-Rollout 的經驗來看，應該要設定在 deployment 上面，但是不知道什麼原因，測試時使用了設定到 service 的方法才成功

[設定到 deployment 設定的方法](https://newrelic.com/instant-observability/argocd-quickstart/03b4a3b9-3a59-4603-91dd-6b0ced1d62de)
[設定到 service 設定的方法](https://www.gushiciku.cn/pl/g7Ok/zh-tw)
