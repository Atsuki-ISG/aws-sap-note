---
id: "SAP-60"
title: "Cost Allocation Tags for Org"
category: "Other"
domain: "Cost Management"
tags:
  - "Cost Allocation Tags"
  - "AWS Organizations"
  - "CUR"
  - "Billing"
created_at: "2026-02-05"
---

# Cost Allocation Tags for Org (SAP-60)

## 1. 状況の読み解き (Context & Requirements)

- **構成図の分析**: AWS Organizations によるマルチアカウント環境。
- **必須要件**:
    - **コスト配分**: `costCenter=security-ops` タグが付いたリソースのコストを把握したい。
    - **組織横断**: 各メンバーアカウントに散らばるリソースを集計対象とする。
    - **リソースレベルの精度**: CUR (Cost and Usage Report) が必要。
    - **改修最小化**: Lambda 開発や手動集計は避けたい。

## 2. 勝負を決めるキーワード (Trigger Words)

- **"組織全体を横断" & "タグ情報を活用"**: コスト配分タグ (Cost Allocation Tags) の利用。
- **"請求額を正確に算出"**: AWS Cost and Usage Report (CUR) が最も詳細。
- **"管理アカウント"**: 組織全体の請求情報（一括請求）を扱う場所。

## 3. なぜその答えか (Reasoning)

- **論理的根拠 (D)**:
    - AWS Organizations の一括請求（Consolidated Billing）において、タグ別コストを集計するには、「**管理アカウント (Management Account)**」でそのタグを「コスト配分タグ」として有効化する必要があります。これにより、組織内の全メンバーアカウントのリソースに付与された同名のタグが、管理アカウントの請求レポート (CUR) にカラムとして追加され、集計可能になります。
    - CUR は S3 に出力され、Athena 等で分析も可能であり、要件である「リソースレベルのブレークダウン」を満たします。
- **選択肢の分析**:
    - **A (Trusted Advisor)**: コスト最適化の推奨は出せますが、特定のユーザータグに基づいた詳細な月次コストレポート生成機能はありません。
    - **B (メンバーアカウントでの有効化)**: 管理アカウントの一括請求レポートにタグを含めるには、管理アカウント側での有効化が必須です（ユーザー定義タグの場合）。また、「手動集計」は工数最小化の要件に反します。
    - **C (Lambda)**: メンバーアカウント側での有効化という点が誤りであるほか、Lambda 開発という「追加の改修」が発生するため、標準機能で済む D に劣ります。

## 4. 比較と公式資料 (Comparison)

- **比較表**:

| Action | Where to Activate Tag | Output Location |
| :--- | :--- | :--- |
| **Org-wide Reporting** | **Management Account** | Management Account's CUR (S3) |
| **Single Account Reporting** | Member Account | Member Account's Report |

- **詳細資料**: [ユーザー定義のコスト配分タグの有効化](https://docs.aws.amazon.com/ja_jp/awsaccountbilling/latest/aboutv2/activating-tags.html)

## 5. 学習メモ (Exam Note)

■ AWS SAP Exam Note: Cost Allocation Tags in Organizations (SAP-60)

> ### 📌 Core Concept
> Organizations 環境下でタグ別コストを集計する場合、タグのキーは **管理アカウント (Payar Account)** で有効化する。「メンバーアカウントでタグ付け→管理アカウントで有効化→CURで集計」の流れ。

> ### 🛠 Trigger ➔ Best Practice
> | 注目要件 (Trigger) | 採るべき構成 (Best Practice) |
> | :--- | :--- |
> | 組織横断のタグ別コスト集計 | **管理アカウント**でコスト配分タグを有効化 + **CUR** |
> | リソースレベルの詳細分析 | **Cost and Usage Report (CUR)** |
> | コストの可視化 (GUI) | Cost Explorer (ただしCURの方が生データとして詳細) |

> ### ⚠️ Pitfall / Anti-Pattern
> - ユーザー定義タグは、タグ付けしただけでは請求書に出ない。「コスト配分タグとして有効化」して初めて反映される（反映まで24時間程度）。
> - メンバーアカウントで有効化しても、管理アカウントの統合レポートには反映されない（管理アカウントでの操作が必要）。

> ### 💡 Exam Shortcut
> Orgのコストタグ = **管理アカウント**でON
