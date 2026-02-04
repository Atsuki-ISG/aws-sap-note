---
id: "SAP-57"
title: "Oracle to Aurora Migration"
category: "Database"
domain: "Acceleration of workload migration and modernization"
tags:
  - "Oracle"
  - "Aurora MySQL"
  - "SCT"
  - "DMS"
  - "Migration"
created_at: "2026-02-05"
---

# Oracle to Aurora Migration (SAP-57)

## 1. 状況の読み解き (Context & Requirements)

- **構成図の分析**: On-Prem Oracle → AWS Aurora MySQL (異種エンジン移行)。
- **必須要件**:
    - **異種移行**: Oracle から MySQL へのスキーマ変換が必要。
    - **最小ダウンタイム**: 数分以内の切り替え = CDC (継続的レプリケーション) が必須。
    - **承認済みツール**: AWS SCT (Schema Conversion Tool) と AWS DMS (Database Migration Service)。
    - **工数極小化**: 手動作業は避ける。

## 2. 勝負を決めるキーワード (Trigger Words)

- **"Oracle ... を Amazon Aurora MySQL へ移行"**: 異種エンジン間移行 (Heterogeneous Migration) なので、**SCT** が必須。
- **"切り替え時のダウンタイムを数分以内"**: **DMS** の **"Full Load + CDC (Ongoing Replication)"** 設定が必要。

## 3. なぜその答えか (Reasoning)

- **論理的根拠 (A)**:
    - 異種データベース移行の標準的なベストプラクティスは **「SCT でスキーマ変換」 + 「DMS でデータ移行」** です。
    - 選択肢 A は、SCT でスキーマ評価・変換を行い、DMS で初期ロード(Full Load)と継続的な変更取り込み(CDC)を行うという、要件（工数削減・ダウンタイム最小化）を完全に満たす手順です。
- **選択肢の分析**:
    - **B (手動作成)**: "担当者工数を極小化したい" という要件に反します。SCT を使うべきです。また、フルロード中に制約チェックを有効にすると移行速度が低下したりエラーになる原因となり、推奨されません（DMSは通常、制約を無効にしてデータをロードし、後で有効化します）。
    - **C (スケール)**: DMS インスタンスのサイジングの話であり、移行アプローチ（スキーマ変換とデータの同期方法）への回答として不十分です。Multi-AZ など信頼性は考慮すべきですが、本質の解決策ではありません。
    - **D (バックアップOFF)**: データロード高速化のテクニックとして Aurora のバックアップ等を一時的に切ることはありますが、移行アプローチの選択肢としては枝葉末節です。また、ログを切るのはトラブルシューティングの観点でリスクがあります。

## 4. 比較と公式資料 (Comparison)

- **比較表**:

| Migration Type | Tools | Process |
| :--- | :--- | :--- |
| **Homogeneous** (同種)<br>例: Oracle to RDS Oracle | Native Tools<br>(Data Pump, RMAN) | バックアップ復元やログ転送が基本。SCTは不要。 |
| **Heterogeneous** (異種)<br>例: Oracle to Aurora MySQL | **AWS SCT** + **AWS DMS** | 1. SCTでスキーマ変換<br>2. DMSでデータ移行 (Full + CDC) |

- **詳細資料**: [AWS Database Migration Service のドキュメント](https://docs.aws.amazon.com/ja_jp/dms/latest/userguide/Welcome.html)

## 5. 学習メモ (Exam Note)

■ AWS SAP Exam Note: Heterogeneous Database Migration (SAP-57)

> ### 📌 Core Concept
> 異種DB移行 (Oracle -> Aurora等) は「SCTで枠を作り、DMSで中身を運ぶ」。
> ダウンタイム最小化には DMS CDC (タスク設定: Full Load + Ongoing Replication) を使う。

> ### 🛠 Trigger ➔ Best Practice
> | 注目要件 (Trigger) | 採るべき構成 (Best Practice) |
> | :--- | :--- |
> | 異種エンジン移行 (Oracle to MySQL等) | **SCT** でスキーマ変換 |
> | ダウンタイム最小化 / 継続書き込み | **DMS** (Full Load + **CDC**) |
> | 工数最小化 | 手動DDL作成(B)はNG。SCTに任せる。 |

> ### ⚠️ Pitfall / Anti-Pattern
> - DMS はスキーマ変換を行わない（簡易的な作成のみ）。異種間では SCT なしで DMS だけ実行してもうまくいかない。
> - フルロード中は外部キー制約などを無効にするのが定石（整合性エラー回避と速度向上のため）。

> ### 💡 Exam Shortcut
> 異種移行 = **SCT** (Schema) + **DMS** (Data)
