# 電子帳簿保存法（電帳法）調査レポート {#sec1}

## 調査日 {#sec2}

2026-03-30

## 背景 {#sec3}

請求書・納品書をPDF生成してメール（SendGrid）で得意先に送信するシステムを開発中。
送信側（東海鋼材工業様）としての法的義務を整理する。

## 前提 {#sec4}

- **東海鋼材工業様の売上高は5,000万円超** → 検索要件は必須
- 現在は紙で請求書・納品書を発行しており、紙の控えを保存 → **合法**
- 新システムでメール（PDF）送信に切り替えると「電子取引」となり、保存ルールが変わる
  - 送信したPDFの控えを**紙に印刷して保存するのはNG**
  - **電子データのまま保存する義務**が発生する

## 結論 {#sec5}

<table style="table-layout: fixed;">
<colgroup>
<col style="width: 40%;">
<col style="width: 60%;">
</colgroup>
<thead>
<tr><th>項目</th><th>結論</th></tr>
</thead>
<tbody>
<tr><td>メールPDF送信は「電子取引」か</td><td><strong>該当する</strong></td></tr>
<tr><td>送信側の保存義務</td><td><strong>ある</strong>（PDFの控えを電子データのまま保存）</td></tr>
<tr><td>電子送信した控えを紙印刷で保存できるか</td><td><strong>不可</strong>（電子データのまま保存が必須。2024年1月完全義務化済）</td></tr>
<tr><td>保存期間</td><td><strong>7年間</strong>（法人）</td></tr>
<tr><td>検索要件</td><td><strong>必須</strong>（売上高5,000万円超のため）</td></tr>
</tbody>
</table>

## 検索要件の詳細 {#sec6}

以下の検索機能を**システムに実装する必要がある**：

1. **取引年月日** での検索（範囲指定可）
2. **取引金額** での検索（範囲指定可）
3. **取引先名** での検索
4. 上記の**組み合わせ検索**

## 改ざん防止措置（いずれか1つ） {#sec7}

<table style="table-layout: fixed;">
<colgroup>
<col style="width: 25%;">
<col style="width: 15%;">
<col style="width: 60%;">
</colgroup>
<thead>
<tr><th>方法</th><th>コスト</th><th>内容</th></tr>
</thead>
<tbody>
<tr><td>事務処理規程の策定</td><td><strong>低（推奨）</strong></td><td>国税庁サンプルを参考に規程を作成・運用</td></tr>
<tr><td>システムで履歴管理</td><td>中</td><td>訂正・削除の履歴を残す仕組み</td></tr>
<tr><td>タイムスタンプ付与</td><td>高</td><td>認定タイムスタンプサービスの利用</td></tr>
</tbody>
</table>

## 法的根拠 {#sec8}

### 条文 {#sec9}

<table style="table-layout: fixed;">
<colgroup>
<col style="width: 35%;">
<col style="width: 65%;">
</colgroup>
<thead>
<tr><th>根拠</th><th>内容</th></tr>
</thead>
<tbody>
<tr><td><strong>電子帳簿保存法 第7条</strong></td><td>電子取引データの保存義務（本則）</td></tr>
<tr><td><strong>施行規則 第2条第6項第5号</strong></td><td>検索要件の具体的内容</td></tr>
<tr><td>├ イ</td><td>取引年月日・取引金額・取引先を検索条件として設定できること</td></tr>
<tr><td>├ ロ</td><td>日付・金額について範囲を指定して検索できること</td></tr>
<tr><td>└ ハ</td><td>二以上の任意の記録項目を組み合わせて検索できること</td></tr>
<tr><td><strong>施行規則 第4条第1項 柱書</strong></td><td>売上高5,000万円以下の場合の検索要件免除（※東海鋼材工業様は該当しない）</td></tr>
</tbody>
</table>

### 国税庁 一問一答（電子取引関係） {#sec10}

<table style="table-layout: fixed;">
<colgroup>
<col style="width: 15%;">
<col style="width: 85%;">
</colgroup>
<thead>
<tr><th>Q番号</th><th>内容</th></tr>
</thead>
<tbody>
<tr><td>問42</td><td>検索機能で注意すべき点</td></tr>
<tr><td>問43</td><td>組み合わせ検索の範囲（「AかつB」のみ、「A又はB」は不要）</td></tr>
<tr><td>問44</td><td>システムがない場合の検索機能確保方法（ファイル命名規則等）</td></tr>
<tr><td>問45</td><td>5,000万円以下の判定方法（基準期間＝2期前の売上高）</td></tr>
<tr><td>問51</td><td>検索要件の「取引金額」は税込・税抜どちらでもよい（統一すること）</td></tr>
</tbody>
</table>

### 参考URL {#sec11}

- <a href="https://www.nta.go.jp/law/joho-zeikaishaku/sonota/jirei/pdf/0024005-113_r603.pdf">国税庁 一問一答【電子取引関係】令和6年6月版（PDF）</a>
- <a href="https://www.nta.go.jp/law/joho-zeikaishaku/sonota/jirei/4-3.htm">国税庁 一問一答ポータルページ</a>
- <a href="https://www.nta.go.jp/law/joho-zeikaishaku/sonota/jirei/tokusetsu/index.htm">国税庁 電子帳簿等保存制度特設サイト</a>
- <a href="https://laws.e-gov.go.jp/law/410M50000040043/">e-Gov 電子帳簿保存法施行規則</a>
- <a href="https://www.nta.go.jp/law/joho-zeikaishaku/sonota/jirei/0021006-031.htm">国税庁 各種規程等のサンプル</a>
  - <a href="https://www.nta.go.jp/law/joho-zeikaishaku/sonota/jirei/word/0021006-031_d.docx">事務処理規程サンプル（法人用・Word）</a>

## 罰則・ペナルティ {#sec12}

### 直接的な罰則 {#sec13}

電帳法自体には、保存義務違反・検索要件不備・改ざん防止措置不備に対する**直接的な刑事罰（罰金・懲役等）は規定されていない**。

### 間接的な不利益（実質的に大きなダメージ） {#sec14}

<table style="table-layout: fixed;">
<colgroup>
<col style="width: 22%;">
<col style="width: 23%;">
<col style="width: 55%;">
</colgroup>
<thead>
<tr><th>不利益</th><th>根拠条文</th><th>内容</th></tr>
</thead>
<tbody>
<tr><td>青色申告の承認取消</td><td>法人税法第127条第1項</td><td>欠損金繰越控除、各種優遇措置が使えなくなる</td></tr>
<tr><td>経費（損金）否認</td><td>法人税法第22条第3項</td><td>保存不備の取引の経費が認められない</td></tr>
<tr><td>消費税の仕入税額控除否認</td><td>消費税法第30条第7項</td><td>請求書の保存不備で控除できない</td></tr>
<tr><td>推計課税</td><td>法人税法第131条</td><td>青色申告取消後、実額より不利な課税の可能性</td></tr>
</tbody>
</table>

### 重加算税の加重措置（令和4年度税制改正） {#sec15}

電子取引データの**隠蔽・仮装**があった場合、通常の重加算税に**10%が加重**される（国税通則法第68条第4項）。

<table style="table-layout: fixed;">
<colgroup>
<col style="width: 30%;">
<col style="width: 35%;">
<col style="width: 35%;">
</colgroup>
<thead>
<tr><th>区分</th><th>通常税率</th><th>加重後</th></tr>
</thead>
<tbody>
<tr><td>過少申告</td><td>35%</td><td><strong>45%</strong></td></tr>
<tr><td>無申告</td><td>40%</td><td><strong>50%</strong></td></tr>
</tbody>
</table>

### 根拠条文・参考情報 {#sec16}

<table style="table-layout: fixed;">
<colgroup>
<col style="width: 35%;">
<col style="width: 65%;">
</colgroup>
<thead>
<tr><th>根拠</th><th>内容</th></tr>
</thead>
<tbody>
<tr><td><strong>国税通則法 第68条第4項</strong></td><td>電子取引データの隠蔽・仮装に対する重加算税10%加重</td></tr>
<tr><td><strong>法人税法 第127条第1項第1号</strong></td><td>帳簿書類の保存不備による青色申告承認取消</td></tr>
<tr><td><strong>法人税法 第22条第3項</strong></td><td>損金算入の要件（保存不備で否認の根拠）</td></tr>
<tr><td><strong>消費税法 第30条第7項</strong></td><td>請求書等の保存を仕入税額控除の要件とする規定</td></tr>
<tr><td><strong>法人税法 第131条</strong></td><td>青色申告取消後の推計課税</td></tr>
</tbody>
</table>

### まとめ {#sec17}

罰金・懲役はないが、税務調査で指摘された場合の金銭的ダメージは大きい。「罰則がないから対応不要」とは言えない。

## システムへの影響 {#sec18}

### 必須対応 {#sec19}

1. **PDF控えの電子保存**: S3に保存（設計済み）→ **7年間保持ポリシーの設定が必要**
2. **検索機能の実装**: 処理済み一覧に日付・金額・取引先の範囲指定・組み合わせ検索を追加
3. **改ざん防止措置**: 事務処理規程の策定（最低限）、またはシステムで訂正・削除履歴を管理

### 要件定義で確認すべき事項 {#sec20}

- 改ざん防止措置の方針（規程 or システム対応）
- 既存の電帳法対応状況（他システムで対応済みか）
- 保存期間の方針（7年 or 安全策で10年）
