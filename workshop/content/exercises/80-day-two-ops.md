この演習では、deploymentプロセスを管理する方法を学習します。

 - Note: ``この演習はオプションです。`` 

アプリケーションが3つのコンテナー/podにスケーリングされていることを確認します。: 

```execute
oc scale --replicas=3 dc/vote-app
```

この演習を通して、[console](%console_url%/k8s/ns/%project_namespace%/pods)でスケーリング/ロールアウトアクションを確認できます。

```execute
oc get pods
```

アプリケーションをテストして、期待どおりに動作することを確認します: 

```execute 
curl -s http://vote-app-%project_namespace%.%cluster_subdomain%/ | grep "<title>"
```

もう一つのウィンドウで確認... 
まず下のターミナルで実行されている``watch``コマンドを停止します。


```execute-2
<ctrl+c>
```

監視する下の端末で``rollout history``を監視するためにこのコマンドを実行します:

```execute-2
watch oc rollout history dc vote-app
```

DeploymentsConfigs（Kubernetesオブジェクト）は、一般的なユーザーアプリケーションに対するきめ細かい管理を提供します。必要に応じて、[OpenShift Deployment Configs](https://docs.openshift.com/container-platform/3.11/dev_guide/deployments/how_deployments_work.html)の詳細を確認してください。試してみましょう。

アプリケーションを再度ロールアウトします: 

```execute
oc rollout latest vote-app
```

これにより、``同じバージョン``のアプリケーションが再びデプロイされます。また、これにより新しい``リビジョン``のデプロイメントが作成されます。下の端末を見て、新しいリビジョンを確認します。

以下を確認します:

```
...   Running         manual change
```

アプリケーションがロールアウトされ、再度``Running``となるのを待ちます。

deploymentのロールバック（取り消し）：

```execute
oc rollout undo dc vote-app
```

これにより、アプリケーションが以前のバージョンにロールバックされます。

アプリケーションを再度ロールアウトします：

```execute
oc rollout latest vote-app
```

ロールアウトが計画どおりに進まない場合は、キャンセルできます。今すぐ試してください：

```execute
oc rollout cancel dc vote-app
```

アプリケーションの以前のバージョン、リビジョン2へのdeploymentをロールバック（取り消し）します： 

```execute
oc rollout undo dc vote-app --to-revision=2
```

新しいロールアウトが``Running``と表示されるはずです。

アプリケーションがロールバックされ、再度``Running`` となるのを待ちます。このコマンドで確認してください:

```execute
oc get pod
```

覚えているなら、アプリケーションの2回目のデプロイメントでは、コンテナ化されたデータベースでMySQLを使用していました。アプリケーションログ出力を見ると、コンテナ化されたデータベースを再度使用して表示されているはずです。

ログ出力を見て、使用しているデータベースを確認します：
```execute
oc logs dc/vote-app 
```

ログ出力の中で次が表示されるはずです: 

```
Connect to : mysql://user:password@db:3306/vote
```

予想される出力が表示されない場合は、アプリケーションが再び``Running``と表示されるまで待ちます。

興味がある場合は、リビジョン1にロールバックして、アプリケーションが内部sqlite組み込みデータベースを使用していることを確認してください: 

```execute
oc rollout undo dc vote-app --to-revision=1
```

ロールアウトが完了するのを待って、出力ログを再度確認します:

```execute
oc logs dc/vote-app 
```

ログ出力の中で次が表示されるはずです: 

```
Connect to : sqlite:////opt/app-root/src/data/app.db
```

この演習では、アプリケーションの新しいバージョンを簡単にロールアウトし、古いバージョンにロールバックすることもできました。

---
これでこの演習は終わりです。

次の演習に進んでください。


[indexへ戻る](../index-aws.ja.md)