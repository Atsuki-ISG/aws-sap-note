---
id: "SAP-63"
title: "IAM Identity Center with On-Prem AD"
category: "Security"
domain: "Cost Management"
tags:
  - "IAM Identity Center"
  - "Active Directory"
  - "SAML"
  - "Cost"
created_at: "2026-02-05"
---

# IAM Identity Center with On-Prem AD (SAP-63)

## 1. 状況の読み解き (Context & Requirements)

- **構成図の分析**: On-Prem AD を ID の単一源泉 (Single Source of Truth) にしたい。
- **必須要件**:
    - **ID管理の一元化**: IAM ユーザー廃止、On-Prem AD 利用。
    - **コスト最小化**: "初期費用と月次コストも最小に抑える" が最重要。
    - **マネージド**: パッチ管理等は任せたい（が、コスト制約が優先）。

## 2. 勝負を決めるキーワード (Trigger Words)

- **"初期費用と月次コストも最小に抑える"**: AWS Managed Microsoft AD は高額（Standardでも月額数百ドル〜）。SAML 連携（Identity Center 自体は無料、AD FS はオンプレ既存なら追加費用なし）が最安。
- **"社内 Active Directory だけを ID 管理の単一源泉"**: 独立したディレクトリを作る D はNG。

## 3. なぜその答えか (Reasoning)

- **論理的根拠 (A)**:
    - **IAM Identity Center (旧 SSO)** は、外部 ID プロバイダー (IdP) との SAML 2.0 連携をサポートしています。オンプレミスの AD FS 等を IdP として利用すれば、AWS 側に高額な Managed AD を構築する必要がなく、**追加の AWS ランニングコストはほぼゼロ**（Organizations/SSOは無料）で済みます。これが「コスト最小」の要件に合致する唯一の解です。
    - IAM Identity Center の自動プロビジョニング機能（SCIM）や SAML アサーションを使えば、AD グループに基づいた権限管理も可能です。
- **選択肢の分析**:
    - **B, C (Managed AD + Trust)**: AWS Managed Microsoft AD を構築し、オンプレ AD と信頼関係を結ぶ構成は技術的に正解でよく使われますが、**コストが高い** です。Managed AD の稼働費がかかるため、「コスト最小化」の要件においては SAML 連携 (A) に劣ります。
    - **D (独立 Managed AD)**: オンプレミス AD と信頼関係を結ばない場合、ユーザーはオンプレとは別に管理することになり、「ID 管理の単一源泉」という要件を満たしません。

## 4. 比較と公式資料 (Comparison)

- **比較表**:

| Integration Method | AWS Cost | Admin Effort | Use Case |
| :--- | :--- | :--- | :--- |
| **SAML 2.0 (A)**<br>(AD FS etc.) | **Low (Free)** | Medium (On-prem maintenance) | コスト重視、既存IdP活用 |
| **AWS Managed AD + Trust (B/C)** | **High** | Low (Managed service) | 互換性重視、レガシーアプリ対応 |
| **AD Connector** | Low/Medium | Low | 小規模、プロキシ利用 |

- **詳細資料**: [外部 ID プロバイダーへの IAM Identity Center の接続](https://docs.aws.amazon.com/ja_jp/singlesignon/latest/userguide/manage-your-identity-source-idp.html)

## 5. 学習メモ (Exam Note)

■ AWS SAP Exam Note: IAM Identity Center & AD Integration (SAP-63)

> ### 📌 Core Concept
> On-Prem AD ユーザーで AWS にログインする方法は主に2つ。
> 1. **SAML連携 (AD FS等)**: コスト最安。AWS側のディレクトリ不要。
> 2. **Managed AD + 信頼関係**: 互換性最強だが高額。

> ### 🛠 Trigger ➔ Best Practice
> | 注目要件 (Trigger) | 採るべき構成 (Best Practice) |
> | :--- | :--- |
> | "コスト最小化" + "On-Prem AD利用" | **SAML 2.0** 連携 (外部IdP接続) |
> | "OSログインも統合" / "レガシー要件" | **AWS Managed Microsoft AD** + 信頼関係 |

> ### ⚠️ Pitfall / Anti-Pattern
> - "AD Connector" も安価だが、IAM Identity Center のデータソースとしては制限がある（現在は利用可能だが、コスト最小ならSAMLが優先されることが多い）。ただし選択肢にないので迷わない。
> - "Simple AD" は信頼関係を結べないので、On-Prem AD 統合要件では使えない。

> ### 💡 Exam Shortcut
> Cost Effectiveness & AD = **SAML (IdP)**
