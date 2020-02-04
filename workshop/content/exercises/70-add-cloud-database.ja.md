この演習では、前にプロビジョニングしたAWS Relational Database Service（``RDS``）インスタンスを操作し、それを使用するようにアプリケーションを構成します。Amazon RDSを使用すると、クラウドでのMySQLデプロイメントのセットアップ、運用、およびスケーリングが簡単になります。

RDSインスタンスのステータスを確認します:

```execute
svcat get instances
```


 - Important! 続行する前に、インスタンスのステータスが``Ready`` になるまで待ちます。


# Bind to the database service 

最初にデータベースのアクセス資格情報（ホスト、ユーザー名、パスワード...）を取得して、アプリケーションで新しいデータベースサービス（``mysql``と呼ばれる）にバインドします。

``bind``コマンドを使用して、``service catalogue``経由でアクセス認証情報を取得します：

```execute
svcat bind mysql --name mysql-binding --secret-name mysql-secret
```


 - バインディングは``mysql-binding``となり、Kubernetesシークレットは``mysql-secret``となります。シークレットには、RDSインスタンスの接続の詳細と資格情報（ホスト、ユーザー名、パスワードなど）が含まれます。
 - Kubernetesシークレットオブジェクトを使用すると、パスワード、OAuthトークン、sshキーなどの機密情報を保存および管理できます。この情報をシークレットに入れる方が、コンテナにそのまま入れるよりも安全で柔軟です。


データベースのバインドを確認します:

```execute
svcat get bindings
```

シークレットを確認します:

```execute
oc describe secret mysql-secret 
```

次の出力が表示されます:

```
Data
====
DB_NAME:           4 bytes
ENDPOINT_ADDRESS:  61 bytes
MASTER_PASSWORD:   15 bytes
MASTER_USERNAME:   4 bytes
PORT:              5 bytes
```

 - エラー：``not found``が表示された場合シークレットはまだ作成されていないことに注意してください。これは、RDSインスタンスがプロビジョニングを完了していないか、バインディングがまだ作成されていないことを意味します。



# Verify the database is running 

RDSインスタンスが稼働中で到達可能であることを確認しましょう。

ヘルプスクリプトを使用して、シークレットから端末のシェル環境に値を抽出し、データベースへの接続をテストします。

```execute
extract-secret mysql-secret
```

後で使用するために、このデータをシェルの環境にインポートします。: 

```execute
eval `extract-secret mysql-secret`
```

 - このコマンドが出力を表示しない場合、成功しています。 
 - このコマンドが``not foun``を返す場合、データベースとそのバインディングが``ready``となりかつ、そのバインディングからシークレットが作成されるまで待機します。 

次に、データベースにアクセスして、``vote``データベースの内容を確認します：

```execute
mysql -h $ENDPOINT_ADDRESS -P $PORT -u $MASTER_USERNAME -p"$MASTER_PASSWORD" -D $DB_NAME -e 'show databases;'
```
出力には``vote``データベースが含まれている必要があります。そうでない場合、またはエラーがある場合は、データベースの準備が整うまで待ちます。

次に、データベースが空かどうかを確認します。

  - アプリケーションがまだ初期化していない場合は、データベースは``空``のはずです。

```execute
mysql -h $ENDPOINT_ADDRESS -P $PORT -u $MASTER_USERNAME -p"$MASTER_PASSWORD" -D $DB_NAME -e 'show tables;'
```
データベースに接続するようにアプリケーションを構成して起動した後にのみ、データベースにコンテンツが存在します。

```
+----------------+
| Tables_in_vote |
+----------------+
| option         |
| poll           |
+----------------+
```


<!--
# Point the application to the database 

If not already done in the previous exercise, the application needs to be configured to use a ``mysql`` database instead of the `built-in` database.  Add this setting to the application.

To do this, add the environment variable ``DB_TYPE`` into the application using the following command:

```execute
oc rollout pause dc vote-app 
oc set env dc vote-app \
   DB_TYPE=mysql
oc rollout resume dc vote-app
```

Note, that the above ``oc set env`` command would normally cause a re-deployment of the application.  In this case ``oc rollout pause dc vote-app`` is used to stop this from happening, since we are not ready to restart it just yet. 

-->

## Configure the application to use the database

次に、データベース資格情報をアプリケーションに注入して、アプリケーションをデータベースに接続します。

これを行うには、シークレットから``deployment``へ資格情報をインポートすることでアプリケーションに資格情報を追加します:

```execute
oc set env dc/vote-app \
    --from=secret/mysql-secret \
    DB_TYPE=mysql  
```
<!--
oc set env --from=secret/mysql-secret dc/vote-app 
-->

前のコマンドは、シークレット：``mysql-secret``から値を取得し、それらをdeployment configへ追加し、アプリケーションを再デプロイします。

環境変数が適切に設定されていることを確認します：

```execute
oc set env dc/vote-app --list
```

上記のコマンドは、データベースへの接続に必要な値が表示される必要があります。

アプリケーションが再デプロイされたら、アプリケーションによってデータベースが初期化されたことを確認します：

```execute
mysql -h $ENDPOINT_ADDRESS -P $PORT -u $MASTER_USERNAME -p"$MASTER_PASSWORD" -D $DB_NAME -e 'show tables;'
```
テーブル（``poll``および``option``）が作成されているはずです。


## Test the application 

ヘルプスクリプトを使用して、アプリケーションにランダムな投票をいくつか投稿します:

```execute 
test-vote-app http://vote-app-%project_namespace%.%cluster_subdomain%/vote.html
```

アプリケーションを使用した後、データベースのテーブルで投票を確認します: 

```execute
mysql -h $ENDPOINT_ADDRESS -P $PORT -u $MASTER_USERNAME -p"$MASTER_PASSWORD" -D $DB_NAME -e 'select * from `option`;'
```

アプリケーションが再び実行されたら、それがまだ機能していることを確認します。

```execute 
curl -s http://vote-app-%project_namespace%.%cluster_subdomain%/ | grep "<title>"
```

ブラウザでアプリケーションをテストします:

[Open the Vote Application](http://vote-app-%project_namespace%.%cluster_subdomain%/)

 - ``隣の人もあなたのアプリケーションへアクセスして投票できるはずです``

アプリケーションを使用して投票を追加した後、データベースで投票を確認します: 

```execute
mysql -h $ENDPOINT_ADDRESS -P $PORT -u $MASTER_USERNAME -p"$MASTER_PASSWORD" -D $DB_NAME -e 'select * from `option`;'
```

---
curlを使用してアプリケーションをテストできます。

このヘルプスクリプトを使用して、アプリケーションにランダムな投票をいくつか投稿してください:

```execute 
test-vote-app http://vote-app-%project_namespace%.%cluster_subdomain%/vote.html
```

<!--
```execute 
for i in {1..20}
do
   echo Casting vote nr. $i
   curl -s -X POST http://vote-app-%project_namespace%.%cluster_subdomain%/vote.html -d "vote=`expr $(($RANDOM % 9)) + 1`" >/dev/null
done
```
-->

結果を表示するには、次のコマンドを使用します。すべての投票オプションの合計が表示されます。:

```execute 
curl -s http://vote-app-%project_namespace%.%cluster_subdomain%/results.html | grep "data: \["
```

次のように、すべての票が表示されます。: 

```
  data: [ "3",  "3",  "2",  "0",  "1",  "5",  "1",  "3",  "2", ],

```

または、ブラウザで結果ページを表示します:

[View Results page](http://vote-app-%project_namespace%.%cluster_subdomain%/results.html)


---
これでこの演習は終わりです。

この演習では、コンテナにRDSデータベースをプロビジョニングし、アプリケーションをそれに接続しました。


[indexへ戻る](../index-aws.ja.md)

