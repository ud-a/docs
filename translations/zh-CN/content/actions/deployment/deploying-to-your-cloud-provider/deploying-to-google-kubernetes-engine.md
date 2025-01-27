---
title: 部署到 Google Kubernetes Engine
intro: 您可以部署到 Google Kubernetes Engine 引擎，作为持续部署 (CD) 工作流程的一部分。
redirect_from:
  - /actions/guides/deploying-to-google-kubernetes-engine
  - /actions/deployment/deploying-to-google-kubernetes-engine
versions:
  fpt: '*'
  ghes: '*'
  ghae: '*'
  ghec: '*'
type: tutorial
topics:
  - CD
  - Containers
  - Google Kubernetes Engine
shortTitle: Deploy to Google Kubernetes Engine
ms.openlocfilehash: 0572a326d52654b256e0e1ad7fe9c9c4e9d547ac
ms.sourcegitcommit: 47bd0e48c7dba1dde49baff60bc1eddc91ab10c5
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/05/2022
ms.locfileid: '147409545'
---
{% data reusables.actions.enterprise-beta %} {% data reusables.actions.enterprise-github-hosted-runners %}

## 简介

本指南介绍如何使用 {% data variables.product.prodname_actions %} 构建容器化应用程序，将其推送到 Google 容器注册表 (GCR)，以及要推送到 `main` 分支时将其部署到 Google Kubernetes Engine (GKE)。

GKE 是 Google Cloud 的托管 Kubernetes 群集服务，可以在云中或您自己的数据中心中托管您的容器化工作负载。 有关详细信息，请参阅 [Google Kubernetes Engine](https://cloud.google.com/kubernetes-engine)。

{% ifversion fpt or ghec or ghae-issue-4856 or ghes > 3.4 %}

{% note %}

注意：{% data reusables.actions.about-oidc-short-overview %}

{% endnote %}

{% endif %}

## 先决条件

在继续创建工作流程之前，您需要完成 Kubernetes 项目的以下步骤。 本指南假定项目的根目录已有 `Dockerfile` 和 Kubernetes 部署配置文件。 有关示例，请参阅 [google-github-actions](https://github.com/google-github-actions/setup-gcloud/tree/master/example-workflows/gke)。

### 创建 GKE 群集

要创建 GKE 群集，首先需要使用 `gcloud` CLI 进行身份验证。 有关此步骤的更多信息，请参阅以下文章：
- [`gcloud auth login`](https://cloud.google.com/sdk/gcloud/reference/auth/login)
- [`gcloud` CLI](https://cloud.google.com/sdk/gcloud/reference)
- [`gcloud` CLI 和 Cloud SDK](https://cloud.google.com/sdk/gcloud#the_gcloud_cli_and_cloud_sdk)

例如：

{% raw %}
```bash{:copy}
$ gcloud container clusters create $GKE_CLUSTER \
    --project=$GKE_PROJECT \
    --zone=$GKE_ZONE
```
{% endraw %}

### 启用 API

启用 Kubernetes Engine 和 Container Registry API。 例如：

{% raw %}
```bash{:copy}
$ gcloud services enable \
    containerregistry.googleapis.com \
    container.googleapis.com
```
{% endraw %}

### 配置服务帐户并存储其凭据

此程序显示如何为您的 GKE 集成创建服务帐户。 它说明了如何创建帐户、向其添加角色、检索其密钥，以及将它们存储为名为 `GKE_SA_KEY` 以 base64 编码的加密存储库机密。

1. 创建新服务帐户：{% raw %}
  ```
  $ gcloud iam service-accounts create $SA_NAME
  ```
  {% endraw %}
1. 检索你刚刚创建的服务帐户的电子邮件地址：{% raw %}
  ```
  $ gcloud iam service-accounts list
  ```
  {% endraw %}
1. 向服务帐户添加角色。 注意：应用限制更严格的角色以满足您的要求。
  {% raw %}
  ```
  $ gcloud projects add-iam-policy-binding $GKE_PROJECT \
    --member=serviceAccount:$SA_EMAIL \
    --role=roles/container.admin
  $ gcloud projects add-iam-policy-binding $GKE_PROJECT \
    --member=serviceAccount:$SA_EMAIL \
    --role=roles/storage.admin
  $ gcloud projects add-iam-policy-binding $GKE_PROJECT \
    --member=serviceAccount:$SA_EMAIL \
    --role=roles/container.clusterViewer
  ```
  {% endraw %}
1. 下载服务帐户的 JSON 密钥文件：{% raw %}
  ```
  $ gcloud iam service-accounts keys create key.json --iam-account=$SA_EMAIL
  ```
  {% endraw %}
1. 将服务帐户密钥存储为名为 `GKE_SA_KEY` 的机密：{% raw %}
  ```
  $ export GKE_SA_KEY=$(cat key.json | base64)
  ```
  {% endraw %} 有关如何存储机密的详细信息，请参阅“[加密机密](/actions/security-guides/encrypted-secrets)”。

### 存储项目名称

将项目的名称存储为名为 `GKE_PROJECT` 的机密。 有关如何存储机密的详细信息，请参阅“[加密机密](/actions/security-guides/encrypted-secrets)”。

### （可选）配置 kustomize
Kustomize 是用于管理 YAML 规范的可选工具。 在创建 `kustomization` 文件之后， 下面的工作流可用于将结果中的图像和管道字段动态设置为 `kubectl`。 有关详细信息，请参阅 [Kustomize 用法](https://github.com/kubernetes-sigs/kustomize#usage)。

### （可选）配置部署环境

{% data reusables.actions.about-environments %}

## 创建工作流程

完成先决条件后，可以继续创建工作流程。

下面的示例工作流程演示如何生成容器映像并推送到 GCR。 然后，它使用 Kubernetes 工具（如 `kubectl` 和 `kustomize`）将映像拉取入群集部署。

在 `env` 密钥下，将值 `GKE_CLUSTER` 更改为群集的名称，将 `GKE_ZONE` 更改为群集区域，将 `DEPLOYMENT_NAME` 更改为部署的名称以及将 `IMAGE` 更改为映像的名称。

{% data reusables.actions.delete-env-key %}

```yaml{:copy}
{% data reusables.actions.actions-not-certified-by-github-comment %}

{% data reusables.actions.actions-use-sha-pinning-comment %}

name: Build and Deploy to GKE

on:
  push:
    branches:
      - main

env:
  PROJECT_ID: {% raw %}${{ secrets.GKE_PROJECT }}{% endraw %}
  GKE_CLUSTER: cluster-1    # Add your cluster name here.
  GKE_ZONE: us-central1-c   # Add your cluster zone here.
  DEPLOYMENT_NAME: gke-test # Add your deployment name here.
  IMAGE: static-site

jobs:
  setup-build-publish-deploy:
    name: Setup, Build, Publish, and Deploy
    runs-on: ubuntu-latest
    environment: production

    steps:
    - name: Checkout
      uses: {% data reusables.actions.action-checkout %}

    # Setup gcloud CLI
    - uses: google-github-actions/setup-gcloud@94337306dda8180d967a56932ceb4ddcf01edae7
      with:
        service_account_key: {% raw %}${{ secrets.GKE_SA_KEY }}{% endraw %}
        project_id: {% raw %}${{ secrets.GKE_PROJECT }}{% endraw %}

    # Configure Docker to use the gcloud command-line tool as a credential
    # helper for authentication
    - run: |-
        gcloud --quiet auth configure-docker

    # Get the GKE credentials so we can deploy to the cluster
    - uses: google-github-actions/get-gke-credentials@fb08709ba27618c31c09e014e1d8364b02e5042e
      with:
        cluster_name: {% raw %}${{ env.GKE_CLUSTER }}{% endraw %}
        location: {% raw %}${{ env.GKE_ZONE }}{% endraw %}
        credentials: {% raw %}${{ secrets.GKE_SA_KEY }}{% endraw %}

    # Build the Docker image
    - name: Build
      run: |-
        docker build \
          --tag "gcr.io/$PROJECT_ID/$IMAGE:$GITHUB_SHA" \
          --build-arg GITHUB_SHA="$GITHUB_SHA" \
          --build-arg GITHUB_REF="$GITHUB_REF" \
          .

    # Push the Docker image to Google Container Registry
    - name: Publish
      run: |-
        docker push "gcr.io/$PROJECT_ID/$IMAGE:$GITHUB_SHA"

    # Set up kustomize
    - name: Set up Kustomize
      run: |-
        curl -sfLo kustomize https://github.com/kubernetes-sigs/kustomize/releases/download/v3.1.0/kustomize_3.1.0_linux_amd64
        chmod u+x ./kustomize

    # Deploy the Docker image to the GKE cluster
    - name: Deploy
      run: |-
        ./kustomize edit set image gcr.io/PROJECT_ID/IMAGE:TAG=gcr.io/$PROJECT_ID/$IMAGE:$GITHUB_SHA
        ./kustomize build . | kubectl apply -f -
        kubectl rollout status deployment/$DEPLOYMENT_NAME
        kubectl get services -o wide
```

## 其他资源

有关这些示例中使用的工具的详细信息，请参阅以下文档：

* 有关完整的初学者工作流，请参阅[“生成并部署到 GKE”工作流](https://github.com/actions/starter-workflows/blob/main/deployments/google.yml)。
* 有关更多初学者工作流和随附的代码，请参阅 Google 的 [{% data variables.product.prodname_actions %} 示例工作流](https://github.com/google-github-actions/setup-gcloud/tree/master/example-workflows/)。
* Kubernetes YAML 自定义引擎：[Kustomize](https://kustomize.io/)。
* Google Kubernetes Engine 文档中的“[部署容器化 Web 应用程序](https://cloud.google.com/kubernetes-engine/docs/tutorials/hello-app)”。
