この演習では、``Red Hat OpenShift``を理解し、コマンドラインおよびWebコンソールについて慣れることができます。


Red Hat OpenShiftは、エンタープライズアプリケーションの開発とデプロイのためのKubernetesコンテナオーケストレータに基づくオープンソースコンテナアプリケーションプラットフォームです。

OpenShift Container Platformに慣れていない場合は、このワークショップで使用するプラットフォームの基本と環境を理解するのに数分かかる価値があります。

OpenShiftのゴールは、開発者とシステム管理者の両方がコンテナ化されたアプリケーションを開発、デプロイ、実行するための素晴らしい体験を提供することです。開発者は、詳細を知らなくてもコンテナ化されたアプリケーションとオーケストレーションの両方を活用できるため、OpenShiftの使用を好むはずです。開発者は、Dockerfileの作成やdockerビルドの実行に時間を費やす代わりに、自由にコードに集中できます。

![image](images/ocp4-arch-diagram.png)

OpenShiftは、いくつかのアップストリームプロジェクトを組み込んだ完全なプラットフォームであり、それらのアップストリームプロジェクトを使いやすくするための追加機能も提供します。プラットフォームの中核は、コンテナとオーケストレーションです。コンテナ観点では、プラットフォームは、docker、podman、buildahでビルドされたイメージを含む``Open Container Initiative`` (OCI)準拠のコンテナイメージをサポートします。オーケストレーション観点では、アップストリームの ``Kubernetes``プロジェクトに多くのリソースを投入しました。これら2つのアップストリームのプロジェクト以外にも、このワークショップで学習``routes``と``deployment configs``のようなKubernetesへの追加セットオブジェクトを作成しています。

このトレーニングで使用するコマンドラインツールは、ocツールと呼ばれます。このツールは、Windows、OS X、およびLinuxオペレーティングシステム用に提供されています。

OpenShiftは、プラットフォームと対話するための使いやすいグラフィカルインターフェイスを提供する機能豊富なWebコンソールも提供します。

始めましょう！！

最初に、コマンドラインを使用して、OpenShiftでの認証方法を表示します：

```execute
oc whoami
```

ユーザー名が表示されます。

``oc``クライアントとサーバーの両方のバージョンを表示します。

```execute
oc version
```

サーバーのバージョンに注意してください。「メジャー」と「マイナー」の番号、たとえば4と1+を探します。

示されているOpenShift APIエンドポイントは、すべてのツール、特に``oc``クライアントとWebコンソールが通信するすべてのOpenShift APIを提供します。``kubectl``コマンドに慣れていれば、このコマンドも使用できます。OpenShiftには、拡張された完全なPaaSを実現する機能などOpenShiftの他のコンポーネント一緒に、統合およびテストされたKubernetesプロジェクトのバニラバージョンが含まれています。

```execute
kubectl version
```

アクセスできるプロジェクト（namespaces）を表示します：

```execute
oc projects
```

このプロジェクト(``%project_namespace%``)は、ワークショップの期間中で使用するあなた専用のプロジェクトです。他の人が自分のプロジェクトで作業する間、あなたはこのプロジェクトで作業します。したがって、誰もが互いに干渉することなく作業することができます。これはRed Hatによって開発され、アップストリームKubernetesプロジェクトに貢献した``Role Based Access Control``（RBAC）システムの一部です。

  - ``Warning: should you see the error message ``Restricted Access, You don't have access to this section due to cluster policy.`` in the console, this means you are trying to access the wrong project.  Please ensure you select the correct project from the drop-down menu at the top of the page``  

以下のリンクを使用して、OpenShift Consoleを見て、次を表示します。

プロジェクトはここで表示できます。必要なのは1つだけです。

* [Projects](%console_url%) 

<!--
The status of your project can be seen here:

* [Status](%console_url%/overview/ns/%project_namespace%)

Click on the ``Dashboard`` button to see your resource usage in your project. As you have just started the workshop you are using nothing, so it shows 0% utilization for CPU and Memory.  
-->

OpenShiftイベントはここで見ることができます：

* [Events](%console_url%/k8s/ns/%project_namespace%/events)

何かが正常に機能していない場合は、ここを参照するといいです。

さまざまなOpenShiftリソースを表示できます：

* [Pods](%console_url%/k8s/ns/%project_namespace%/pods) - あなたのプロジェクト内のPodを表示. 
* [Build Configs](%console_url%/k8s/ns/%project_namespace%/buildconfigs) - アプリケーションイメージのビルドリソース.
* [Deployment Configs](%console_url%/k8s/ns/%project_namespace%/deploymentconfigs) - アプリケーションのライフサイクルを管理するリソース.
* [Routes](%console_url%/k8s/ns/%project_namespace%/routes) - 外部ネットワークからアプリケーションへのアクセスを許可するリソース.

<!--
* [Workloads](%console_url%/k8s/cluster/projects/%project_namespace%/workloads)
-->

<!--
You can view various technologies, including Source to Image, Templates and Operators here:

* [Developer Catalog](%console_url%/catalog/ns/%project_namespace%)
-->

<!--
Note, this is not availabe on RHPDS  or only for admin users... 
* [Operator management](%console_url%/operatormanagement/ns/%project_namespace%)
-->

端末タブに戻るか、ここをクリックしてください：

* [Terminal](%terminal_url%)

右上隅のメニューを使用して``Open Console``を選択すると、別のタブでOpenShift Consoleを開くことができます。

---
これでこの演習は終わりです。

この演習では、OpenShift、コンソール、およびコマンドラインについて紹介しました。


[indexへ戻る](../index-aws.ja.md)