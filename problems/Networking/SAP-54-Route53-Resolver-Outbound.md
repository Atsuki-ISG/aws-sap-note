---
id: "SAP-54"
title: "Route 53 Resolver Outbound Endpoint"
related_note: "../../notes/Networking/SAP-54-Route53-Resolver-Outbound.md"
---

# Problem Statement

【SAP-54】 ある医療機器メーカーでは、社員専用ポータルを VPC 内の Amazon EC2 で新規構築し、Site-to-Site VPN によりオンプレミスと接続している。
ポータルは生産計画や品質管理 API など既存オンプレミスサービスと連携し、これらはすべて corp.internal プライベート DNS ゾーンのホスト名で公開される。
同社はコンプライアンス上、権威 DNS を社内データセンターでのみ運用し、AWS へゾーンを複製しない方針を採っており、運用部門は可用性とスケーラビリティを確保しつつ EC2 からの名前解決を自動化したい。
こうした背景を踏まえ、ハイブリッド環境を設計するソリューションアーキテクトは、VPC 内インスタンスが corp.internal のホスト名を解決できるようにするため、どのアーキテクチャ／サービス設定を採用すべきか？

# Options

A. VPC の DNS ホスト名を有効化し、Amazon Route 53 Resolver のアウトバウンドエンドポイントを作成する。corp.internal へのクエリをオンプレミス DNS へ転送するルールを作成し、そのルールを VPC に関連付ける。
B. Systems Manager ドキュメントで hosts ファイルを配布し、全 EC2 に corp.internal の必要なホスト名と IP アドレスの静的エントリを定期的に注入する。
C. Route 53 で corp.internal の空のプライベートホストゾーンを作成し、オンプレミスの corp.internal ゾーンに NS レコードで Route 53 のネームサーバーを登録する。
D. VPC の DNS ホスト名を有効化し、Route 53 Resolver のインバウンドエンドポイントを作成する。オンプレミス DNS から corp.internal クエリを新しいインバウンドエンドポイントにフォワードするよう設定する。

# Answer
A
