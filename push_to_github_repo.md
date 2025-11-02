# GitHubでのバージョン管理を始めるための備忘録 (By HitakaK)

## 1-SSH鍵の登録
まずローカルでSSH鍵の作成を行います。  
`$ ssh-keygen -t rsa -b 4096`  
鍵のパス/名前、パスワードを設定して作成を完了します。  
- `-t`: 鍵のタイプ (RSA暗号など)
- `-b`: ビット数 (長さ？)
  
次に、GitHubへの登録を行います。
公開鍵の内容をクリップボードにコピーします。  
例) `$ cat id_rsa.pub`  
GitHubの右上のプロフィールアイコンをクリック → 「Settings」 → 「SSH and GPG keys」 → 「New SSH key」にアクセスし、鍵の名前 (自分がわかりやすいように)と公開鍵の内容 (クリップボードにコピーしたやつ)を入力します。  
Key typeは「Authentication Key」のままでOKです。  

毎回SSHの長いコマンドを入力するのは大変なのでSSH-Agentを利用します。  
`~/.ssh/config`を編集します。  
```config
Host github
	HostName github.com
	IdentityFile ~/.ssh/github
	User git
```
SSH接続できるようになったことを確認します。  
`$ ssh -T git@github.com`  
`Hi <<名前>>! You've successfully authenticated, ~~~`と出てきたら完了です。  

## 2-ローカルリポジトリの作成
バージョン管理したいディレクトリに移動し、`$ git init`を実行します。これでこのディレクトリがGitリポジトリになります。  
そのディレクトリに何もファイルがない場合、とりあえず`README.md`だけ作ってみましょう。  
`$ echo "# README" > README.md`  
リポジトリの内容を保存して最初の状態とします。  
`$ git add .`  
登録されたファイル名などは`$ git status`で確認できます。  
登録した内容をコミットします。  
`$ git commit -m 'first commit'`  
これで準備はOKです！  

## 3-リモートリポジトリの作成
ローカルのバージョン管理情報を別のサーバに保存しておくと安心です。  
GitHubにアクセスします。  
右上のプロフィールアイコンをクリック → 「Repositories」 → 「New」をクリックします。  
Repository nameを入力し、Choose visibility (レポジトリの公開/非公開)を選択します。  
それ以外はそのままで「Create repository」をクリックします。  

## 4-ローカルの内容をリモートにプッシュ(保存)
GitHubで空のリポジトリの「Code」ページの最下部にある「…or push an existing repository from the command line」に従って作業を進めます。  
`$ git remote add origin git@github.com:<<ユーザ名>>/<<リポジトリ名>>.git`  
`$ git branch -M main`  
`$ git push -u origin main`  
自分で入力したりせず大人しくコピペしましょう。  
GitHubのページを再読み込みするとローカルリポジトリの内容がきちんと反映されていることが確認できます。
