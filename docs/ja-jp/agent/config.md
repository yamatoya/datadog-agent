# オプションの設定

このページでは、バージョン5と比較して、Agentバージョン6以降で対応したオプション設定について詳細を説明します。
ここに掲載されていないオプション設定がある場合、それは以下のケースかもしれません。

 * そのオプションはそのままサポートされています。その場合は、 [例のファイル][datadog-yaml] で確認できます。
 * そのオプションは現在開発中の機能を参照しています。The option refers to a feature that's currently under developmen
 * そのオプションは計画されているが、まだ実装されていない機能を参照しています。

## 環境変数

メインの設定ファイル(`datadog.yaml`)でAgentがサポートしているオプションは全て、環境変数でも設定できます。以下のルールで使えます。

1. オプション名はアッパーケースで`DD_`プレフィックスつけます。 例: `hostname` -> `DD_HOSTNAME`

2. ネストした設定オプションはアンダースコアで結合します。 例:
   ```yaml
   cluster_agent:
     cmd_port: <some_value>
   ```
   -> `DD_CLUSTER_AGENT_CMD_PORT=<some_value>`

   例外: 今の所、`apm_config` と `process_config`配下にあるネストしたオプションのみ環境変数で設定できます。

3. 設定値の一覧は、スペースで分割します。 例:
   ```yaml
   ac_include:
     - "image:cp-kafka"
     - "image:k8szk"
   ```
   -> `DD_AC_INCLUDE="image:cp-kafka image:k8szk"`

4. _任意の値_, _ユーザー定義_ キーのマップ構造が予想されるオプションは、JSON形式にします。 例:
   ```yaml
   docker_env_as_tags:
     ENVVAR_NAME: tag_name
   ```
   -> `DD_DOCKER_ENV_AS_TAGS='{ "ENVVAR_NAME": "tag_name" }'`

   補足: このルールは、事前に定義されたキーによるマップ構造のオプションには適用されません。それらのオプションの場合は、ルール`2`を参照してください。

補足: 設定ファイルの設定オプションで指定した_全ての_ネストしたオプションは、環境変数で指定したネストしたオプションで上書かれます。一つだけ例外があります。: `proxy` 設定オプションです。プロキシ設定の詳細については [the dedicated Agent proxy documentation](https://docs.datadoghq.com/agent/proxy/#agent-v6)を参照してください。

## オーケストレーション + Agent 管理

オーケストレーションは可能な限り延期されます。この目的のために、Linuxではupstart/systemd、
WIndowsではWindowsサービスを使用します。Agentに同梱されているAPMとProcessエージェントを有効にするには、
メインの設定ファイル（`datadog.yaml`）の設定フラグ経由で設定します。

### Process Agent
process agentを有効にするために `datadog.yaml`に次の設定を追加します:
```
...
process_config:
  enabled: true
...
```

### Trace Agent
trace agentを有効にするために `datadog.yaml`に次の設定を追加します:
```
...
apm_config:
  enabled: true
...
```

OSレベルのサービスは既定で全てのAgentで有効になっています。Agentは設定を処理し、起動するか、正常に終了するかを
決定します。OSレベルのサービスを無効にするかもしれません。Agentを再度有効にしたい場合は、手作業で起動すする必要があります。

## version 6の新しいオプション

新規、名前変更、何らかの変更された設定の一覧です。

| 旧名称 | 新名称 | 備考 |
| --- | --- | --- |
| `proxy_host`  | `proxy`  | Proxy設定は、`http://user:password@proxyurl:port`のようにURIリストで表現します。(詳細は[datadog.yaml][datadog-yaml]の`proxy`セクションを参照してください。) |
| `collect_instance_metadata` | `enable_metadata_collection` | 新しいメタデータコレクションメカニズムを有効にします |
| `collector_log_file` | `log_file` ||
| `syslog_host`  | `syslog_uri`  | Syslog設定は、URIとして表現されます。 |
|| `syslog_pem`  | TLS クライアント検証用のSyslog設定のクライアント証明書 |
|| `syslog_key`  | TLS クライアント検証用のSyslog設定のクライアントプライベートキー |


## インスタンス設定への統合

特定の統合オプションに加え、Agentは、`instance`セクションで次の拡張オプションをサポートしています。

* `min_collection_interval`: デフォルトの15秒よりも短い間隔で確認すべきチェックのために、異なる間隔を秒で設定できます。
* `empty_default_hostname`: `true`に設定した時、メトリックやイベント、サービスチェックの投稿時に、ホスト名を付与しません。
* `tags`: チェックで送信されたタグにカスタムタグを追加して送信します。

## 削除されたオプション

これは新しいAgentで削除された設定オプションの一覧です。
以下のどちらかの理由です。
* 新しいオプションで置き換えられたもの
* バージョン5から異なる挙動になった機能に関するもの

| Name | Notes |
| --- | --- |
| `proxy_port` | `proxy`に置換しました |
| `proxy_user` | `proxy`に置換しました |
| `proxy_password` | `proxy`に置換しました |
| `proxy_forbid_method_switch` | 廃止 |
| `use_mount` | v5で廃止されました |
| `use_curl_http_client` | 廃止 |
| `exclude_process_args` | 廃止された機能 |
| `check_timings` | superseded by internal stats |
| `dogstatsd_target` | |
| `dogstreams` | |
| `custom_emitters` | |
| `forwarder_log_file` | `log_file`に置換しました |
| `dogstatsd_log_file` | `log_file`に置換しました |
| `jmxfetch_log_file` | `log_file`に置換しました |
| `syslog_port` | `syslog_uri`に置換しました |
| `check_freq` | |
| `collect_orchestrator_tags` | 現在はメタデータ収集で実装され機能です |
| `utf8_decoding` | |
| `developer_mode` | |
| `use_forwarder` | |
| `autorestart` | |
| `dogstream_log` | |
| `use_curl_http_client` | |
| `gce_updated_hostname` | v5で `gce_updated_hostname`をtrueにした時と同じような動作をv6でhします。May affect reported hostname, see [doc][gce-hostname] |
| `collect_security_groups` | v6はセキュリティーグループホストタグを収集しません。機能はAWSインテグレーションでまだ提供されています。  |

[datadog-yaml]: https://raw.githubusercontent.com/DataDog/datadog-agent/master/pkg/config/config_template.yaml
[gce-hostname]: changes.md#gce-hostname
