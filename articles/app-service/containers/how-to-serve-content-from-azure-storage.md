---
title: Linux 上にカスタムストレージコンテナーをアタッチする
description: Azure App Service で Linux コンテナーにカスタムネットワーク共有をアタッチする方法を学びます。 アプリ間でのファイルの共有、静的コンテンツのリモート管理、ローカルでのアクセスなどを行います。
author: msangapu-msft
ms.topic: article
ms.date: 2/04/2019
ms.author: msangapu
ms.openlocfilehash: 00c60edeefa5fd8d1304aa5fc301a3b0304f5ca3
ms.sourcegitcommit: 265f1d6f3f4703daa8d0fc8a85cbd8acf0a17d30
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/02/2019
ms.locfileid: "74671786"
---
# <a name="attach-azure-storage-containers-to-linux-containers"></a>Azure Storage コンテナーを Linux コンテナーにアタッチする

このガイドでは、[Azure Storage](/azure/storage/common/storage-introduction)を使用して、Linux 上の App Service に、ネットワーク共有をアタッチする方法について説明します。 利点としては、セキュリティで保護されたコンテンツ、コンテンツの移植性、永続ストレージ、複数のアプリへのアクセス、複数の転送方法などがあります。

## <a name="prerequisites"></a>前提条件

- 既存の Web アプリ (Linux 上の App Service または Web App for Containers)。
- [Azure CLI](/cli/azure/install-azure-cli) (2.0.46 以降)。

## <a name="create-azure-storage"></a>Azure Storage を作成する

> [!NOTE]
> Azure Storage は、既定ではないストレージであり、別途請求され、Web アプリには含まれません。
>
> Bring Your Own Storage は、インフラストラクチャの制限により、Storage Firewall 構成の使用をサポートしていません。
>

[Azure ストレージ アカウント](https://docs.microsoft.com/azure/storage/common/storage-quickstart-create-account?tabs=azure-cli)を作成します。

```azurecli
#Create Storage Account
az storage account create --name <storage_account_name> --resource-group myResourceGroup

#Create Storage Container
az storage container create --name <storage_container_name> --account-name <storage_account_name>
```

## <a name="upload-files-to-azure-storage"></a>Azure Storage にファイルをアップロードする

ローカル ディレクトリをストレージ アカウントにアップロードするには、次の例のような [`az storage blob upload-batch`](https://docs.microsoft.com/cli/azure/storage/blob?view=azure-cli-latest#az-storage-blob-upload-batch) コマンドを使用します。

```azurecli
az storage blob upload-batch -d <full_path_to_local_directory> --account-name <storage_account_name> --account-key "<access_key>" -s <source_location_name>
```

## <a name="link-storage-to-your-web-app-preview"></a>ストレージを Web アプリにリンクする (プレビュー)

> [!CAUTION]
> Web アプリ内の既存のディレクトリをストレージ アカウントにリンクすると、ディレクトリの内容が削除されます。 既存アプリのファイルを移行する場合は、始める前に、アプリとその内容をバックアップしてください。
>

ストレージ アカウントを App Service アプリのディレクトリにマウントするには、[`az webapp config storage-account add`](https://docs.microsoft.com/cli/azure/webapp/config/storage-account?view=azure-cli-latest#az-webapp-config-storage-account-add) コマンドを使用します。 ストレージの種類としては、AzureBlob または AzureFiles を使用できます。 このコンテナーには AzureBlob を使用します。

```azurecli
az webapp config storage-account add --resource-group <group_name> --name <app_name> --custom-id <custom_id> --storage-type AzureBlob --share-name <share_name> --account-name <storage_account_name> --access-key "<access_key>" --mount-path <mount_path_directory>
```

ストレージ アカウントにリンクする他のすべてのディレクトリについて、これを行う必要があります。

## <a name="verify"></a>確認

ストレージ コンテナーを Web アプリにリンクしたら、次のコマンドを実行してこれを確認できます。

```azurecli
az webapp config storage-account list --resource-group <resource_group> --name <app_name>
```

## <a name="use-custom-storage-in-docker-compose"></a>Docker Compose でカスタム ストレージを使用する

Azure Storage は、custom-id を使用してマルチコンテナー アプリを使用してマウントできます。custom-id 名を表示するには、[`az webapp config storage-account list --name <app_name> --resource-group <resource_group>`](/cli/azure/webapp/config/storage-account?view=azure-cli-latest#az-webapp-config-storage-account-list) を実行します。

*docker-compose.yml* ファイルで、`volumes` オプションを `custom-id` にマップします。 例:

```yaml
wordpress:
  image: wordpress:latest
  volumes:
  - <custom-id>:<path_in_container>
```

## <a name="next-steps"></a>次の手順

- [Azure App Service での Web アプリの構成](../configure-common.md)。
