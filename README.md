# Argo Project install kustomization

⚠️️⚠️This `readme` file is too old! It will be updated in the future.⚠️️⚠️

## Cluster Requested

- istio

## package version

| package name  | version | update date |
|---------------|---------|-------------|
| argocd        | 2.7.1   | 2023-05-06  |
| argo-rollouts | 1.4.1   | 2023-05-06  |

## ArgoCD

參考以下三個連結

* [說明文件](https://loadbalancing.se/2021/03/22/argocd-behind-istio-on-rancher/)
* [官方 issues](https://github.com/argoproj/argo-cd/issues/2784)
* [第三方的主要 GitRepo](https://github.com/epacke/argo-istio)

設定檔中也包含了 `--insecure` 執行參數，以便使用 http 連線
此定義檔主要增加 Kubernetes 的 Label 設定 (app, version)，讓 Istio 可正常存取
為了方便以後的更新，有特別將各種不同類型的 patch 設定檔切割出來

## 安裝檔更新

1. download install yaml (option)
    * 指定版本
        ```bash=
        curl -sSL https://raw.githubusercontent.com/argoproj/argo-cd/v2.6.1/manifests/install.yaml -o install.yaml
        curl -sSL https://raw.githubusercontent.com/argoproj/argo-cd/v2.6.1/manifests/ha/install.yaml -o install-ha.yaml
        curl -sSL https://github.com/argoproj/argo-rollouts/releases/download/v1.4.0/install.yaml -o install.yaml
        curl -sSL https://github.com/argoproj/argo-rollouts/releases/download/v1.4.0/dashboard-install.yaml -o dashboard-install.yaml
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
