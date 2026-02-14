# PDF分割・統合ツール 生成プロンプト

> **使い方**: 以下のプロンプトをそのままAI（Claude、ChatGPTなど）に貼り付けてください。
> 1つのHTMLファイルとして、完全に動作するPDF分割ツールが生成されます。

---

## プロンプト（ここから下をコピーして貼り付け）

````
以下の仕様で、1つのHTMLファイルとして完結する「ローカルPDF分割・統合ツール」を作成してください。
サーバー不要・完全ブラウザ処理で、ファイルは外部に送信されません。

【重要】UI上のアイコンについて
- 絵文字（Unicode Emoji）は環境によって表示が崩れるため、一切使用しないでください。
- 代わりに、各項目の先頭に記号や文字で代用してください（例: [PDF], [ZIP], [1], [2] など）。
- アイコンが必要な場合は、テキスト記号（■ ● ▶ ◆ など）で代用してください。

## 使用ライブラリ（すべてCDN読み込み、cdnjs.cloudflare.com を使用）
- pdf-lib 1.17.1（PDFの読み込み・分割・新規PDF作成・メタデータ設定）
- PDF.js 3.11.174（PDFを画像としてレンダリング）※workerSrcも同バージョンで設定必須
- JSZip 3.10.1（ファイルをZIPにまとめる）
- FileSaver.js 2.0.5（ファイルのダウンロード保存）

## 画面構成（上から順に4ステップ＋実行ボタン＋ステータス）

### ヘッダー
- タイトル「PDF 分割・統合ツール」
- サブタイトル「完全ローカル処理。サーバーには送信されません。」

### ステップ1: ファイル選択
- PDFファイル選択用のinput（accept=".pdf"）
- <input type="file">は非表示（display: none）にし、カスタムデザインの<label>をボタンとして表示
- ボタンのテキスト: 「PDFファイルを選択」
- 読み込み後にファイル情報エリアを表示（例: 「ファイル名.pdf (全12ページ)」）
- ファイル情報は初期非表示、読み込み後に.visibleクラスで表示切替

### ステップ2: ページ範囲指定
- 入力補助エリア: 開始ページ・「〜」・終了ページのnumber入力＋「追加」ボタンを横並び（flexbox）
  - ボタンを押すと下のテキスト入力に「1-3」の形式で自動追記
  - 既に値がある場合はカンマ区切りで追加（例: 既存「1-3」に追加 → 「1-3, 7-9」）
  - 追加後、number入力はクリア
- 手動入力欄: テキストinput（placeholder: 「例: 1-3, 5, 7-9」）
  - inputイベントで全角→半角の自動変換を常時実行:
    - 全角数字（０-９）→ 半角数字
    - 全角ハイフン類（ー、－、—）→ 半角ハイフン(-)
    - 全角カンマ類（，、、）→ 半角カンマ(,)
  - 入力変更時にモードプレビューも更新する

### ステップ3: 出力モード選択（ラジオボタン・name="outputMode"）
3つの選択肢をカード形式で縦に並べる:

1. 「まとめて1ファイルに結合」 (value="merge") ※デフォルト選択
   - 説明: 「すべてのページを1つのファイルにまとめます」

2. 「グループごとに別ファイルで出力（ZIP）」 (value="split")
   - 説明: 「カンマ区切りの各グループを独立したファイルとして出力（ZIPでまとめてダウンロード）」

3. 「グループごとに別ファイルで出力（ZIPなし）」 (value="split-nozip")
   - 説明: 「各ファイルを0.8秒間隔で個別ダウンロード（セキュリティソフトでZIPがブロックされる場合）」

モードプレビュー: ラジオボタンの下に動的プレビューエリアを配置
- ページ範囲入力があるときだけ表示（.visibleクラスで切替）
- 黄色系の背景（#fff9e6）に左側オレンジのボーダー（border-left: 4px solid #ffa726）
- 選択モードに応じた具体例を表示:
  - merge: 「例: 「1-3, 7-9」→ すべてのページを含む1つのファイルが生成されます」
  - split: 「→ 「1-3のファイル」、「7-9のファイル」が生成され、ZIPでまとめてダウンロードされます」
  - split-nozip: 「→ 「1-3のファイル」、「7-9のファイル」が0.8秒間隔で個別にダウンロードされます」

### ステップ4: 出力形式選択（ラジオボタン・name="outputFormat"）
1. 「PDFファイル (.pdf)」 (value="pdf") ※デフォルト選択
   - 説明: 「選択ページだけの新しいPDFを生成」
2. 「画像: まとめてZIPで保存」 (value="image-zip")
   - 説明: 「各ページをJPEG画像にしてZIPでダウンロード」
3. 「画像: 1枚ずつ保存」 (value="image-single")
   - 説明: 「0.8秒間隔で1枚ずつダウンロード（ZIPがブロックされる場合の代替）」

### 実行ボタン
- テキスト: 「ファイルの分割・ダウンロード」
- ファイル未読み込み時はdisabled（グレーアウト、cursor: not-allowed）
- ホバーで浮き上がり（translateY(-2px) + box-shadow）、active時にscale(0.98)

### ステータス表示エリア
- 初期非表示、処理中に.visibleクラスで表示
- 4種類の状態表示:
  - success: 背景 #e8f5e9、ボーダー #4caf50、文字色 #2e7d32
  - error: 背景 #ffebee、ボーダー #f44336、文字色 #c62828
  - processing: 背景 #e3f2fd、ボーダー #2196f3、文字色 #1565c0
  - warning: 背景 #fff3e0、ボーダー #ff9800、文字色 #e65100
- fadeInアニメーション付き（opacity 0→1、translateY -10px→0、0.3秒）
- innerHTMLで表示（HTMLタグ使用可能にする）

## UIデザインの詳細

### ステップ番号バッジ
各ステップのタイトルに丸い番号バッジを付ける:
- 紺→紫のグラデーション背景（#1a237e → #4a148c）、白文字
- 28x28pxの円形、inline-flex中央寄せ、font-size: 14px

### ラジオボタンのカード型UI
- 各選択肢を白背景のカードとして表示（padding: 16px、border-radius: 10px）
- ボーダー: 通常 2px solid #e0e0e0、ホバー時 border-color: #4a148c、選択時 border-color: #4a148c + 背景: #f3e5f5
- .selectedクラスで選択状態を管理
- JavaScriptのクリックイベントで:
  1. クリックされたカード内のradioをcheckedにする
  2. 同じname属性の全カードから.selectedを除去
  3. クリックされたカードに.selectedを付与
  4. name="outputMode"の場合はモードプレビューも更新
- ラベル部分は太字（font-weight: 600）、説明文はfont-size: 13px、color: #666でラベル下に表示

## 技術的な処理の流れ

### グローバル変数
- pdfData: ArrayBuffer（読み込んだPDFのバイトデータ）
- pdfDoc: pdf-libのPDFDocumentオブジェクト
- totalPages: 総ページ数
- fileName: 拡張子(.pdf)を除いたファイル名

### ファイル読み込み処理
1. file.arrayBuffer()でArrayBufferとして読み込み
2. pdf-libのPDFDocument.load(arrayBuffer)でPDFドキュメントを作成しページ数取得
3. number入力のmax属性にtotalPagesを設定
4. 実行ボタンのdisabledを解除（false）
5. エラー時: console.errorで出力し、ステータスに「暗号化されたPDFの可能性があります」を表示

### ページ範囲パース関数（2種類必要）

parsePageGroups(input, maxPages) -- 分割モード用
- カンマ区切りで分割し、各グループを配列の配列で返す
- 例: "1-3, 7-9" → [[0,1,2], [6,7,8]]（0始まりインデックス）
- 各グループ内でSetで重複排除し、ソートして配列化
- 空のグループ（無効な入力）はスキップ

parsePageRangeFlat(input, maxPages) -- 結合モード用
- 全グループをまとめて1つのSetに入れ、ソートして1つの配列で返す

共通仕様:
- ページ番号は1始まりでユーザー入力 → 内部では -1 して0始まりインデックスに変換
- 範囲指定（"3-7"）と単一指定（"5"）の両方に対応
- 範囲外のページ番号はMath.max(1, ...)とMath.min(maxPages, ...)でクランプ

### PDF出力: 結合モード (generateMergedPDF)
1. PDFLib.PDFDocument.create()で新規PDF作成
2. PDFLib.PDFDocument.load(pdfData)で元PDFを読み込み
3. メタデータ設定（Windowsセキュリティ警告の軽減のため）:
   - setTitle(`${fileName}_split`)
   - setProducer('PDF Split Tool')
   - setCreator('Local PDF Tool')
   - setCreationDate(new Date())
   - setModificationDate(new Date())
4. newPdf.copyPages(sourcePdf, pages)で指定ページをコピー
5. copiedPages.forEach(page => newPdf.addPage(page))
6. newPdf.save()でバイト配列取得 → Blob作成（type: 'application/pdf'）
7. saveAs(blob, `${fileName}_split.pdf`)でダウンロード
8. 完了メッセージに<small>タグでWindowsブロック解除手順を追記

### PDF出力: 分割モード・ZIP (generateSplitPDFs)
1. JSZipインスタンスを作成
2. 各グループごとにループ:
   - PDFDocument.create() + メタデータ設定
   - copyPages → addPage → save()
   - ファイル名の決定:
     - 単一ページ: `${fileName}_p5.pdf`
     - 範囲: `${fileName}_p1-3.pdf`（先頭ページと末尾ページから生成）
   - zip.file(filename, pdfBytes)でZIPに追加
3. zip.generateAsync({type: 'blob'})でZIP Blob生成
4. saveAs(zipBlob, `${fileName}_split.zip`)

### PDF出力: 分割モード・ZIPなし (generateSplitPDFsNoZip)
1. PDFLib.PDFDocument.load(pdfData)で元PDFを1回読み込み
2. 各グループごとにループ:
   - PDFDocument.create() + メタデータ設定
   - copyPages → addPage → save()
   - Blob作成 → saveAs()で個別ダウンロード
   - ファイル名はZIP版と同じルール
   - 進捗をステータスに表示（「PDFファイル生成中... (2/3)」）
3. ファイル間に0.8秒のdelay: await new Promise(resolve => setTimeout(resolve, 800))
4. 最後のファイルの後はdelayなし（if (i < pageGroups.length - 1) で判定）

### 画像レンダリング共通関数 (renderPageToImage)
1. pdfjsLib.getDocument({data: pdfData}).promiseでPDF読み込み
2. pdf.getPage(pageNum + 1)でページ取得（PDF.jsは1始まり）
3. page.getViewport({scale: 2.0})でviewport作成（高解像度）
4. canvas要素を動的生成（document.createElement('canvas')）
5. canvas.width/heightにviewportのサイズを設定
6. page.render({canvasContext, viewport}).promiseでレンダリング
7. canvas.toBlob(callback, 'image/jpeg', 0.9)でJPEGに変換
8. Promiseでblobを返す

### 画像ZIP出力 (generateImagesZIP)
- 引数: pages（フラットな配列）, mode（'merge' or 'split'）
- 結合モード: imagesフォルダにpage_001.jpg形式で格納
- 分割モード: parsePageGroupsを再呼び出しし、グループごとにサブフォルダ（p1-3/, p7-9/）を作成
- ZIPファイル名: `${fileName}_images.zip`

### 画像1枚ずつ出力 (generateImagesSingle)
- 各画像をsaveAsで個別ダウンロード
- ファイル名: `${fileName}_page_001.jpg`形式（元のページ番号ベース）
- 0.8秒間隔（最後のファイル後はdelayなし）

### メイン実行処理 (executeBtnのclickイベント)
- バリデーション: pdfData未設定 or ページ範囲未入力でエラー表示して終了
- outputModeとoutputFormatの値を取得
- 以下の組み合わせで分岐:
  - pdf + merge → generateMergedPDF
  - pdf + split → generateSplitPDFs
  - pdf + split-nozip → generateSplitPDFsNoZip
  - image-zip + (any mode) → generateImagesZIP
  - image-single + (any mode) → generateImagesSingle
- try-catch-finallyでエラーハンドリング
  - catch: console.error + ステータスにエラー表示
  - finally: executeBtn.disabled = false（ボタンを再有効化）

## デザイン要件
- CSS変数は不使用（直接カラーコード指定）
- 背景グラデーション: linear-gradient(135deg, #1a237e 0%, #4a148c 40%, #c2185b 100%)
- bodyのpadding: 40px 20px、line-height: 1.6
- メインコンテナ: 白背景、max-width: 800px、border-radius: 20px、box-shadow: 0 20px 60px rgba(0,0,0,0.3)、padding: 40px
- 入力グループ: background: #f8f9fa、border-radius: 12px、border: 2px solid #e0e0e0
- ボタン類のグラデーション: #1a237e → #4a148c
- 実行ボタン: #1a237e → #c2185b（紺→ピンク）、font-size: 18px、font-weight: 700
- inputのフォーカス時: outline: none、border-color: #4a148c
- モバイル対応: @media (max-width: 600px)でコンテナのpadding縮小、入力補助を縦並びに変更
- CSSリセット: * { margin: 0; padding: 0; box-sizing: border-box; }
- フォントファミリー: 'Helvetica Neue', 'Hiragino Sans', 'Hiragino Kaku Gothic ProN', 'Yu Gothic', sans-serif

## 注意事項
- 絵文字（Unicode Emoji）は一切使用しないこと。テキスト記号で代用する。
- PDF.jsのworkerSrcを必ず設定すること（CDNのpdf.worker.min.jsで、本体と同じバージョン）
- 全角入力の自動変換はinputイベントで常時実行すること
- エラーハンドリング: 暗号化PDFなど読み込み失敗時のメッセージを必ず表示
- デバッグログはconsole.errorで出力
- 生成PDFにはメタデータ（Title, Producer, Creator, CreationDate, ModificationDate）を必ず設定する
- ステータスの完了メッセージには「Windowsでブロックされた場合: ファイルを右クリック → プロパティ → ブロックの解除にチェック → OK」の手順を含める
- 印刷用CSSは不要（省略してよい）

すべてを1つのHTMLファイルにまとめて、コピペでそのまま動く状態で出力してください。
````

---

## 補足

- **カスタマイズのヒント**: プロンプト末尾に「タイトルを○○に変更」「カラーを○○系に変更」などを追記すれば、デザインを自由に変えられます。
- **画像解像度の変更**: プロンプト内の `scale: 2.0` の値を上げると高画質（ファイルサイズ増）、下げると軽量になります。
- **対応ブラウザ**: Chrome, Edge, Firefox の最新版で動作します（IE非対応）。
- **絵文字を使いたい場合**: プロンプト冒頭の「絵文字は一切使用しないでください」の指示を削除し、各項目に好みの絵文字を追記してください。
