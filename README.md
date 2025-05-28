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
      - aws lambda update-function-code --function-name my-hello-lambda --zip-file fileb://function.zip --region ap-northeast-1

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

AWS Lambda 関数（例：`my-hello-lambda`）は、以下の手順で **マネジメントコンソールから簡単に作成**できます。

---

## 🧭 手順：Lambda 関数の作成（Node.js 編）

### ① AWS コンソールにログイン

→ [https://console.aws.amazon.com/lambda](https://console.aws.amazon.com/lambda)

---

### ② 「関数の作成」ボタンをクリック

---

### ③ 以下のように入力・選択

| 項目    | 内容例                                                                   |
| ----- | --------------------------------------------------------------------- |
| 関数名   | `my-hello-lambda`                                                     |
| ランタイム | `Node.js 18.x`                                                        |
| 実行ロール | 「既存のロールを使用」or「新規作成」<br>→ とりあえず `lambda-basic-execution` ポリシーがついていればOK |

※後でCodePipelineから更新するので「とりあえず関数を作るだけ」でOKです。

---

### ④ 作成ボタンをクリック

---

## ✅ 作成後に確認すること

### ✅ コードが不要な場合（GitHubから自動で上書きされる）

GitHub に push → CodeBuild で `function.zip` 作成 → Lambda に上書き

### ✅ 関数の ARN を控える

後で `CodePipeline` または `CodeBuild` から更新する際に必要になる場合があります。

```
arn:aws:lambda:ap-northeast-1:123456789012:function:my-hello-lambda
```

---

## 🔒 実行ロールの注意点

Lambda が CloudWatch にログを出力できるように、**最低限以下のポリシー**がロールについていればOK：

```json
{
  "Effect": "Allow",
  "Action": ["logs:CreateLogGroup", "logs:CreateLogStream", "logs:PutLogEvents"],
  "Resource": "*"
}
```




