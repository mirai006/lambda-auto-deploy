GitHub のリポジトリ（この場合 `mirai006/lambda-auto-deploy`）にファイルを追加する方法は大きく分けて2つあります：

---

## ✅ 方法①：GitHub Web画面から直接ファイルを追加する

### 手順：

1. リポジトリのページで「**Add file**」ボタンをクリック
   👉（`Code` タブの右側あたり）

2. 「**Create new file**」を選択

3. 例えば：

   * `index.js` と入力してファイルを作成
   * 中身を以下のように書く：

     ```js
     exports.handler = async (event) => {
         return {
             statusCode: 200,
             body: "Hello from Lambda!"
         };
     };
     ```

4. 下にスクロールして「**Commit new file**」で保存

> 同様に `buildspec.yml` も同じように追加できます。

---

## ✅ 方法②：ローカルPCで作成して `git push`

### 手順（ターミナル）：

```bash
# クローン
git clone https://github.com/mirai006/lambda-auto-deploy.git
cd lambda-auto-deploy

# ファイル作成（index.js など）
echo 'exports.handler = async (event) => {
  return {
    statusCode: 200,
    body: "Hello from Lambda!"
  };
};' > index.js

# buildspec.yml も作る
echo 'version: 0.2

phases:
  install:
    commands:
      - echo "Nothing to install"
  build:
    commands:
      - zip function.zip index.js

artifacts:
  files:
    - function.zip' > buildspec.yml

# GitHubに反映
git add .
git commit -m "add lambda and buildspec"
git push origin main
```

---

## 🌟おすすめ

はじめてなら **GitHub Webから追加する方法**（方法①）が簡単でおすすめです！

---

### ファイルを2つ追加しましょう：

* `index.js`
* `buildspec.yml`

追加できたら、「CodePipelineでデプロイ先にLambda関数を指定」して連携するステップに進めますよ！
希望があれば一緒にその設定も説明します。
