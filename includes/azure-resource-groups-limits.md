---
author: tfitzmac
ms.service: azure-resource-manager
ms.topic: include
ms.date: 08/19/2019
ms.author: tomfitz
ms.openlocfilehash: b4338560d515289d3e9aedfa716ec7c3ace51131
ms.sourcegitcommit: f4f626d6e92174086c530ed9bf3ccbe058639081
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/25/2019
ms.locfileid: "75392422"
---
| リソース | 既定の制限 | 上限 |
| --- | --- | --- |
| リソースの種類ごとの、[リソース グループ](../articles/azure-resource-manager/management/overview.md#resource-groups)あたりのリソース数 |800 |リソースの種類によっては、800 の制限を超えることがあります。 「[リソース グループあたり 800 インスタンスに制限されないリソース](../articles/azure-resource-manager/management/resources-without-resource-group-limit.md)」を参照してください。 |
| デプロイ履歴でのリソース グループあたりのデプロイ数 |800<sup>1</sup> |800 |
| デプロイあたりのリソース数 |800 |800 |
| 一意のスコープあたりの管理ロック数 |20 |20 |
| リソースまたはリソース グループあたりのタグ数 |50 |50 |
| タグ キーの長さ |512 |512 |
| タグ値の長さ |256 |256 |

<sup>1</sup> リソース グループあたり 800 のデプロイという上限に達した場合、不要になった履歴からデプロイを削除します。 デプロイ履歴からエントリを削除しても、デプロイ リソースには影響しません。 詳細については、「[デプロイ数が 800 を超えたときのエラーを解決する](../articles/azure-resource-manager/templates/deployment-quota-exceeded.md)」を参照してください。

#### <a name="template-limits"></a>テンプレートの制限

| 値 | 既定の制限 | 上限 |
| --- | --- | --- |
| パラメーター |256 |256 |
| 変数: |256 |256 |
| リソース (コピー数を含む) |800 |800 |
| 出力 |64 |64 |
| テンプレート式 |24,576 文字 |24,576 文字 |
| エクスポートされたテンプレート内のリソース |200 |200 | 
| テンプレート サイズ |4 MB |4 MB |
| パラメーター ファイル サイズ |64 KB |64 KB |

入れ子になったテンプレートを使用すると、一部のテンプレートの制限を超過することができます。 詳細については、[Azure リソース デプロイ時のリンクされたテンプレートの使用](../articles/azure-resource-manager/templates/linked-templates.md)に関するページを参照してください。 パラメーター、変数、出力の数を減らすために、いくつかの値を 1 つのオブジェクトに結合することができます。 詳しくは、[パラメーターとしてのオブジェクト](../articles/azure-resource-manager/resource-manager-objects-as-parameters.md)に関する記事をご覧ください。
