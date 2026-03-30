# Research & Design Decisions

---
**Purpose**: 請求書・納品書メール送信システムの技術設計に関する調査結果と設計判断の記録

**Usage**:
- 設計フェーズでの調査活動と結果の記録
- design.mdには詳細すぎる設計判断のトレードオフを文書化
- 将来の監査や再利用のための参照資料
---

## Summary
- **Feature**: `invoice-delivery-mail-system`
- **Discovery Scope**: New Feature（グリーンフィールド開発）
- **Key Findings**:
  - Streamlitは社内向け管理画面としてMVP開発に最適（数時間で構築可能）
  - SendGrid Python SDKは添付ファイル付きメール送信に十分な機能を提供
  - Selenium（ヘッドレス）は軽技Webのようなjavascriptレンダリングが必要なBIツールのスクレイピングに適している
  - AWS Lambda + EventBridgeは定時バッチ処理に最適（15分制限に注意）
  - PDF生成はWeasyPrintがHTML/CSSベースで開発効率が高い

## Research Log

### 軽技Webスクレイピング技術
- **Context**: 軽技Web（BIツール）から売上明細データをCSV形式で取得する必要がある
- **Sources Consulted**:
  - [ZenRows - Web Scraping With a Headless Browser](https://www.zenrows.com/blog/headless-browser-python)
  - [iProyal - How to Run Selenium in Headless Mode with Python in 2026](https://iproyal.com/blog/headless-browser-python/)
  - [ScrapingBee - How to master Selenium web scraping in 2026](https://www.scrapingbee.com/blog/selenium-python/)
- **Findings**:
  - Seleniumは最も成熟したブラウザ自動化ツールで、JavaScript実行が必要なページに適している
  - ヘッドレスモードはGUIなしで動作し、サーバー環境やバックグラウンドジョブに最適
  - 2026年時点でSelenium Managerが自動でドライバー管理を行うため、手動でのChromeDriver管理は不要
  - 推奨設定: `--headless=new`, `--disable-gpu`, `--no-sandbox`, `--window-size=1920,1080`
  - Playwrightも代替候補だが、既存のPythonエコシステムとの親和性でSeleniumを選択
- **Implications**:
  - AWS環境でのヘッドレスブラウザ実行にはLambda Layerまたはコンテナ化が必要
  - スクレイピング処理時間が長くなる可能性があり、Lambda 15分制限に注意

### PDF生成ライブラリ選定
- **Context**: 請求書・納品書PDFをテンプレートから生成する必要がある
- **Sources Consulted**:
  - [DEV Community - Generate PDFs in Python: WeasyPrint vs ReportLab](https://dev.to/claudeprime/generate-pdfs-in-python-weasyprint-vs-reportlab-ifi)
  - [Nutrient - Top 10 Python PDF generator libraries](https://www.nutrient.io/blog/top-10-ways-to-generate-pdfs-in-python/)
  - [PDForge - The Best Python Libraries for PDF Generation in 2025](https://pdforge.com/blog/the-best-python-libraries-for-pdf-generation-in-2025)
- **Findings**:
  - WeasyPrint: HTML/CSSベースで開発が容易、Web開発スキルが活かせる
  - ReportLab: 座標ベースで精密なレイアウト制御が可能だが学習コストが高い
  - 「90%のケースでWeasyPrintの方がシンプルで開発が速い」
  - 請求書・納品書のような定型フォーマットにはWeasyPrintが適している
- **Implications**:
  - HTML/CSSテンプレートで請求書・納品書のデザインを管理できる
  - 既存の紙ベース原本のレイアウトを再現しやすい

### メール送信サービス（SendGrid）
- **Context**: 毎日10時に承認済みの請求書・納品書をメール送信する
- **Sources Consulted**:
  - [Twilio - Sending Email Attachments with SendGrid and Python](https://www.twilio.com/en-us/blog/sending-email-attachments-with-twilio-sendgrid-python)
  - [sendgrid-python GitHub](https://github.com/sendgrid/sendgrid-python/blob/main/use_cases/attachment.md)
  - [SendGrid Docs - v3 API Python Code Example](https://docs.sendgrid.com/for-developers/sending-email/v3-python-code-example)
- **Findings**:
  - sendgrid-python SDKで添付ファイル付きメール送信が容易
  - APIキーは環境変数で管理（セキュリティベストプラクティス）
  - レート制限の考慮が必要（429エラーへのリトライ実装）
  - 送信者メールアドレスは事前にSendGridで検証が必要
  - 送信履歴はSendGrid管理画面で確認可能（本システムでの履歴管理不要）
- **Implications**:
  - PDF添付にはbase64エンコードが必要
  - バッチ送信時のレート制限対策として間隔を空けた送信が推奨

### フロントエンド技術選定
- **Context**: 管理画面のフロントエンド技術を選定する
- **Sources Consulted**:
  - [Deep Learning Nerds - Streamlit vs React](https://www.deeplearningnerds.com/streamlit-vs-react-choosing-the-right-framework-for-your-web-app/)
  - [Medium - Django/React vs Streamlit](https://medium.com/@data.dev.backyard/django-react-vs-streamlit-which-one-to-choose-for-your-startup-r-d-project-b51d884b84ca)
  - [Data Revenue - Streamlit vs Dash vs Shiny](https://www.datarevenue.com/en-blog/data-dashboarding-streamlit-vs-dash-vs-shiny-vs-voila)
- **Findings**:
  - Streamlit: Pythonのみで数時間でMVPが構築可能、データサイエンティスト向け
  - React: 本番環境向け、高度なカスタマイズが可能だが開発に数週間~数ヶ月
  - 社内ツール・MVPにはStreamlitが最適
  - IP制限のみの認証（ログイン画面不要）という要件にStreamlitは適合
- **Implications**:
  - 開発コスト・期間を大幅に削減可能
  - 将来的に機能拡張が必要な場合はReactへの移行を検討

### AWS インフラストラクチャ
- **Context**: AWS環境でのデータベースとバッチ処理基盤を選定
- **Sources Consulted**:
  - [Vantage - RDS vs Aurora Pricing](https://www.vantage.sh/blog/aws-rds-vs-aurora-pricing-in-depth)
  - [Bytebase - Aurora vs RDS Pricing Comparison](https://www.bytebase.com/blog/aws-aurora-vs-rds-pricing/)
  - [AWS EventBridge Tutorial](https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-run-lambda-schedule.html)
  - [AWS S3 Presigned URLs Best Practices](https://docs.aws.amazon.com/prescriptive-guidance/latest/presigned-url-best-practices/overview.html)
- **Findings**:
  - **RDS PostgreSQL**: 予測可能な低トラフィックに適している、無料枠あり、t3.microから開始可能
  - **Aurora Serverless v2**: 変動するトラフィックに適しているが、無料枠なし
  - **Lambda + EventBridge**: cron式でスケジュール実行、最大15分のタイムアウト
  - **S3 Presigned URL**: PDF保存とダウンロードに適している、有効期限は最大7日
- **Implications**:
  - 小規模アプリケーションにはRDS PostgreSQLがコスト効率が良い
  - バッチ処理が15分を超える場合はStep Functionsの検討が必要
  - PDFファイルはS3に保存し、必要時にPresigned URLで取得

### FastAPI アーキテクチャ
- **Context**: バックエンドAPIフレームワークの設計パターン
- **Sources Consulted**:
  - [GitHub - fastapi-best-practices](https://github.com/zhanymkanov/fastapi-best-practices)
  - [Render - FastAPI production deployment best practices](https://render.com/articles/fastapi-production-deployment-best-practices)
  - [Medium - FastAPI Project Structure for Large Applications](https://medium.com/@devsumitg/the-perfect-structure-for-a-large-production-ready-fastapi-app-78c55271d15c)
- **Findings**:
  - Gunicorn + Uvicornワーカーが本番環境の標準構成
  - 非同期処理はasync/awaitを適切に使用（ブロッキング操作に注意）
  - 依存性注入（DI）パターンでテスタブルなコードを実現
  - APIキーは環境変数で管理（.envファイルは開発環境のみ）
- **Implications**:
  - 関心の分離: ルーティング、モデル、スキーマ、サービス、データベース
  - スクレイピング等の長時間処理はバックグラウンドタスクで実行

## Architecture Pattern Evaluation

| Option | Description | Strengths | Risks / Limitations | Notes |
|--------|-------------|-----------|---------------------|-------|
| レイヤードアーキテクチャ | プレゼンテーション → サービス → データ | シンプル、理解しやすい | 大規模化時に依存関係が複雑化 | 本システムの規模に適合 |
| クリーンアーキテクチャ | ドメイン中心、依存性逆転 | テスタブル、保守性高い | 小規模システムにはオーバーエンジニアリング | 将来の拡張時に検討 |
| マイクロサービス | 機能ごとに分離 | スケーラブル | 運用コスト高、複雑性増大 | 本システムには不適合 |

**選択**: レイヤードアーキテクチャ（FastAPI + Streamlit + PostgreSQL）
**理由**: システム規模に対して適切な複雑さ、開発チームの学習コスト最小化、AWS環境への容易なデプロイ

## Design Decisions

### Decision: フロントエンド技術（Streamlit選択）
- **Context**: 社内向け管理画面の構築に適した技術を選定
- **Alternatives Considered**:
  1. React + TypeScript — 高度なカスタマイズ、本番品質のUI
  2. Streamlit — Pythonのみで迅速な開発
  3. Django Admin — 標準的な管理機能
- **Selected Approach**: Streamlit
- **Rationale**:
  - IP制限のみの認証でログイン画面不要という要件にマッチ
  - Pythonバックエンド（FastAPI）との統合が容易
  - 開発期間の大幅短縮（数時間でMVP構築可能）
  - データテーブル、フォーム、ボタン等のUIコンポーネントが標準搭載
- **Trade-offs**:
  - カスタマイズ性はReactより低い
  - 将来的に複雑なUIが必要になった場合は移行が必要
- **Follow-up**: ユーザビリティテスト後に必要に応じてUI改善

### Decision: データベース（RDS PostgreSQL選択）
- **Context**: 売上明細データと承認状態を永続化するデータベースを選定
- **Alternatives Considered**:
  1. RDS PostgreSQL — 予測可能なワークロード向け、低コスト
  2. Aurora Serverless v2 — 変動するワークロード向け、スケーラブル
  3. DynamoDB — NoSQL、サーバーレス
- **Selected Approach**: RDS PostgreSQL（t3.micro）
- **Rationale**:
  - 予測可能な低トラフィック（社内数名が利用）
  - 無料枠が利用可能（12ヶ月間）
  - リレーショナルデータ（請求書、納品書、承認状態）に適している
  - PostgreSQLはJSON型もサポートし柔軟性がある
- **Trade-offs**:
  - トラフィック急増時のスケーリングは手動対応が必要
  - Aurora Serverlessのような自動スケーリングなし
- **Follow-up**: 利用状況をモニタリングし、必要に応じてインスタンスサイズ変更

### Decision: PDF生成（WeasyPrint選択）
- **Context**: 請求書・納品書PDFを生成する技術を選定
- **Alternatives Considered**:
  1. WeasyPrint — HTML/CSSベース、開発効率高い
  2. ReportLab — 座標ベース、精密レイアウト
  3. fpdf2 — 軽量、シンプル
- **Selected Approach**: WeasyPrint
- **Rationale**:
  - HTML/CSSでテンプレートを作成でき、Webスキルが活かせる
  - 既存の紙ベース原本のレイアウト再現が容易
  - 「90%のケースでReportLabより開発が速い」
- **Trade-offs**:
  - ReportLabより処理が遅い可能性
  - 非常に複雑なレイアウトには不向き
- **Follow-up**: テンプレート作成後にパフォーマンス検証

### Decision: バッチ処理（Lambda + EventBridge選択）
- **Context**: 毎日10時のメール送信バッチ処理基盤を選定
- **Alternatives Considered**:
  1. Lambda + EventBridge — サーバーレス、cron式スケジュール
  2. EC2 + crontab — 従来型、常時起動コスト
  3. ECS Scheduled Tasks — コンテナベース
- **Selected Approach**: Lambda + EventBridge Scheduler
- **Rationale**:
  - サーバーレスで運用コスト最小化
  - cron式（`0 1 * * ? *` UTC = 日本時間10時）でスケジュール設定
  - 実行時間のみ課金
  - リトライ機能内蔵
- **Trade-offs**:
  - 15分のタイムアウト制限
  - 大量データ処理時はStep Functionsが必要
- **Follow-up**: 送信件数が増加した場合のタイムアウト監視

## Risks & Mitigations

| Risk | Impact | Mitigation |
|------|--------|------------|
| 軽技Webのログイン/スクレイピング失敗 | データ取得不可 | リトライ機能実装、エラー通知、手動トリガー機能 |
| Lambda 15分タイムアウト | バッチ処理未完了 | 処理の分割、Step Functions導入検討 |
| SendGrid レート制限 | メール送信遅延 | 間隔を空けた送信、リトライキュー |
| 軽技WebのUI変更 | スクレイピング破損 | セレクタの抽象化、監視アラート、定期的な動作確認 |
| PDF生成のメモリ不足（Lambda） | 生成失敗 | メモリ設定調整、ECS Fargateへの移行検討 |

## References

- [Twilio SendGrid Python SDK](https://github.com/sendgrid/sendgrid-python) — メール送信ライブラリ
- [Selenium Python Documentation](https://selenium-python.readthedocs.io/) — ブラウザ自動化
- [WeasyPrint Documentation](https://doc.courtbouillon.org/weasyprint/stable/) — PDF生成
- [FastAPI Documentation](https://fastapi.tiangolo.com/) — バックエンドフレームワーク
- [Streamlit Documentation](https://docs.streamlit.io/) — フロントエンドフレームワーク
- [AWS EventBridge Scheduler](https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-run-lambda-schedule.html) — バッチスケジューリング
- [AWS S3 Presigned URLs](https://docs.aws.amazon.com/prescriptive-guidance/latest/presigned-url-best-practices/overview.html) — ファイルアクセス
