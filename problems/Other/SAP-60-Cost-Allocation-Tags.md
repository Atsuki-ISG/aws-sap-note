---
id: "SAP-60"
title: "Cost Allocation Tags for Org"
related_note: "../../notes/Other/SAP-60-Cost-Allocation-Tags.md"
---

# Problem Statement

【SAP-60】 あるオンライン決済プラットフォームを展開する会社では、複数 AWS アカウントを AWS Organizations で一括管理し、各事業部ごとに VPC を分離した上で Amazon EC2 やコンテナを自動スケールさせながら提供している。
ユーザーには決済 API を 24 時間低遅延で提供しており、その保護目的でコンプライアンスチームが各 VPC 内に専用のセキュリティツールを EC2 でデプロイし、検査結果を同チーム専用アカウントへ送信している。
監査部門から「セキュリティ関連コストを正確に把握し、月次でコンプライアンスチームへ社内請求したい」との要望が出たため、同ツールを含むリソースには costCenter=security-ops タグを付与済みである。
現在の運用では環境ごとにインスタンス数が変動するため、集計は組織全体を横断しリソースレベルまでブレークダウンできる精度が求められている。
ソリューションアーキテクトは、このタグ情報を活用しつつ、インフラレベルで追加の改修を最小化して請求額を正確に算出できるようにするために、どのアプローチを採用すべきか？

# Options

A. AWS Trusted Advisor の組織ビューを利用して、costCenter=security-ops タグを持つリソースの月次コストレポートを自動生成し、セキュリティオペレーションチームにメール送信する。
B. メンバーアカウントで costCenter タグを有効化し、管理アカウントから月次の AWS Cost and Usage Report を作成する。CUR のタグ別内訳を使って costCenter=security-ops のコストを Excel などで手動集計する運用とする。
C. メンバーアカウント側で costCenter タグをコスト配分タグとして有効化し、管理アカウントの S3 バケットに出力される CUR を毎月 Lambda 関数で集計する。Lambda は costCenter=security-ops のリソースコストを計算し、結果をセキュリティオペレーションチームに通知する。
D. 管理アカウントで costCenter タグをコスト配分タグとして有効化し、AWS Cost and Usage Report （CUR） が管理アカウントの S3 バケットに毎月出力されるよう設定する。CUR のタグ別内訳から costCenter=security-ops の合計コストを抽出し、社内請求に利用する。

# Answer
D
