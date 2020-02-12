この演習では、ソースコードを変更し、コンテナを再構築/再デプロイします。

# Change the application source code 

エディタを使用してソースコードを変更するか、以下の``sed``コマンドを使用できます。
まず、正しいソースコードディレクトリにいることを確認します。

```execute
cd ~/flask-vote-app
```

<!--
If you want to use an editor, you could use the ``nano`` editor to change the title of the voting definition (e.g. "Linux distribution") and then save the file again:

```execute
nano seeds/seed_data.json
```
Once you have made the change, save the file using ``CTL-X``, then ``Y`` and then hit ``ENTER``.  
-->

次のコマンドを使用して、投票定義ファイルの一部のテキストを置き換えます：

```execute
sed -i "s/avourite distribution/avourite Linux Distribution/" seeds/seed_data.json
```
ソースコードに変更を加えました。単語を``Distribution``へ変更しました。開発者として、あなたは今、’ローカルワークステーション’上で、アプリケーションのビルドと’test’を行いたいはずです。そのあとGitHubへ変更の``commit`` と ``push``を行いたいはずです。

gitで行った変更を確認します:

```execute
git status
```

Gitは、seed_data.jsonファイルに変更を加えたことを示します:
 - ``modified:   seeds/seed_data.json``.

Gitは、コミットしようとしている変更を表示することもできます。

```execute
git diff seeds/seed_data.json
```

# Commit and push the changes 

変更をコミットします: 

```execute
git commit -m "Important change" . 
```

 - ``次のコマンドの後、前の演習でGitHub webhookをセットアップしていた場合、ソースからイメージへのビルドが自動的に開始されるのが見えるはずです。下のターミナルでビルドポッドが実行されているのがわかります。``

次に、変更をGitHubにプッシュします:

```execute
git push 
```

前の演習でWebhookを設定していた場合は、ビルドが自動的に開始されるのを確認できます。下のターミナルウィンドウを見てください、``vote-app-2-build 1/1 Running`` が確認できるはずです。


 - ``Warning: 前の演習でWebhookをセットアップしなかった場合、次のコマンドを使用して新しいビルドを手動でトリガーします``

```execute
# ONLY RUN IF YOU DID NOT SET UP THE WEBHOOK IN THE PREVIOUS LAB
oc start-build vote-app   
```

続行する前にビルドが実行されていることを確認してください。

繰り返しますが、ビルドの進行状況を確認してください：
```execute
oc logs bc/vote-app --follow
```
pythonアプリケーションが再構築され、新しいイメージが内部レジストリにコミット/プッシュされるのが見えるはずです。アプリケーションを自動的に再デプロイする必要があります。注、これは、数分かかることがあります。特に``Copying blob...``と``Storing signatures``が遅くなることがあります。

新しいPodが起動して実行されたら、行った変更が適切に展開されていることを確認します。

# Verify the application changes 

```execute 
curl -s http://vote-app-%project_namespace%.%cluster_subdomain%/ | grep "<title>"
```

出力には、次のような変更が含まれている必要があります。

```
    <title>Favourite Linux Distribution</title>
```

 - ``これが機能しない場合は、戻ってビルドが正常に終了し、アプリケーションが再デプロイされていることを確認します``

次のURLを使用して、ブラウザーでアプリケーションをテストします:

[Open the Vote Application](http://vote-app-%project_namespace%.%cluster_subdomain%/)


---
これでこの演習は終わりです。

これで、コードを変更して結果をテストする方法がわかりました。次に、投票アプリケーションをデータベースに接続します。



[indexへ戻る](../index-aws.ja.md)