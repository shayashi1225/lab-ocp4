この演習では、Webコンソールを使用してコンテナ化されたアプリケーションをデプロイする方法を試します。OpenShift Container Platformは、アプリケーションをデプロイするさまざまな方法を提供します。コンテナイメージを展開できます。または、OpenShift Container Platformをソースコードリポジトリにポイントして、アプリケーションをビルドおよびデプロイします。また、テンプレートを使用してワークロード全体を展開できます。この演習では、Webコンソールを介して簡単なコンテナーイメージを展開するのがいかに簡単かを確認します。

'Home' -> ['Projects'](%console_url%) をクリックします。作成されたプロジェクトが表示されます。このプロジェクトにアプリケーションをデプロイします。

![project](images/deploy-img1.png)

プロジェクトをクリックすると、実行中のワークロードがないことを示すプロジェクトコンソールが表示されます。

次に、上部の左メニュー項目の``Administrator``をクリックして``Developer``選択し、``Developer``パースペクティブに切り替えますDeveloper。開発者が懸念するすべてのものに焦点を当てたDeveloperパースペクティブが表示されます。

``+Add``メニュー項目をクリックしてコンテナを展開できます。
コンテナイメージを展開するために、左側のメニューで'+Add'をクリックします。

![project](images/exercise-2-1.png)

Deploy Imageページで次のように入力します:

```copy
openshiftroadshow/parksmap-katacoda:1.2.0
```
’Image Name'テキストボックスに入力し、横にある拡大クラスアイコンをクリックします。これにより、Dockerハブからのクエリがトリガーされ、イメージ情報がプルダウンされます。

``Application Name``および``Name``フィールドに「myapp」を入力します。

'Create'ボタンをクリックして、OpenShiftにコンテナーイメージをデプロイします。バックグラウンドで、OpenShiftはイメージをプルダウンし、必要なOpenShiftオブジェクト（service、deploymentConfig）を作成し、イメージをデプロイします。


![project](images/exercise-2-2.png)

アプリケーションを表示するトポロジビューが表示されます。

![project](images/exercise-2-3.png)

アプリケーションドーナツをクリックして、詳細情報のあるサイドメニューを開きます。

![project](images/exercise-2-5.jpeg)

Routeオブジェクトに関する情報を表示するページに移動します。Locationページの右側のセクションの下には、アプリケーションにアクセスするためのURLがあります。

URLをクリックして、アプリケーションにアクセスします。アプリケーションは、単に世界の地図を表示します。表示された場合、アプリケーションは正常に動いています!!

![project](images/deploy-img-e.png)

OpenShiftで最初のアプリケーションを実行しました。おめでとうございます！

この演習のためにリソースをクリーンアップします。これ以上使用することはありません。

```execute
oc delete all --all
```

ターミナルで、削除されているすべての構成とオブジェクトが表示されます。次のような出力が表示されます:

```
pod "parksmap-katacoda-1-9f8xx" deleted
pod "parksmap-katacoda-1-deploy" deleted
replicationcontroller "parksmap-katacoda-1" deleted
service "parksmap-katacoda" deleted
deploymentconfig.apps.openshift.io "parksmap-katacoda" deleted
imagestream.image.openshift.io "parksmap-katacoda" deleted
route.route.openshift.io "myname" deleted
```

これで、ラボのセクションは終了です。


[indexへ戻る](../index-aws.ja.md)