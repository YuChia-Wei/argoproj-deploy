# Argo 相關系統的 kustomization 安裝設定檔

## ArgoCD

參考以下三個連結

* [說明文件](https://loadbalancing.se/2021/03/22/argocd-behind-istio-on-rancher/)
* [官方 issues](https://github.com/argoproj/argo-cd/issues/2784)
* [第三方的主要 GitRepo](https://github.com/epacke/argo-istio)

設定檔中也包含了 `--insecure` 執行參數，以便使用 http 連線
此定義檔主要增加 Kubernetes 的 Label 設定 (app, version)，讓 Istio 可正常存取
為了方便以後的更新，有特別將各種不同類型的 patch 設定檔切割出來

* 安裝檔案說明
    |檔案|說明|
    |--|--|
    |[Kustomiztion.yaml](./ArgoCD/kustomization.yaml)|kustomiztion 定義檔|
    |[install-2.3.4.yaml](./ArgoCD/install-2.3.4.yaml)|argocd 官方 2.3.4 版本安裝檔|
    |[VirtualService.yaml](./ArgoCD/VirtualService.yaml)|Istio 所需的 VirtualService 服務安裝檔|
    |[Gateway.yaml](./ArgoCD/Gateway.yaml)|Istio 所需的 Gateway 服務安裝檔 **(不是 IngressGateway)**|
    |[istio_patches_deployment.yaml](./ArgoCD/istio_patches_deployment.yaml)| argocd 官方版本安裝檔的覆蓋參數，增加 Istio 所需的 Label 定義 |
    |[istio_patches_service.yaml](./ArgoCD/istio_patches_service.yaml)| argocd 官方版本安裝檔的覆蓋參數，增加 Istio 所需的 Label 定義 |
    |[istio_patches_statefulset.yaml](./ArgoCD/istio_patches_statefulset.yaml)| argocd 官方版本安裝檔的覆蓋參數，增加 Istio 所需的 Label 定義 |

### 安裝檔準備

1. download install yaml (option)
    * 指定版本
        ```bash=
         curl -sSL https://raw.githubusercontent.com/argoproj/argo-cd/v2.3.4/manifests/install.yaml -o install-2.3.4.yaml
        ```
    * 最新版本
        ```bash=
         curl -sSL https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml -o install-latest.yaml
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

### When Upgrade

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

### Install

```
kubectl apply -k ./
```
