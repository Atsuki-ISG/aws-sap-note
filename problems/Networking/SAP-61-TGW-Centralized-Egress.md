---
id: "SAP-61"
title: "Centralized Egress with TGW"
related_note: "../../notes/Networking/SAP-61-TGW-Centralized-Egress.md"
---

# Problem Statement

【SAP-61】 あるグローバル小売企業は、ECサイトおよび在庫連携システムを支える SaaS 基盤を AWS 上で運用している。
顧客向け API や分析バッチなどのワークロードを数百の AWS アカウントに展開し、各 VPC は複数 AZ に跨るパブリック / プライベートサブネットを持つ。
利用者はモバイルアプリや店舗端末から HTTPS でサービスを呼び出し、バッチは外部 API と通信するためアウトバウンド接続が不可欠である。
現在は各 VPC のパブリックサブネットに NAT Gateway を配置しているが、①コスト削減、②通信ログを中央で監査するという経営判断により、ネットワーク部門はハブ＆スポーク方式に統合したいと考えている。
運用ポリシー上、各ワークロードチームが変更できるのは自アカウント内のルートテーブルのみであり、ネットワーク基盤は数百 VPC 規模でも性能劣化なく水平展開できることが前提となる。
ソリューションアーキテクトは、メインアカウントの Egress VPC にすでに配置した NAT Gateway を全スポーク VPC から共用し、プライベートサブネットのインターネット向けトラフィックを原則として同 VPC 経由に集約する必要がある。
ビジネス上の狙いを踏まえつつ、ルーティングループを避け帯域を確保するという技術的制約も満たさねばならない。
この状況でソリューションアーキテクトは、インフラ設計レベルでどのような構成を採用すべきか？

# Options

A. Egress VPC と各スポーク VPC の間に VPC ピアリング接続を多数作成し、プライベートサブネットからのデフォルトルートをすべて Egress VPC へのピアリングに向ける。 Egress VPC 側ではルートテーブルで各スポーク VPC 宛の戻りルートを個別に設定する。
B. 各アカウントごとに専用の Transit Gateway を新規作成し、アカウント内の VPC と Egress VPC の両方をそれぞれの Transit Gateway にアタッチする。インターネット向けトラフィックはまずローカルの Transit Gateway を経由し、そこから Egress VPC へ多段ルーティングする。
C. AWS Transit Gateway を使い、全スポーク VPC のプライベートサブネットからのデフォルトルートを TGW 経由で Egress VPC の NAT Gateway に集約する。 TGW ルートテーブルで 0.0.0.0/0 を Egress VPC アタッチメントに向け、Egress VPC 側のルートテーブルでインターネット宛を NAT Gateway に向けることでルーティングループを避けつつ帯域を確保する。
D. Egress VPC 内の NAT Gateway をエンドポイントサービスとして公開し、各スポーク VPC から AWS PrivateLink （インターフェイスエンドポイント）を作成して NAT Gateway を参照する。プライベートサブネットのデフォルトルートを PrivateLink エンドポイントに向ける。

# Answer
C
