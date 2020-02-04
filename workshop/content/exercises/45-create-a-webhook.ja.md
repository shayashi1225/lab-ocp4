この演習では、ソースコードの変更がGitHubのリポジトリにコミットされるたびに自動的に実行されるようにビルドを構成します。これは``webhook``を使用して実行できます。

この演習はオプションです。

# Create a Webhook 

Webhookを構成する場合は、次のヘルパースクリプトを使用します。ヘルパースクリプトはGitHubでwebhookを構成するために必要な``values``をすべて表示します。

ヘルパースクリプトを実行します：

```execute
getwebhook vote-app %cluster_subdomain%
```

 - ``Important: 以下の手順に従い、ヘルパースクリプトによって表示される値を使用します.``

1. Log into https://github.com
1. あなたの``flask-vote-app`` repositoryへ移動
1. ``Settings`` をクリックして``Webhooks``を選択
1. "``Add Webhook``"ボタンをクリック
1. 次の情報を使用してフォームに入力します：

- Payload URL: ``see the helper script output``
- Content type: application/json
- Secret: ``see the helper script output``
- SSL verification: Disabled 

 - ``WARNING``: エラーThe URL you've entered is not validが表示される場合は、使用しているブラウザによっては改行でスペースが挿入される可能性があるため、長いURLをGitHubフォームにコピーして貼り付ける際に注意してください！ 

他の設定はそのままにします。

最後に、"``Add Webhook``"ボタンをクリックします。

 - ``WARNING``:  エラー：``The URL you've entered is not valid``が表示される場合は、貼り付けたURLを確認して``spaces`` を削除してください。

GGitHubはすぐにwebhookをテストし、次のページに結果を表示します。tickWebhookの隣に表示される場合、それは機能していることを意味します（注意、ページを更新する必要があるかもしれません）。

---
これで演習は終わりです。

この演習では、GitHubのリポジトリにコードをコミットするたびにビルドが自動的にトリガーされるようにwebhookを作成しました。


[indexへ戻る](../index-aws.ja.md)