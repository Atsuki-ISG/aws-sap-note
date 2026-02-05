---
id: "SAP-63"
title: "IAM Identity Center with On-Prem AD"
related_note: "../../notes/Security/SAP-63-IAM-Identity-Center-AD-Federation.md"
---

# Problem Statement

【SAP-63】 国内で EC サイトと実店舗 POS を運営する小売企業が、単一の AWS アカウント上で基幹システムを稼働させています。
IT サポート部門は AWS マネジメントコンソールから日次のインスタンス状態確認やタグ付け作業を行っており、現在は職務別に作成した IAM ユーザーでログインしています。
だが「社内 Active Directory だけを ID 管理の単一源泉とし、運用負荷と監査コストを下げたい」という経営要請が出たため、重複する IAM ユーザーを廃止し AD の資格情報でコンソールにシームレス接続できる仕組みを導入する方針となりました。
スケールアウトや多アカウント展開は当面予定がなく、初期費用と月次コストも最小に抑える前提です。
ソリューションアーキテクトは AWS IAM Identity Center （旧 AWS SSO） を採用することを決め、既存オンプレミス AD と連携させて認可をロール単位で制御する構成を検討しています。
運用は少人数体制のため、マネージドサービスを用いてパッチ適用やレプリケーションの管理を極力 AWS 側に任せる必要があります。
これらの要件と制約を踏まえ、最もコスト効率が高いアプローチを選択できるのはどの構成か？

# Options

A. AWS Organizations で組織を作成し、管理アカウントで IAM Identity Center を有効化する。オンプレミスの Active Directory と連携するために、オンプレミス側に Active Directory フェデレーションサービス（AD FS）などの SAML 対応 IdP を構成し、IAM Identity Center の ID ソースとしてこの外部 IdP を指定する。オンプレミス Active Directory の既存グループを SAML クレームで渡し、それらに対応するアクセス許可セットを IAM Identity Center 上に作成して、AWS マネジメントコンソールへのシングルサインオンアクセスを付与する。
B. AWS Organizations で組織を作成し、AWS Directory Service for Microsoft Active Directory (AWS Managed Microsoft AD) を構築してオンプレミス Active Directory と双方向信頼を設定する。IAM Identity Center の ID ソースに AWS Managed Microsoft AD を指定し、そこに作成したグループへアクセス許可セットを割り当てる。オンプレミスのグループは信頼を通じて利用する。
C. AWS Organizations で組織を作成し、すべての機能を有効化する。AWS Directory Service for Microsoft Active Directory (AWS Managed Microsoft AD) を構築してオンプレミス Active Directory と双方向信頼を設定し、IAM Identity Center の ID ソースとしてこの AWS Managed Microsoft AD ディレクトリを選択する。アクセス許可セットは主に AWS Managed Microsoft AD 側のグループにマッピングし、必要に応じてオンプレミス側のグループも補助的に利用する。
D. AWS Organizations で組織を作成し、管理アカウントで IAM Identity Center を有効化する。AWS Managed Microsoft AD の新規ディレクトリを作成するが、オンプレミス Active Directory とは信頼関係を結ばず、IAM Identity Center の ID ソースとしてこのディレクトリだけを使用する。AWS Managed Microsoft AD 内のユーザーとグループにアクセス許可セットを割り当てる。

# Answer
A
