この演習では、データベースを``コンテナの中で``起動し、デフォルトの組み込みデータベースの代わりにこのデータベースを使用するようにアプリケーションを構成します。


まず、アプリケーションのレプリカが1つだけ実行されていることを確認します。:

```execute
oc scale dc vote-app --replicas=1
```

# Launch the database in a container

MySQLデータベースを起動し、アプリケーションをそれに接続します。MySQLは、Structured Query Language（SQL）を使用する、無料で利用可能なオープンソースのリレーショナルデータベース管理システム（RDBMS）です。

これを実行してMySQLコンテナーを開始します。


```execute
oc new-app --name db mysql:5.7 \
   -e MYSQL_USER=user \
   -e MYSQL_PASSWORD=password \
   -e MYSQL_DATABASE=vote 
```

MySQLは、上記のコマンドパラメータで指定した``user``と``password``資格情報を持つ``vote``と呼ばれるデータベースで構成されます。

ログ出力を見てください：

```execute
oc logs dc/db
```

データベースが実行されるまで約30秒かかります。ログ出力に``ready for connections``が表示されルはずです。そうでない場合は、上記の``oc logs``コマンドを再試行してください。

データベースが稼働したら、voteデータベースが存在するかどうかを確認して確認します：

```execute
mysql -h db.%project_namespace%.svc -u user -ppassword -D vote -e "show databases"
```

- 以下が表示されます。そうでない場合は、データベースが起動するまで待機するか、上記を確認してから再試行してください。 

```
+--------------------+
| Database           |
+--------------------+
| information_schema |
| vote               |
+--------------------+
```

## Connect the application to the database 

まず、現在の投票アプリの出力を見てください:

```execute
oc logs dc/vote-app | grep ^Connect 
```

データを保存するために``sqlite``（内部データベース）を使用していることがわかります。これを以下で変更します。

この時点では、アプリケーションはMySQLデータベースの存在を認識していません！新しいデータベースを使用するには、アプリケーションを再構成する必要があります。これを行うには、投票アプリケーションの環境変数を変更し、Podを再起動します。投票アプリケーションは、起動時に環境変数の存在を探し、それらを使用して自身を構成します。データベース資格情報が定義されている場合、アプリケーションはデータベースに接続し、必要なテーブルとデータをプロビジョニングします。

アプリケーションをデータベースに接続します：

```execute
oc set env dc vote-app \
   ENDPOINT_ADDRESS=db \
   PORT=3306 \
   DB_NAME=vote \
   MASTER_USERNAME=user \
   MASTER_PASSWORD=password \
   DB_TYPE=mysql
```

上記のコマンドは、起動時の引数として環境変数を設定します。構成の変更により、deployment configurationはPodを自動的に再起動します。

アプリケーションが正常に実行されており、データベースに接続できたことを確認します:

```execute
oc logs dc/vote-app 
```

出力に次の内容が表示され、使用されているデータベース資格情報が表示されます。そうでない場合は、待ってから``oc logs``コマンドを再試行してください。


```
---> Running application from Python script (app.py) ...
Connect to : mysql://user:password@db:3306/vote
...
```

データベースに投票アプリケーションテーブルが作成されていることを確認します。

<!--
POD=`oc get pods --selector app=workspace -o jsonpath='{.items[?(@.status.phase=="Running")].metadata.name}'`; echo $POD

kubectl get pods --field-selector=status.phase=Running -o name
-->

```execute
mysql -h db.%project_namespace%.svc -u user -ppassword -D vote -e "show tables"
```

pollとoptionsテーブルが存在するはずです。

```
+----------------+
| Tables_in_vote |
+----------------+
| option         |
| poll           |
+----------------+
```


<!--
```
POD=`oc get pods --selector app=workspace -o jsonpath='{.items[?(@.status.phase=="Running")].metadata.name}'`; echo $POD
```
-->

# Test the application 

テストスクリプトを使用して、ランダムな投票をアプリケーションに投稿します:

```execute 
test-vote-app http://vote-app-%project_namespace%.%cluster_subdomain%/vote.html
```
（このスクリプトは、さらに投票する場合は複数回実行できます）。

結果を表示するには、次のコマンドを使用します。投票が表示されるはずです。:

<!--
```execute 
curl -s http://vote-app-%project_namespace%.%cluster_subdomain%/results.html | grep "data: \["
```
-->

```execute
mysql -h db.%project_namespace%.svc -u user -ppassword -D vote -e 'select * from `option`;'
```

  - ``Note:`` アプリケーションが期待どおりに動作しない場合のトラブルシューティングの手順についてはページの最後を参照してください。


```
+----+------------------------------+---------+-------+
| id | text                         | poll_id | votes |
+----+------------------------------+---------+-------+
|  1 | Mint                         |       1 |     3 |
|  2 | Fedora                       |       1 |     4 |
|  3 | Debian                       |       1 |     7 |
|  4 | Ubuntu                       |       1 |     3 |
|  5 | Fedora CoreOS                |       1 |     2 |
|  6 | Centos                       |       1 |     5 |
|  7 | Red Hat Universal Base Image |       1 |     7 |
|  8 | Alpine                       |       1 |     1 |
|  9 | Arch                         |       1 |     5 |
| 10 | Other                        |       1 |     3 |
+----+------------------------------+---------+-------+
```

コンソールでコンテナ/Podを表示します:

* [View the Pods](%console_url%/k8s/ns/%project_namespace%/pods) 

ブラウザで投票アプリケーションの結果ページを開きます: 

* [Open the Application](http://vote-app-%project_namespace%.%cluster_subdomain%/results.html) 

これで、アプリケーションは組み込みのデータベースに依存しなくなり必要に応じて自由にスケールアウト（``コンテナの追加``）ができます。
コンソールに移動し、アプリケーションポッドを1から3にスケーリングします（3を超えてスケ​​ーリングしないでください）。Deployment Config：``vote-app``のメニューからpod ``count``を変更します。


* [Deployment Configs](%console_url%/k8s/ns/%project_namespace%/deploymentconfigs)

次のコマンドも使用できます:

```execute
oc scale dc vote-app --replicas=3
```

30〜60秒後に、下のウィンドウに次のような出力が表示されます:

```
vote-app-3-52kqp    1/1     Running     0          17s
vote-app-3-nb5fk    1/1     Running     0          17s
vote-app-3-p2j4w    1/1     Running     0          7m45s
```

- すべてのアプリケーション`state`がデータベースに保存されていることに注意してください。したがってstatelessな各コンテナ/Podは必要に応じて自由に停止および開始できます。

アプリケーションが期待どおりに動作していることを確認します。データは残ります：

[Open the Vote Application](http://vote-app-%project_namespace%.%cluster_subdomain%/results.html) 

 - ``投票アプリケーションを1に戻すか、次のコマンドを使用することを忘れないでください:``

```execute
oc scale dc vote-app --replicas=1
```

---
これでこの演習は終わりです。

この演習では、テスト目的でデータベースを起動し、アプリケーションをそれに接続しました。

次の演習に進みます。

---
### Troubleshooting instructions

投票アプリケーションは機能していますか？

アプリケーションが初めて起動してデータベースを初期化すると、ログ出力は次のようになります：

![log output 1](images/vote-app-start-1.png)

アプリケーションが再起動すると、ログ出力は次のようになります：

![log output 1](images/vote-app-start-2.png)

出力にエラーメッセージが表示されていますか？

問題を修正できない場合は、データベースを空にして、次のコマンドを使用してアプリケーションを再起動します: 

```execute 
delete-test-database
```

それでも問題を解決できない場合は、次のコマンドを使用してデータベースを再作成し、アプリケーションを再起動します。: 

```execute 
recreate-test-database
```

スクリプトが完了したら、指示に戻って再試行してください。

<!-- drop table `option`;  delete from `option`;  -->


[indexへ戻る](../index-aws.ja.md)