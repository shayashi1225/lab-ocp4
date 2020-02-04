この演習では、プロビジョニングされたリソースをクリーンアップして削除します。

## Remove the RDS Instance 

本番クラウドベースのデータベースのプロビジョニングを解除します。

まず、バインディングを削除する必要があります。これは、資格情報とシークレットを削除することを意味します。これは、サービスカタログを介して行うのが最適です： 

```execute
svcat unbind --name mysql-binding
```

...そして、MySQLサービスのプロビジョニングを解除します:

```execute
svcat deprovision mysql 
```

MySQLインスタンスは削除され、約5〜10分かかります。

ここで進行状況を確認します：

```execute
svcat get instances 
```

## Remove the application and the container database

次に、コンテナベースのデータベースを削除します 

```execute 
oc delete all --selector=app=db 
```

`selector`オプションと`app=db`ラベルの名前を使用して、特定のアプリケーションを簡単に削除できるます。

アプリケーションを削除します：

```execute 
oc delete all --selector=app=vote-app  
```

アプリケーションの一部のみを削除するためには、 `selector`を使用します。
念のため、プロジェクトからすべてを削除します：


```execute 
oc delete all --all & 
```

## Optionally, delete your GitHub repository ``flask-vote-app``

これを行うには、GitHub.comにログインして``flask-vote-app``リポジトリを見つけ、リポジトリをクリックし``Settings``、ページの下部にある``Delete this Repository``ボタンをクリックします。

また、 https://github.com/settings/tokens　へアクセスして、　``hub for...`` というトークンを削除します.  

## Remove the local GitHub personal access token  
このワークショップの開始時に、GitHubリポジトリをフォークしました。リポジトリにさらにアクセスできるようにするために、トークンが生成されました。次に、このトークンを削除します:

```execute 
rm -f ~/.gitconfig ~/.config/hub 
```

---
これでこの演習は終わりです。


[indexへ戻る](../index-aws.ja.md)