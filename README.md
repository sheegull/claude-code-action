# Claude Maxプラン料金内でClaude Code GitHub Actionsを使うためのガイドまとめ🎉

## 📋 必要なもの

- **Claude Maxサブスクリプション**（月額$100または$200）
- **GitHubアカウント**（リポジトリの管理者権限必須）
- **Claude Code**がローカルにインストール済み

## 🔧 事前準備：リポジトリのフォーク

### フォークが必要なリポジトリ

1. **claude-code-action**
    - https://github.com/Akira-Papa/claude-code-action
2. **claude-code-base-action**
    - https://github.com/Akira-Papa/claude-code-base-action

### フォーク後の重要な変更

フォークしてローカルにクローンしたら、以下の変更を行う：

**変更前：**
action.yml
```yaml
- name: Run Claude Code
  id: claude-code
  if: steps.prepare.outputs.contains_trigger == 'true'
  uses: Akira-Papa/claude-code-base-action@main
```

**変更後：**

```yaml
- name: Run Claude Code
  id: claude-code
  if: steps.prepare.outputs.contains_trigger == 'true'
  uses: あなたのGitHubアカウント名/claude-code-base-action@main
```

---

## 📝 セットアップ手順

### Step 1: Claude Codeでログイン確認 🔑

1. **ターミナルでClaude Codeを起動**

    ```bash
    claude

    ```

2. **ログイン状態を確認**

    ```
    /status

    ```

    以下の表示を確認：

    ```
    Account: Login Method: Claude Max Account (5x)

    ```

3. **ログインしていない場合**

    ```
    /login

    ```

    を実行してClaude Maxアカウントでログイン


---

### Step 2: OAuth認証情報の取得 📄

**認証ファイルから情報を取得：**

```bash
cat ~/.claude/.credentials.json

```

**・macの人はキーチェーンでclaudeで検索し、パスワードをコピーすると、**access_token/refresh_token/expires_atがコピーされる

![image.png](attachment:f863f631-fdc5-4c99-9477-81d6dc0ad3b4:image.png)

**以下の3つの値をメモ：**

- `access_token`
- `refresh_token`
- `expires_at`

---

### Step 3: GitHub Appのインストール 📱

1. **Claude Code内でコマンド実行**

    ```
    /install-github-app

    ```

2. **ブラウザが開いたら**
    - 対象リポジトリを選択
    - **重要**：APIキーを求められたら、一旦APIキーを設定する（後で削除）
3. **対象リポジトリへのアクセスを許可**

**代替方法：**
直接ブラウザでアクセス

```
https://github.com/apps/claude

```

---

### Step 4: GitHubシークレットの設定 🔐

**対象リポジトリのSettings > Secrets and variables > Actionsで設定：**

| シークレット名 | 値 |
| --- | --- |
| `CLAUDE_ACCESS_TOKEN` | credentials.jsonの`access_token` |
| `CLAUDE_REFRESH_TOKEN` | credentials.jsonの`refresh_token` |
| `CLAUDE_EXPIRES_AT` | credentials.jsonの`expires_at` |

**⚠️ 重要な作業：**
Claude Appのインストールが確認できたら、対象リポジトリのSettings > Secrets and variables > Actionsから`ANTHROPIC_API_KEY`を削除する

---

### Step 5: ワークフローファイルの作成 📄

対象リポジトリに`.github/workflows/claude.yml`を作成：

```yaml
name: Claude Code
on:
    issue_comment:
        types: [created]
    pull_request_review_comment:
        types: [created]
    issues:
        types: [opened, assigned]
    pull_request_review:
        types: [submitted]

jobs:
    claude:
        if: |
            (github.event_name == 'issue_comment' && contains(github.event.comment.body, '@claude')) ||
            (github.event_name == 'pull_request_review_comment' && contains(github.event.comment.body, '@claude')) ||
            (github.event_name == 'pull_request_review' && contains(github.event.review.body, '@claude')) ||
            (github.event_name == 'issues' && (contains(github.event.issue.body, '@claude') || contains(github.event.issue.title, '@claude')))

        runs-on: ubuntu-latest

        permissions:
            contents: write
            pull-requests: write
            issues: write
            id-token: write

        steps:
            - name: Checkout repository
              uses: actions/checkout@v4
              with:
                  fetch-depth: 1

            - name: Run Claude Code
              id: claude
              uses: あなたのGitHubアカウント名/claude-code-action@main
              with:
                  use_oauth: 'true'
                  claude_access_token: ${{ secrets.CLAUDE_ACCESS_TOKEN }}
                  claude_refresh_token: ${{ secrets.CLAUDE_REFRESH_TOKEN }}
                  claude_expires_at: ${{ secrets.CLAUDE_EXPIRES_AT }}

```

**注意：** `uses: あなたのGitHubアカウント名/claude-code-action@main` の部分を実際のアカウント名に変更

---

### Step 6: ワークフローファイルをmainブランチにマージ 🔀

1. **新しいブランチを作成**

    ```bash
    git checkout -b add-claude-workflow

    ```

2. **ファイルをコミット＆プッシュ**

    ```bash
    git add .github/workflows/claude.yml
    git commit -m "Add Claude GitHub Action workflow"
    git push origin add-claude-workflow

    ```

3. **PRを作成してmainブランチへマージ**

---

### Step 7: 動作確認 ✅

GitHubリポジトリのIssueやPRで`@claude`メンションして呼び出せたら完了！

---

## 🚀 使用例

### 基本的な使い方

### PRやIssueでの質問

```
@claude このコードの改善点を教えてください

```

### コード修正依頼

```
@claude エラーハンドリングを追加してください

```

### コードレビュー

```
@claude このPRをレビューしてください

```

---

## 💡 ポイント

1. **フォークしたリポジトリの使用が必須**
    - OAuth認証をサポートしているのはフォーク版のみ
2. **APIキーの一時設定と削除**
    - GitHub Appインストール時に一旦設定が必要
    - OAuth設定後は削除して安全性を確保
3. **mainブランチへのマージが必要**
    - GitHub ActionsはデフォルトでmainブランチのワークフローファイルPMを読み込む
4. こちら対象のリポジトリは、他の人から@claudeメンションできないように、privateに設定しておきましょう💡

これで、追加のAPI料金なしでClaude MaxをGitHub Actionsで活用できます！🎊
