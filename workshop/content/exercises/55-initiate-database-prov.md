この演習では、AWS Relational Database Service（RDS）インスタンスをプロビジョニングします。これにより後の演習でアプリケーションを設定して使用できるようになります。Amazon RDSを使用すると、クラウドでのMySQLデプロイメントのセットアップ、運用、およびスケーリングが簡単になります。

``AWS Service Broker``を使用してデータベースをプロビジョニングします。


- ``AWS Service Brokerは、Red Hat OpenShiftおよびKubernetesを介してネイティブAWSサービスを直接公開できるようにするオープンソースプロジェクトです。ブローカーは、OpenShift内でのAWSサービスの簡単な統合を直接提供します。``

![AWS Service Broker Arch](images/aws-service-broker-architecture.png)

このOpenShiftクラスター上で構成され実行されている``AWS Broker``を使って、AWS RDS MySQLサービスをプロビジョニングします。そしてMySQLデータベースをアプリケーションに接続してテストします。

# Provision a RDS MySQL database using the service catalogue 

RDSのプロビジョニングを開始するには、次の``svcat``コマンドを実行します。: 

```execute
svcat provision mysql --class rdsmysql --plan custom  \
   -p PubliclyAccessible=true \
   -p DBName=vote \
   -p AccessCidr=0.0.0.0/0 \
   -p MasterUsername=user \
   -p MasterUserPassword=ocpWorkshop2019 \
   -p MultiAZ=false \
   -p DBInstanceClass=db.m4.large \
   -p AutoMinorVersionUpgrade=false \
   -p PortNumber=13306 \
   -p BackupRetentionPeriod=0 
```
 <!-- -p VpcId=vpc-03a00c0e08cc9bec3  note that this param is not needed.  The AWS Service Broker should be configured with the target VPN -->

上記のコマンドで指定されているように、MySQLのインスタンスは``db.m4.large``のインスタンスタイプでプロビジョニングされ、パブリックにアクセスできます。

コマンドラインからRDSインスタンスのステータスを確認します:

```execute
svcat get instances
```

ステータスは``Provisioning`である必要があります。データベースのプロビジョニングには約20分かかることに注意してください。後の演習で使用します。

コンソールで作業したい場合は、[Service Catalog](%console_url%/catalog/ns/%project_namespace%)にアクセスして作成することもできます。利用できる多くの技術が表示されます。それらの1つがRDSサービスです。検索ボックスに``rds``と入力します。``Amazon RDS for MySQL``サービスクラスが表示されます。この"Amazon RDS for MySQL"をクリックして、このサービスに関する情報を参照してください。次に、``Create Service Instance`` ボタンをクリックして、MySQLインスタンスの定義に使用できるすべてのパラメーターを表示します。最も重要なオプションは次のとおりです。

1. サービスインスタンス名
1. DBインスタンスクラス
1. マスターユーザーのパスワード
1. DBName
1. StorageEncrypted
1. 一般にアクセス可能
1. マスターユーザー名


``Please note: 実際にはページ下部の「作成」ボタンをクリックしないでください！`` ``svcat``コマンド を使用して、すでに上記のRDSインスタンスをプロビジョニングしています。 

[Provisioned Services](%console_url%/provisionedservices/ns/%project_namespace%/) をコンソールで見て、RDSインスタンスのステータスを確認します。``Not ready``が表示されるはずです。``mysql``サービスオブジェクトを確認すると、``The instance is being provisioned asynchronously``と、ページの下部にも表示されるはずです。また、``Service Bindings``は何も表示されません。サービスバインディングは、サービスインスタンスとアプリケーション間のリンクです。心配しないでください。今後の演習で作成します。


- 重複するネットワークセグメントが自動的に選択されるため（またはその他の理由）、RDSインスタンスがプロビジョニングに失敗する場合があります。この場合、最終的にステータスが``Failed``として表示されます。
- RDSインスタンスのプロビジョニングに問題がある場合は、それを削除して再試行する必要があります。失敗したRDSインスタンスを削除するには、`"svcat deprovision mysql"`コマンドを実行します。

  <!--follow the steps in the section ``Remove the RDS Instance`` in the last exercise exercise called [Clean up](90-clean-up).  -->

データベースのプロビジョニングには約20分かかります。

---
That's the end of this exercise.

これでこの演習は終わりです。

インスタンスのステータスが`Provisioning`として表示されたら、次の演習に進みます。


[indexへ戻る](../index-aws.ja.md)