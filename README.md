# bridge-quiz

総合学習教室ブリッジ（滋賀県彦根市稲枝町）の学習診断クイズ。

公開URL: https://rikimat.github.io/bridge-quiz/

中身は React + Babel standalone のシングル `index.html`。8問×2種類の診断（学びの土台診断 / 高校受験 準備力診断）を実装。

---

## 構成

```
bridge-quiz/
├── index.html                  # 本体（これ1つでサイト全部）
├── .github/
│   └── workflows/
│       └── deploy.yml          # main への push で自動デプロイ
└── README.md                   # このファイル
```

## デプロイの仕組み

`main` ブランチに push されると、GitHub Actions（`.github/workflows/deploy.yml`）が自動で走り、GitHub Pages に反映されます。

* `verify` ジョブ: `index.html` の存在とサイズを確認（壊れた空ファイルが上がってもデプロイされません）
* `build` ジョブ: そのまま静的サイトとしてアップロード
* `deploy` ジョブ: GitHub Pages に公開

手動再デプロイは [Actions タブ](https://github.com/RikimaT/bridge-quiz/actions) → "Deploy bridge-quiz to GitHub Pages" → "Run workflow" から。

## アクセス解析

GoatCounter（プライバシー重視・Cookie レス・無料）を埋め込んでいます。

ダッシュボード: https://bridge-quiz.goatcounter.com/

取れる情報: 訪問数・国/地域・端末種別・流入元（リファラ）・ページ別アクセス（クイズ種別／設問番号／結果タイプ）。

**個人特定はできません**（IP アドレスを保存しない仕様のため）。

### 初回セットアップ

1. https://www.goatcounter.com/signup でアカウント作成
2. サイトコード `bridge-quiz` を確保（`bridge-quiz.goatcounter.com` になる）
3. メール `rikima81@gmail.com` で登録
4. 完了。`index.html` は既に対応済みなので、登録した瞬間から計測が始まります

### 別の解析サービスに変えたいとき

`index.html` の "アクセス解析: GoatCounter" コメント以下のスクリプトを差し替えるだけです。

---

## 復旧手順（また 404 になったとき）

### Step 0: まず Actions の状態を見る

1. https://github.com/RikimaT/bridge-quiz/actions
2. 最新ワークフローが赤（失敗）なら、クリックして原因確認

### Step 1: index.html が壊れていないか確認

```bash
gh api repos/RikimaT/bridge-quiz/contents/index.html | jq -r '.size'
```

数字が 20000 以上なら OK。`null` や極端に小さい値ならファイルが壊れています。

### Step 2: 壊れていたら手元から再 push

```bash
cd ~/bridge-quiz
git pull
# 必要なら index.html を編集
git add index.html
git commit -m "fix: restore index.html"
git push
```

push されると Actions が自動で走って復旧します。

### Step 3: Pages 設定が消えていないか確認

```bash
gh api repos/RikimaT/bridge-quiz/pages
```

* 200 が返る ＆ `"build_type": "workflow"` ならOK
* 404 なら Pages が無効化されている。以下で再有効化:

```bash
gh api repos/RikimaT/bridge-quiz/pages -X POST \
  -F build_type=workflow
```

### Step 4: それでも直らないとき

GitHub Pages 自体に障害が起きている可能性があります。
https://www.githubstatus.com/ を確認してください。

---

## よくある壊れ方と対策

| 症状 | 原因の可能性 | 対策 |
|---|---|---|
| 急に 404 | GitHub Desktop が裏で空コミットを push | Actions ログを確認、必要なら revert |
| 数日後にまた 404 | レガシー Pages 設定の不安定さ | この README の Step 3 で `build_type=workflow` を確認 |
| Actions が失敗 | `index.html` の構文ミス | ローカル（Chrome）で開いて表示確認してから push |
| 解析が動かない | GoatCounter 未登録 | 上記「初回セットアップ」手順 |

---

## ローカルで確認するには

```bash
cd ~/bridge-quiz
open index.html
```

Chrome / Safari で直接開けます。
