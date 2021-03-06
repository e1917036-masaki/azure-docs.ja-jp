---
title: セキュリティで保護された Azure マネージド ワークステーションを理解する - Azure Active Directory
description: セキュリティで保護された Azure マネージド ワークステーションについて説明し、これらが重要な理由を理解します。
services: active-directory
ms.service: active-directory
ms.subservice: devices
ms.topic: conceptual
ms.date: 11/18/2019
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: frasim
ms.collection: M365-identity-device-management
ms.openlocfilehash: c26197a14e78b1cf1a1e078ba0145eca207206bf
ms.sourcegitcommit: c31dbf646682c0f9d731f8df8cfd43d36a041f85
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/27/2019
ms.locfileid: "74561976"
---
# <a name="understand-secure-azure-managed-workstations"></a>セキュリティで保護された Azure マネージド ワークステーションを理解する

セキュリティで保護された分離したワークステーションは、管理者、開発者、重要なサービス オペレーターのような機密性の高い役割のセキュリティには非常に重要です。 クライアント ワークステーションのセキュリティが侵害されると、多くのセキュリティ制御とセキュリティ保証が失敗するか、効果がなくなる可能性があります。

このドキュメントでは、特権アクセス ワークステーション (PAW) として広く知られている、セュリティで保護されたワークステーションを構築するために必要なものについて説明します。 この記事には、最初のセキュリティ制御を設定するための詳細な手順も含まれています。 このガイダンスでは、クラウド ベースのテクノロジがどのようにサービスを管理することができるかについて説明します。 これは、Windows 10RS5、Microsoft Defender Advanced Threat Protection (ATP)、Azure Active Directory、および Microsoft Intune で導入されたセキュリティ機能に依存します。

> [!NOTE]
> この記事では、セキュリティで保護されたワークステーションの概念とその重要性について説明します。 概念について既によくご存知で、デプロイにスキップする場合は、「[セキュリティで保護されたワークステーションをデプロイする](howto-azure-managed-workstation.md)」をご覧ください。

## <a name="why-secure-workstation-access-is-important"></a>セキュリティで保護されたワークステーションのアクセスが重要な理由

クラウド サービスの急速な導入と、どこからでも作業できるようになったことで、新たな悪用方法が生み出されました。 攻撃者は、管理者が作業するデバイス上の脆弱なセキュリティ制御を悪用することで、特権付きリソースへのアクセス権を取得します。

特権の悪用とサプライ チェーン攻撃は、攻撃者が組織を侵害するために使用する上位 5 つの方法に入っています。 [Verizon 脅威レポート](https://enterprise.verizon.com/resources/reports/dbir/)、および[セキュリティ インテリジェンス レポート](https://aka.ms/sir)によると、これらは 2018 年に報告されたインシデントで 2 番目に多く検出された戦術です。

ほとんどの攻撃者は次の手順を踏みます。

1. 入口を見つけるために、多くの場合は業界を特定して、偵察する。
1. 収集した情報を分析して、価値が低いと思われているワークステーションに侵入する最善の方法を特定する。
1. とどまって、[水平](https://en.wikipedia.org/wiki/Network_Lateral_Movement)移動する手段を見つける。
1. 機密データを抜き取る。

偵察中に、攻撃者は、リスクが低いと思われるデバイスや過小評価されていると思われるデバイスに頻繁に侵入します。 これらの脆弱性のあるデバイスを使用して、水平移動の機会を見つけ、管理ユーザーとデバイスを探します。 攻撃者が特権ユーザー ロールへのアクセスを得ると、価値の高いデータを特定し、そのデータの抜き取りに成功します。

![一般的なセキュリティ侵害のパターン](./media/concept-azure-managed-workstation/typical-timeline.png)

このドキュメントでは、このような水平移動攻撃からコンピューティング デバイスを保護するのに役立つソリューションについて説明します。 このソリューションは、機密性の高いクラウド リソースがあるデバイスに侵入される前にチェーンを断ち切ることで、管理とサービスを価値の低い生産性向上デバイスから分離します。 このソリューションでは、次のような、Microsoft 365 Enterprise スタックの一部であるネイティブの Azure サービスが使用されます。

* デバイス管理用の Intune (アプリケーションと URL のセーフ リストを含む)
* Autopilot (デバイスの設定、デプロイ、更新用)
* Azure AD (ユーザー管理、条件付きアクセス、および多要素認証用)
* Windows 10 (現在のバージョン) (デバイス正常性構成証明書とユーザー エクスペリエンス用)
* Defender ATP (クラウド管理によるエンドポイントの保護、検出、および応答用)
* Azure AD PIM (リソースへのジャストインタイム (JIT) の特権アクセスなどの認証の管理用)
* Log Analytics、および Sentinel (監視とアラート用)

## <a name="who-benefits-from-a-secure-workstation"></a>セキュリティで保護されたワークステーションからメリットが得られる人

すべてのユーザー、およびオペレーターは、セキュリティで保護されたワークステーションを使用するとメリットが得られます。 PC やデバイスを侵害する攻撃者は、キャッシュされたすべてのアカウントを偽装する可能性があります。 デバイスにログインするときに、資格情報やトークンを使用する可能性もあります。 このリスクにより、管理者権限を含む特権ロールで使用されるデバイスをセキュリティ保護することが非常に重要になります。 特権アカウントを持つデバイスは、水平移動攻撃や特権エスカレーション攻撃の標的になります。 これらのアカウントは、次のようなさまざまな資産に使用することができます。

* オンプレミスまたはクラウドベースのシステムの管理者
* 重要なシステムの開発者のワークステーション
* 露出度の高いソーシャル メディア アカウントの管理者
* SWIFT 支払い端末などの機密性の高いワークステーション
* 企業機密を扱うワークステーション

リスクを軽減するには、これらのアカウントが使用されている特権ワークステーションに対して、管理者特権でのセキュリティ制御を実装する必要があります。 詳細については、「[Azure Active Directory 機能のデプロイ ガイド](../fundamentals/active-directory-deployment-checklist-p2.md)」、[Office 365 ロードマップ](https://aka.ms/o365secroadmap)、および[特権アクセスのセキュリティ保護のロードマップ](https://aka.ms/sparoadmap)に関するページを参照してください。

## <a name="why-use-dedicated-workstations"></a>専用のワークステーションを使用する理由

既存のデバイスにセキュリティを追加することはできますが、安全な基盤から始めることをお勧めします。 高度なセキュリティ レベルを維持するために組織が有利な立場に立てるようにするには、セキュリティで保護された既知のデバイスから始めて、一連の既知のセキュリティ制御を実装します。

メールや Web の閲覧によって攻撃ベクトルの数が増え続けていることで、デバイスを信頼できると保証することがますます困難になっています。 このガイドでは、専用のワークステーションが標準的な生産性、閲覧、およびメールから分離されていることを前提としています。 デバイスから生産性、Web の閲覧、およびメールを取り除くと、生産性に悪影響を及ぼす可能性があります。 ただし、この保護は通常、ジョブ タスクで明示的に要求されず、セキュリティ インシデントのリスクが高いシナリオには適しています。

> [!NOTE]
> ここでの Web の閲覧は、リスクの高いアクティビティになる可能性がある任意の Web サイトへの一般的なアクセスを指しています。 このような閲覧は、Web ブラウザーを使用してサービス (Azure、Office 365、その他のクラウド プロバイダー、SaaS アプリケーションなど) のために少数の既知の正常な管理用 Web サイトにアクセスすることとは明らかに違います。

コンテインメント戦略では、攻撃者が機密性の高い資産へのアクセスを得ることを阻止するコントロールの数と種類を増やすことで、セキュリティを強化します。 この記事で説明されているモデルは、階層化された権限設計を使用して、特定のデバイスへの管理特権を制限します。

## <a name="supply-chain-management"></a>サプライ チェーン管理

セキュリティで保護されたワークステーションに不可欠なのは、"信頼のルート" と呼ばれる信頼されたワークステーションを使用するサプライ チェーン ソリューションです。 信頼のルートのハードウェアの選択で検討する必要があるテクノロジとして、最新のラップトップに搭載されている次のテクノロジが含まれます。 

* [トラステッド プラットフォーム モジュール (TPM) 2.0](https://docs.microsoft.com/windows-hardware/design/device-experiences/oem-tpm)
* [BitLocker ドライブ暗号化](https://docs.microsoft.com/windows-hardware/design/device-experiences/oem-bitlocker)
* [UEFI セキュア ブート](https://docs.microsoft.com/windows-hardware/design/device-experiences/oem-secure-boot)
* [Windows Update 経由で配布されたドライバーとファームウェア](https://docs.microsoft.com/windows-hardware/drivers/dashboard/understanding-windows-update-automatic-and-optional-rules-for-driver-distribution)
* [仮想化および HVCI 対応](https://docs.microsoft.com/windows-hardware/design/device-experiences/oem-vbs)
* [ドライバーとアプリの HVCI 対応](https://docs.microsoft.com/windows-hardware/test/hlk/testref/driver-compatibility-with-device-guard)
* [Windows Hello](https://docs.microsoft.com/windows-hardware/design/device-experiences/windows-hello-biometric-requirements)
* [DMA I/O 保護](https://docs.microsoft.com/windows/security/information-protection/kernel-dma-protection-for-thunderbolt)
* [System Guard](https://docs.microsoft.com/windows/security/threat-protection/windows-defender-system-guard/system-guard-how-hardware-based-root-of-trust-helps-protect-windows)
* [モダン スタンバイ](https://docs.microsoft.com/windows-hardware/design/device-experiences/modern-standby)

このソリューションでは、[Microsoft Autopilot](https://docs.microsoft.com/windows/deployment/windows-autopilot/windows-autopilot) テクノロジと最新の技術要件を満たすハードウェアを使用して、信頼のルートをデプロイします。 ワークステーションをセキュリティで保護するため、Autopilot では Microsoft OEM 用に最適化された Windows 10 デバイスが利用できます。 これらのデバイスは、製造元から既知の正常な状態で提供されます。 Autopilot では、セキュリティで保護されていない可能性があるデバイスを再イメージ化する代わりに、Windows デバイスを “ビジネスに即応する” 状態に変換できます。 設定とポリシーを適用し、アプリをインストールし、Windows 10 のエディションも変更します。 たとえば、Autopilot では、高度な機能が使用できるように、デバイスの Windows インストールを Windows 10 Pro から Windows 10 Enterprise に変更することができます。

![セキュリティで保護されたワークステーションのレベル](./media/concept-azure-managed-workstation/supplychain.png)

## <a name="device-roles-and-profiles"></a>デバイスのロールとプロファイル

このガイダンスでは、ユーザー、開発者、および IT スタッフにとってより安全性の高いソリューションを作成するのに役立ついくつかのセキュリティ プロファイルとロールについて取り上げています。 これらのプロファイルは、強化された、またはセキュリティで保護されたワークステーションからメリットが得られる一般的なユーザーのために、使いやすさとリスクのバランスを取ります。 ここで提供されている設定の構成は、業界で広く認められている標準に基づいています。 このガイダンスでは、Windows 10 を強化し、デバイスまたはユーザーの侵害に関連するリスクを軽減する方法を示します。 最新のハードウェア テクノロジと信頼のルート デバイスを活用するには、[デバイス正常性構成証明](https://techcommunity.microsoft.com/t5/Intune-Customer-Success/Support-Tip-Using-Device-Health-Attestation-Settings-as-Part-of/ba-p/282643)を使用します。これは、**高セキュリティ** プロファイルで開始すると有効になります。 この機能は、デバイスの初期ブート中に攻撃者が永続性を維持できないようにするために用意されています。 セキュリティ機能とリスクの管理に役立つポリシーとテクノロジを使用します。
![セキュリティで保護されたワークステーションのレベル](./media/concept-azure-managed-workstation/seccon-levels.png)

* **基本的なセキュリティ** – 標準的なマネージド ワークステーションは、ほとんどの自宅や小規模企業で手始めとして使用するのに適しています。 これらのデバイスは Azure AD に登録され、Intune で管理されます。 このプロファイルは、ユーザーにアプリケーションの実行と Web サイトの閲覧を許可します。 [Microsoft Defender](https://www.microsoft.com/windows/comprehensive-security) などのマルウェア対策ソリューションを有効にする必要があります。

* **強化されたセキュリティ** – このエントリ レベルの保護されたソリューションは、ホーム ユーザー、小規模企業のユーザー、および一般的な開発者に適しています。

   強化されたワークステーションは、低セキュリティのプロファイルのセキュリティを強化するポリシー ベースの方法です。 これにより、メールや Web の閲覧などの生産性向上ツールも使用しつつ、顧客データを使用する安全な手段が提供されます。 監査ポリシーと Intune を使用して、ユーザーの動作やプロファイルの使用について強化されたワークステーションを監視することができます。 Windows 10 (1809) スクリプトを使用して、強化されたワークステーションのプロファイルをデプロイします。これは [Advanced Threat Protection (ATP)](https://docs.microsoft.com/windows/security/threat-protection/microsoft-defender-atp/microsoft-defender-advanced-threat-protection) を使用した高度なマルウェア保護を活用します。

* **高セキュリティ** – ワークステーションの攻撃対象領域を減らすための最も効果的な手段は、ワークステーションの自己管理機能を削除することです。 ローカル管理者権限を削除することは、セキュリティを向上する 1 つの手段ですが、正しく実装されていない場合は生産性に影響を与える可能性があります。 高セキュリティのプロファイルは、強化されたセキュリティのプロファイルを基盤として、ローカル管理の削除という 1 つの重要な変更を加えています。このプロファイルは、ハイ プロファイル ユーザー (役員、給与支払いと機密データ ユーザー、サービスとプロセスの承認者) 向けに設計されています。

   高セキュリティのユーザーには、より制御された環境でありながら、メールや Web の閲覧などのアクティビティを引き続き使い勝手良く実行できる必要があります。 ユーザーは、Cookie、お気に入り、作業のためのその他のショートカットなどの機能を期待しています。 ただし、このようなユーザーは変更を加えたりデバイスをデバッグしたりする必要がない場合があります。 また、ドライバーのインストールも必要としていません。 高セキュリティのプロファイルは、High Security - Windows10 (1809) スクリプトを使用してデプロイされます。

* **特殊** – 攻撃者は、開発者と IT 管理者を標的にします。彼らは、攻撃者にとって関心のあるシステムを変更できるからです。 特殊ワークステーションは、ローカル アプリケーションの管理や Web サイトの制限により、高セキュリティのワークステーションのポリシーを拡張します。 また、ActiveX、Java、ブラウザー プラグイン、およびその他の Windows コントロールなどのリスクの高い生産性機能も制限します。 このプロファイルは、DeviceConfiguration_NCSC - Windows10 (1803) SecurityBaseline スクリプトを使用してデプロイします。

* **セキュリティ保護** – 管理者アカウントを侵害する攻撃者は、データ盗難、データ改ざん、またはサービスの中断によってビジネスに重大な損害を与える可能性があります。 この強化された状態では、ワークステーションによってローカル アプリケーション管理の直接的な制御を制限するすべてのセキュリティ制御とポリシーが有効になります。 セキュリティで保護されたワークステーションには生産性向上ツールがないため、デバイスの侵害がより困難になります。 フィッシング攻撃の最も一般的なベクトルであるメールとソーシャル メディアがブロックされます。 セキュリティ保護されたワークステーションは、Secure Workstation - Windows 10 (1809) SecurityBaseline スクリプトを使用してデプロイできます。

   ![セキュリティ保護されたワークステーション](./media/concept-azure-managed-workstation/secure-workstation.png)

   セキュリティ保護されたワークステーションにより、管理者に、明確なアプリケーション制御とアプリケーション保護を備えた、強化されたワークステーションが提供されます。 このワークステーションは、資格情報の保護、デバイスの保護、および悪用からの保護を使用して、悪質な行動からホストを保護します。 また、すべてのローカル ディスクが BitLocker で暗号化されます。

* **分離** – このカスタムのオフライン シナリオは、最も極端なものを表しています。 このケースのインストール スクリプトは提供されていません。 サポートされていない、またはパッチが適用されていないレガシ オペレーティング システムを必要とする、ビジネスに不可欠な機能を管理する必要がある場合があります。 たとえば、高価値の生産ラインや生命維持装置などです。 セキュリティは非常に重要であり、クラウド サービスは利用できないため、これらのコンピューターを手動で、または Enhanced Security Admin Environment (ESAE) などの分離された Active Directory フォレスト アーキテクチャを使用して管理および更新できます。 このような状況では、基本の Intune 以外のすべてのアクセス権の削除と ATP 正常性チェックを考慮してください。

   * [Intune ネットワーク通信の要件](https://docs.microsoft.com/intune/network-bandwidth-use)
   * [ATP ネットワーク通信の要件](https://docs.microsoft.com/azure-advanced-threat-protection/configure-proxy)

## <a name="next-steps"></a>次の手順

[セキュリティで保護された Azure マネージド ワークステーションをデプロイする](howto-azure-managed-workstation.md)。
