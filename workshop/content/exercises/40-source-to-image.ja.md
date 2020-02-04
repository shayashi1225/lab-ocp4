この演習では、OpenShiftで投票アプリケーションをビルドおよび起動するために``Source to Image``を使用します。

OpenShiftプロジェクトの最初の主な目的の1つは、開発者がプラットフォームでコードを実行するのを本当に簡単にすることでした。

”oc new-app”は、OpenShiftでさまざまな方法によりアプリケーションを初期化するコマンドです。OpenShiftでソースコードを実行するために使用します。

Source 2 Image (s2i) は次のことを行います：

1. 一致するランタイムの"builder image"からコンテナを起動します。今回は、Python 2.7のビルダーイメージです。
2. 実行中のビルダーコンテナーでアプリケーションのビルドを実行します。
3. ビルドが成功すると、s2iはビルドされたアプリケーションを含む新しいイメージをコミットし、OpenShiftの内部レジストリにプッシュします。
4. 新しいイメージが作成されたため、コンテナーが起動されます。コンテナが以前のビルドからすでに実行されている場合、コンテナは再デプロイされます。

下部のターミナルで次のコマンドを実行して、残りの演習中に実行中のcontainers/podsを表示します：

```execute-2
watch "oc get pods | grep -v -e ' Completed ' -e \-deploy"
```

このコマンドはほとんどの演習で実行されます。今はまだ何も作成していないので``No resources found``が表示されるはずです。

# Execute the Source 2 Image build process 

次に``new-app`` を``--dry-run``オプションを指定してコマンドを実行し、コマンドが何をするかを確認します

Now run the ``new-app`` command with the ``--dry-run`` option to see what it will do:

```execute
oc new-app python:2.7~. --name vote-app --dry-run
```

 - ``Note: 通常、"new-app"コマンドはソースコードに基づいて一致するビルダーイメージを自動的に選択しますが、コードが適切に機能するにはPythonバージョン2.7が特に必要であるため、使用するビルダーイメージの名前とバージョンを明示的に指定します（python： 2.7）。``

上記のコマンドは次のことを行います:

1. 現在の作業ディレクトリ"."を調べ、pythonソースコードを検出し、関連するGitHubリポジトリを決定します。
2. ``build configuration``（BC）というビルドオブジェクトを作成します。ビルド構成は以下を認識します：
  1. ``python builder image``を保持するリポジトリの場所 
  2. GitHubリポジトリなどからソースコードを取得する場所
  3. 内部コンテナレジストリにプッシュされる出力イメージの名前（``vote-app``）
3. ``image streams``（IS）（別名OpenShiftイメージオブジェクト）を作成して、ビルダーと最終的なアプリケーションイメージを追跡します
  1. これらのimage streamsは、画像が更新されるタイミングを検出し、アプリケーションの再構築または再展開をトリガーできます。
4. ``deployment configuration``（DC）と呼ばれる展開オブジェクトを作成します。デプロイメント構成は以下を認識します。
  1. イメージが更新された場合のアプリケーションの再デプロイ方法
5. ``service object``を作成して、1つ以上の実行中のアプリケーションコンテナへの検出とアクセスを可能にします。
警告やエラーがなく、すべてが正常に見える場合は、``--dry-run``オプションなしでコマンドを実行します:

```execute
oc new-app python:2.7~. --name vote-app 
```
このコマンドは、作成されたすべてのOpenShiftオブジェクトをリストします。

<!--
- ``Note: Should the build configuration already exists from a previous invocation, start the build again with the following command:``

```execute
oc start-build vote-app 
```
-->

下のターミナルウィンドウで、ビルドコンテナが実行されているのを確認できます。それがあなたの投票アプリケーションを構築しているものです。``vote-app-1-build   1/1     Running``と表示されるはずです。

次のように、コンソールおよびコマンドラインでビルドプロセスを実行できます。

```execute 
oc logs bc/vote-app --follow 
```

コンソールでビルドの出力を表示するには、ビルド（``vote-app-1``）をクリックしてからLogsタブをクリックします：

[View the build](%console_url%/k8s/ns/%project_namespace%/builds)


**Wait for the build to finish before continuing**.

ビルドには数分かかり、特に``Copying blob...``、``Storing signatures``操作が遅くなる可能性があります。

ビルド出力の中に次のものが表示されます:

```
...
Cloning "https://github.com/johnsmith/flask-vote-app.git" ...
...
STEP 8: RUN /usr/libexec/s2i/assemble
...
STEP 10: COMMIT temp.builder.openshift.io/...
...
Successfully pushed ...
Push successful
```

ビルド中はどうなりますか？

1. ソースコードが複製されます。
2. Pythonビルダーイメージが起動し、コードがコピーされます。
3. s2i ``assemble script``が実行されます。これはPythonアプリケーションの構築方法を知っています。
4. Pythonの依存関係がインストールされます
5. 実行中のコンテナがコミットされ、新しいイメージが作成されます
6. その後、イメージはOpenShiftの内部コンテナーレジストリにプッシュされます
ビルドが完了すると、イメージが自動的に起動され、pod内のコンテナーが作成されます。

下のターミナルウィンドウで、ビルドコンテナが完了し（``vote-app-1-build Completed``）、新しいアプリケーション``vote-app-1-xxyyzz``コンテナが起動しているのが見えるはずです。

次のコマンドを実行して、プロジェクトで実行されているポッドを表示することもできます: 

```execute
oc get pods
```

ビルドが完了するまで待ちます。ビルドログ出力に（``Push successful``）が表示され、ビルドポッドが ``Completed``と表示されるはずです。

次のようなものが表示されるはずです:

```
NAME               READY     STATUS      RESTARTS   AGE
vote-app-1-build   0/1       Completed   0          4m
vote-app-1-deploy  0/1       Running     0          3m
vote-app-1-gxq5k   1/1       Running     0          30s
```

1. vote-app-1-buildポッドは、それがしていたこと、すなわちpythonアプリケーションのビルドを完了しました。
2. 投票アプリケーションポッドをデプロイするために、vote-app-1-deployポッドが起動されました。
3. これで、vote-app-1-gxq5kポッドが開始され``Running``となります。


次に、コンソールを見て、実行中のアプリケーションを表示します。``vote-app-1-xxyyzz``と呼ばれる実行中のpodがあるはずです。また、vote-app-1-xxyyzz pod->Terminalタブをクリックして、ターミナルウィンドウを開き、実行中のコンテナーの内部を確認します。コンテナ内で``ps -ef``を実行して、実行中のpythonプロセスを確認します。: 

[View the console](%console_url%/k8s/ns/%project_namespace%/pods) 

実行中のコンテナ内に次のようなものが表示されるはずです。:

```
(app-root)sh-4.2$ ps -ef
UID         PID   PPID  C STIME TTY          TIME CMD
1002280+      1      0  0 08:24 ?        00:00:00 python app.py
1002280+     34      0  0 08:24 pts/0    00:00:00 sh
1002280+     64      0  0 08:27 pts/1    00:00:00 sh
1002280+     88     64  0 08:27 pts/1    00:00:00 ps -ef
(app-root)sh-4.2$
```

プロジェクトのステータスは次のとおりです:

[Project Status](%console_url%/overview/ns/%project_namespace%) 

ここでは、アプリケーションのライフサイクルの管理に役立つDeployment Configurationオブジェクトを見ることができます。 


# Expose the application for testing 

デフォルトでは、アプリケーションはOpenShiftの外部からアクセスできません。次に、アプリケーションを外部ネットワークに公開して、テストできるようにします:

```execute
oc expose svc vote-app
```
上記のコマンドは``route``オブジェクトを作成します。OpenShift Container Platformルートは、www.example.comなどのホスト名でサービスを公開するため、外部クライアントは名前でサービスにアクセスできます。

ルートオブジェクトを確認します:

```execute 
oc get route
```

アプリケーションへのアクセスに使用するホスト名が表示されます。 

# Test the application 

アプリケーションが機能していることを確認するには、curlを使用するか、ブラウザにURLをロードします。

curlを使用して、アプリが機能していることを確認します:

```execute 
curl http://vote-app-%project_namespace%.%cluster_subdomain%/ 
```

または、予想される出力を確認する別の方法を使用します。:

```execute 
curl -s http://vote-app-%project_namespace%.%cluster_subdomain%/ | grep "<title>"
```

次の出力が表示されます。これは、アプリケーションが機能していることを意味します：

```
    <title>Favourite distribution</title>
```

アプリケーションは、ヘルパースクリプトを使用してさらにテストできます。

ヘルプスクリプトを使用して、アプリケーションにランダムな投票をいくつか投稿します：

```execute 
test-vote-app http://vote-app-%project_namespace%.%cluster_subdomain%/vote.html
```

結果を表示するには、次のコマンドを使用します。すべての投票オプションの合計が表示されます。

```execute 
curl -s http://vote-app-%project_namespace%.%cluster_subdomain%/results.html | grep "data: \["
```

次のようなものが表示され、すべての票が表示されます

```
  data: [ "3",  "3",  "2",  "0",  "1",  "5",  "1",  "3",  "2", ],

```

または、ブラウザで結果ページを表示します:

[View Results page](http://vote-app-%project_namespace%.%cluster_subdomain%/results.html)


Note that:

 - メッセージ``Application is not available``が表示される場合、これはアプリケーションがまだ実行されていないか、ビルドが失敗したことを意味します。
 - あなたの隣人はあなたの申請書にアクセスして投票を提出できるはずです。ブラウザごとに1つの投票のみが許可されることに注意してください。
 - デフォルトでは、アプリケーションは組み込みデータベースを使用して投票データを保存します。後の演習では、外部MySQLデータベースを使用するようにアプリケーションを構成します。
 

<!--
Finally, take a look at the resource utilization in the Console: 

* [Status](%console_url%/overview/ns/%project_namespace%)

Click on the ``Dashboard`` button to see your resource usage in your project. 
-->

---
これで演習は終わりです。

この演習では、アプリケーションを構築してテストしました。


---
## Example output of a full application build:

```
$ oc logs bc/vote-app
Cloning "https://github.com/sjbylo3/flask-vote-app.git" ...
	Commit:	23d4bdeec2449deb1532280cce6be54b6f0200f0 (update)
	Author:	Your Name <you@ example.com>
	Date:	Wed Jul 3 09:35:55 2019 +0000
Caching blobs under "/var/cache/blobs".
Getting image source signatures
Copying blob sha256:db1d55616933198cd32cb3a3a658a903a9205c733af15ca6423268d83a2a5840
...
Writing manifest to image destination
Storing signatures
07822e6843338f8ad388f1f34294082de46f7e897c6a743d60dde1e3af55be71
Generating dockerfile with builder image image-registry.openshift-image-registry.svc:5000/openshift/python@sha256:b604de44d1d298873ba1620e2941536a4ec2c836b43eafdcbcd61132bd446d70
STEP 1: FROM image-registry.openshift-image-registry.svc:5000/openshift/python@sha256:b604de44d1d298873ba1620e2941536a4ec2c836b43eafdcbcd61132bd446d70
STEP 2: LABEL "io.openshift.build.image"="image-registry.openshift-image-registry.svc:5000/openshift/python@sha256:b604de44d1d298873ba1620e2941536a4ec2c836b43eafdcbcd61132bd446d70" "io.openshift.build.commit.author"="Your Name <you@example.com>" "io.openshift.build.commit.date"="Wed Jul 3 09:35:55 2019 +0000" "io.openshift.build.commit.id"="23d4bdeec2449deb1532280cce6be54b6f0200f0" "io.openshift.build.commit.ref"="master" "io.openshift.build.commit.message"="update" "io.openshift.build.source-location"="https://github.com/sjbylo3/flask-vote-app.git"
STEP 3: ENV OPENSHIFT_BUILD_NAME="vote-app-6" OPENSHIFT_BUILD_NAMESPACE="lab-ocp4" OPENSHIFT_BUILD_SOURCE="https://github.com/sjbylo3/flask-vote-app.git" OPENSHIFT_BUILD_REFERENCE="master" OPENSHIFT_BUILD_COMMIT="23d4bdeec2449deb1532280cce6be54b6f0200f0"
STEP 4: USER root
STEP 5: COPY upload/src /tmp/src
STEP 6: RUN chown -R 1001:0 /tmp/src
STEP 7: USER 1001
STEP 8: RUN /usr/libexec/s2i/assemble
---> Installing application source ...
---> Installing dependencies ...
You are using pip version 7.1.0, however version 19.1.1 is available.
You should consider upgrading via the 'pip install --upgrade pip' command.
Collecting flask (from -r requirements.txt (line 1))
  Downloading https://files.pythonhosted.org/packages/9a/74/670ae9737d14114753b8c8fdf2e8bd212a05d3b361ab15b44937dfd40985/Flask-1.0.3-py2.py3-none-any.whl (92kB)
Collecting flask-sqlalchemy (from -r requirements.txt (line 2))
  Downloading https://files.pythonhosted.org/packages/08/ca/582442cad71504a1514a2f053006c8bb128844133d6076a4df17117545fa/Flask_SQLAlchemy-2.4.0-py2.py3-none-any.whl
Collecting mysql-python (from -r requirements.txt (line 3))
  Downloading https://files.pythonhosted.org/packages/a5/e9/51b544da85a36a68debe7a7091f068d802fc515a3a202652828c73453cad/MySQL-python-1.2.5.zip (108kB)
Collecting itsdangerous>=0.24 (from flask->-r requirements.txt (line 1))
  Downloading https://files.pythonhosted.org/packages/76/ae/44b03b253d6fade317f32c24d100b3b35c2239807046a4c953c7b89fa49e/itsdangerous-1.1.0-py2.py3-none-any.whl
Collecting Werkzeug>=0.14 (from flask->-r requirements.txt (line 1))
  Downloading https://files.pythonhosted.org/packages/9f/57/92a497e38161ce40606c27a86759c6b92dd34fcdb33f64171ec559257c02/Werkzeug-0.15.4-py2.py3-none-any.whl (327kB)
Collecting Jinja2>=2.10 (from flask->-r requirements.txt (line 1))
  Downloading https://files.pythonhosted.org/packages/1d/e7/fd8b501e7a6dfe492a433deb7b9d833d39ca74916fa8bc63dd1a4947a671/Jinja2-2.10.1-py2.py3-none-any.whl (124kB)
Collecting click>=5.1 (from flask->-r requirements.txt (line 1))
  Downloading https://files.pythonhosted.org/packages/fa/37/45185cb5abbc30d7257104c434fe0b07e5a195a6847506c074527aa599ec/Click-7.0-py2.py3-none-any.whl (81kB)
Collecting SQLAlchemy>=0.8.0 (from flask-sqlalchemy->-r requirements.txt (line 2))
  Downloading https://files.pythonhosted.org/packages/62/3c/9dda60fd99dbdcbc6312c799a3ec9a261f95bc12f2874a35818f04db2dd9/SQLAlchemy-1.3.5.tar.gz (5.9MB)
Collecting MarkupSafe>=0.23 (from Jinja2>=2.10->flask->-r requirements.txt (line 1))
  Downloading https://files.pythonhosted.org/packages/b9/2e/64db92e53b86efccfaea71321f597fa2e1b2bd3853d8ce658568f7a13094/MarkupSafe-1.1.1.tar.gz
Installing collected packages: itsdangerous, Werkzeug, MarkupSafe, Jinja2, click, flask, SQLAlchemy, flask-sqlalchemy, mysql-python
  Running setup.py install for MarkupSafe
  Running setup.py install for SQLAlchemy
  Running setup.py install for mysql-python
Successfully installed Jinja2-2.10.1 MarkupSafe-1.1.1 SQLAlchemy-1.3.5 Werkzeug-0.15.4 click-7.0 flask-1.0.3 flask-sqlalchemy-2.4.0 itsdangerous-1.1.0 mysql-python-1.2.5
STEP 9: CMD /usr/libexec/s2i/run
STEP 10: COMMIT temp.builder.openshift.io/lab-ocp4/vote-app-6:08b9efd8
Getting image source signatures
Copying blob sha256:8783de338a118d308a5f8e00576afc318fac3a8a35767d95948493915cc249a8
...
Writing manifest to image destination
Storing signatures
--> 4efd91078c869feb60bcdbae4b6683cb12984fb20d4dc1bf208f1d7684375860

Pushing image image-registry.openshift-image-registry.svc:5000/lab-ocp4/vote-app:latest ...
Getting image source signatures
Copying blob sha256:db1d55616933198cd32cb3a3a658a903a9205c733af15ca6423268d83a2a5840
...
Writing manifest to image destination
Storing signatures
Successfully pushed //image-registry.openshift-image-registry.svc:5000/lab-ocp4/vote-app:latest@sha256:cf182b356492d25b9a5af1e014564bbb52691c530e2a8e8928ce70898a0596f5
Push successful
```


[indexへ戻る](../index-aws.ja.md)