いいですね！AWS Lambda に CodePipeline で自動デプロイする「**超シンプルなサンプル**」を紹介します。

---

## 🎯 目標

GitHub に Lambda 関数のコードを push → CodePipeline が自動で ZIP → Lambda にデプロイ

---

## 📝 前提条件

* AWS アカウント
* GitHub リポジトリ（例：`lambda-auto-deploy`）
* Lambda 関数を事前に作成（例：`my-hello-lambda`）
* IAM ロール：

  * CodePipeline 実行用ロール
  * CodeBuild 実行用ロール（Lambdaに書き込む権限が必要）

---

## 📁 GitHub リポジトリ構成（例）

```
lambda-auto-deploy/
├── index.js
└── buildspec.yml
```

---

### ✨ `index.js`

```js
exports.handler = async (event) => {
    return {
        statusCode: 200,
        body: "Hello from CodePipeline + Lambda!"
    };
};
```

---

### ⚙️ `buildspec.yml`

```yaml
version: 0.2

phases:
  install:
    commands:
      - echo "Nothing to install"
  build:
    commands:
      - zip function.zip index.js

artifacts:
  files:
    - function.zip
```

---

## 🧪 Lambda デプロイ構成（CodePipeline）

1. **ソースステージ**

   * ソース：GitHub を選択（アクセストークン連携）

2. **ビルドステージ（CodeBuild）**

   * 環境：Amazon Linux（Node.js ランタイム不要）
   * `buildspec.yml` を使って ZIP ファイル作成

3. **デプロイステージ**

   * デプロイ先：AWS Lambda
   * 関数名：`my-hello-lambda`
   * アーティファクト名：`function.zip`

---

## 🛡 IAM ロールのポイント

**CodeBuild ロール**には以下のポリシーが必要：

```json
{
  "Effect": "Allow",
  "Action": [
    "lambda:UpdateFunctionCode"
  ],
  "Resource": "arn:aws:lambda:<region>:<account-id>:function:my-hello-lambda"
}
```

---

## ✅ 完成するとこうなる

1. `index.js` を GitHub に push
2. CodePipeline が自動で動く
3. ZIP を作成 → Lambda に反映
4. Lambda が即時更新！

---

## 📣 おまけ：テスト方法

Lambdaコンソールで「テスト」→ `Hello from CodePipeline + Lambda!` と返ってくればOK。

---

## 📌 カスタムしたい？

* Node.js 以外（Python, Go など）にも対応可能
* 複数ファイル、依存ありの場合 → `npm install` や `pip install` も `buildspec.yml` に追加すればOK

---

必要であれば、\*\*CloudFormationやCDKでIaC（コードによる自動構築）\*\*もできます。
「GUIで作りたい」「IaCで管理したい」などご希望あれば、それ向けに用意します！
