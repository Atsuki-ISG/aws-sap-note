---
id: "SAP-59"
title: "Multi-Region Active-Passive DR"
category: "Architecture"
domain: "Design for New Solutions"
tags:
  - "Route 53"
  - "Disaster Recovery"
  - "Active/Passive"
  - "Multi-Region"
created_at: "2026-02-05"
---

# Multi-Region Active-Passive DR (SAP-59)

## 1. 状況の読み解き (Context & Requirements)

- **構成図の分析**: us-east-1 (Primary) と us-west-1 (Secondary/Standby) のマルチリージョン構成。
- **必須要件**:
    - **Active/Passive**: 災害時のみ切り替える待機系構成。
    - **自動スケール**: 急増するアクセスに対応 (Auto Scaling)。
    - **迅速な切り替え**: 工数を増やさず自動化されていること (Route 53 Health Check + Failover)。

## 2. 勝負を決めるキーワード (Trigger Words)

- **"アクティブ／パッシブ形式の待機系"**: Active/Active (レイテンシベース等) ではなく、Failover Routing を選ぶ強い根拠。
- **"フェイルオーバーできる状態"**: Route 53 Failover Routing Policy。

## 3. なぜその答えか (Reasoning)

- **論理的根拠 (A)**:
    - 要件である「Active/Passive」を実現する最も標準的な AWS パターンです。
    - 両リージョンに独立したスタック (VPC, ALB, ASG) を構築し、Route 53 の **フェイルオーバールーティングポリシー (Failover Routing Policy)** を使用します。
    - プライマリ (us-east-1) にヘルスチェックを設定し、正常時はすべてプライマリへ、異常時のみセカンダリ (us-west-1) へ DNS 回答を切り替えることで、迅速かつ自動的な DR 切り替えが可能です。
- **選択肢の分析**:
    - **B (地域間ALB)**: **ALB はリージョンを跨げません**。VPC ピアリングがあっても、ターゲットグループに別リージョンのインスタンスを IP 登録することは技術的に可能ですが（レイテンシ大）、ALB 自体は単一リージョン障害の単一障害点 (SPOF) となるため DR として成立しません。
    - **C (Active/Active)**: 「アクティブ／パッシブ形式」という要件に矛盾します。レイテンシベースルーティングは Active/Active 用です。
    - **D (デフォルトポリシー)**: シンプルルーティング（または複数値回答）でヘルスチェックをつける構成ですが、これだと「正常時はプライマリのみ」という制御が難しく、ラウンドロビン的に分散される可能性があります。「アクティブ／パッシブ」を明示するならフェイルオーバールーティング (A) が正解です。

## 4. 比較と公式資料 (Comparison)

- **比較表**:

| Routing Policy | Use Case | Works as DR? |
| :--- | :--- | :--- |
| **Failover** | Active/Passive | **Yes** (Primary fail -> Secondary) |
| **Latency** | Active/Active | Yes (User routed to fastest healthy region) |
| **Geolocation** | Compliance/Content restriction | Maybe (but complex for pure DR) |
| **Simple/Multivalue** | Load distribution | No (Requires explicit Active/Passive logic) |

- **詳細資料**: [Amazon Route 53 のフェイルオーバールーティングポリシー](https://docs.aws.amazon.com/ja_jp/Route53/latest/DeveloperGuide/routing-policy-failover.html)

## 5. 学習メモ (Exam Note)

■ AWS SAP Exam Note: Route 53 DR Patterns (SAP-59)

> ### 📌 Core Concept
> DR戦略（Active/Passive）の切り替えトリガーは Route 53 フェイルオーバールーティングが基本。
> ALBなどリージョンサービスは各リージョンに独立して持たせる（リージョンまたぎ共有はNG）。

> ### 🛠 Trigger ➔ Best Practice
> | 注目要件 (Trigger) | 採るべき構成 (Best Practice) |
> | :--- | :--- |
> | Active/Passive (待機系) | **Failover** Routing Policy |
> | Active/Active (性能重視) | **Latency** Routing Policy |
> | 地域ごとの規制 (GDPR等) | **Geolocation** Routing Policy |

> ### ⚠️ Pitfall / Anti-Pattern
> - "ALB をリージョン間で共有する" 選択肢は実現不可能またはアンチパターン（SPOF）。
> - Active/Passive 要件で Latency Routing を選ばないこと（両方稼働してしまう）。

> ### 💡 Exam Shortcut
> Active/Passive = **Failover** Routing
> Active/Active = **Latency** Routing
