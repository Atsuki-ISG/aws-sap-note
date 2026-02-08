---
id: "SAP-61"
title: "Centralized Egress with TGW"
category: "Networking"
domain: "複雑な組織に対応するソリューションの設計 (Design Solutions for Organizational Complexity)"
tags:
  - "Transit Gateway"
  - "NAT Gateway"
  - "Centralized Egress"
  - "Cost Optimization"
created_at: "2026-02-08"
---

# Centralized Egress with TGW (SAP-61)

## 1. 状況の読み解き (Context & Requirements)

- **構成図の分析**: 数百の VPC (Spokes) からインターネットへの出口を一つ (Hub / Egress VPC) に集約したい。
- **必須要件**:
    - **ハブ＆スポーク方式**: ネットワーク統合。
    - **コスト削減 & 監査**: NAT Gateway を各VPCに置くのをやめて集約したい。
    - **スケーラビリティ**: 数百 VPC 規模。
    - **運用制約**: ワークロードチームはルートテーブルのみ変更可能。

## 2. 勝負を決めるキーワード (Trigger Words)

- **"数百の AWS アカウント" / "ハブ＆スポーク方式"**: VPC Peering は管理限界（フルメッシュや多数の1対1管理がつらい）。**Transit Gateway (TGW)** が最適解。
- **"Egress VPC に...NAT Gateway を...共用"**: "Centralized Egress" パターン。
- **"ルーティングループを避け"**: TGW ルートテーブルと VPC ルートテーブルの適切な設計が必要。

## 3. なぜその答えか (Reasoning)

- **論理的根拠 (C)**:
    - **Transit Gateway** は、数千の VPC を接続できるハブ＆スポーク構成のマネージドサービスであり、「数百 VPC 規模」に最適です。
    - **Centralized Egress 構成**:
        1. Spoke VPC のルートテーブル: `0.0.0.0/0` -> TGW。
        2. TGW ルートテーブル: `0.0.0.0/0` -> Egress VPC Attachment。
        3. Egress VPC (TGW Subnet) ルートテーブル: `0.0.0.0/0` -> NAT Gateway。
        4. NAT Gateway: インターネットへ。
    - このフローにより、NAT Gateway の集約（コスト削減）と通信ログの一元管理（監査）が実現できます。
- **選択肢の分析**:
    - **A (VPC Peering)**: 数百のピアリングを作成・管理するのは運用負荷が高すぎます。また、Egress VPC 側のルートテーブルに数百行の戻りルートを書く必要があり、ルートテーブルの上限（通常50ルート、拡張可能だが管理困難）に達するリスクがあります。
    - **B (各アカウントにTGW)**: TGW はアカウントを跨いで共有できるサービスなので、アカウントごとに作成するのはコスト増かつ管理が複雑になるだけで無意味です（TGW Peeringが必要になる）。
    - **D (PrivateLink)**: NAT Gateway は PrivateLink (Gateway Load Balancer Endpoint ではなく Interface Endpoint) のターゲットにはなれません。PrivateLink は特定のサービス公開用であり、汎用的なアウトバウンド通信（NAT）の代わりにはなりません。

## 4. 比較と公式資料 (Comparison)

- **比較表**:

| Method | Scale | Management | Cost | Use Case |
| :--- | :--- | :--- | :--- | :--- |
| **Transit Gateway (C)** | Thousands of VPCs | Easy (Hub & Spoke) | Cost for TGW attachments/data | 大規模、一元管理 |
| **VPC Peering (A)** | Small number | Hard (Mesh / Many routes) | Low (Data transfer only) | 小規模、単純な1対1 |
| **PrivateLink** | N/A for NAT | N/A | N/A | SaaS公開など |

- **詳細資料**: [トランジットゲートウェイを使用した送信トラフィックの集約](https://docs.aws.amazon.com/ja_jp/whitepapers/latest/building-scalable-secure-multi-vpc-network-infrastructure/centralized-egress.html)

## 5. 学習メモ (Exam Note)

■ AWS SAP Exam Note: TGW Centralized Egress (SAP-61)

> ### 📌 Core Concept
> 多数のVPCのアウトバウンドを集約（コスト削減・監査）するなら **Transit Gateway** + **Egress VPC**。
> ルーティングは「Spoke -> TGW -> Egress VPC -> NAT GW -> Internet」のバケツリレー。

> ### 🛠 Trigger ➔ Best Practice
> | 注目要件 (Trigger) | 採るべき構成 (Best Practice) |
> | :--- | :--- |
> | 数百VPC / ハブ＆スポーク | **Transit Gateway (TGW)** |
> | NAT Gateway 集約 / コスト削減 | **Centralized Egress Architecture** |
> | インバウンド集約 (検査) | **Gateway Load Balancer (GWLB)** or **Centralized Inspection** |

> ### ⚠️ Pitfall / Anti-Pattern
> - VPC Peering で数百接続は「管理不能」または「ルート上限」でNG。
> - PrivateLink で NAT はできない。
> - TGW のルートテーブルと VPC のルートテーブルは別物。混同しないこと。

> ### 💡 Exam Shortcut
> Many VPCs + Central NAT = **TGW**
