## Helix Authentication Extensionの導入

Helix Authentication Extensionの導入方法を記します。
参照: [Administrator's Guide for Helix Authentication Extension](https://github.com/perforce/helix-authentication-extension/blob/master/docs/Administrator-Guide-for-Helix-Authentication-Extension-v2019.1.md)

Helix Authentication Extensionは、Helix Core 2019.1以降で実装されたExtensionsの機能を必要とします。


1. 登録されているHelix Core Extensionの一覧を確認します。
   ```bash
   p4 extension --list --type=extensions
   ```

   Helix Authentication Extensionを未導入の場合は、`extension Auth::loginhook`を含むエントリが出力されません。
   Helix Authentication Extensionを導入済の場合は、以下のようなエントリが出力されます。
   ```
   ... extension Auth::loginhook
   ... rev 1
   ... developer Perforce
   ... description-snippet SSO auth integration
   ... UUID 117E9283-732B-45A6-9993-AE64C354F1C5
   ... version 1.0
   ... enabled true
   ... arch-dir server.extensions.dir/117E9283-732B-45A6-9993-AE64C354F1C5/1-arch
   ... data-dir server.extensions.dir/117E9283-732B-45A6-9993-AE64C354F1C5/1-data
   ... hasGlobalConfig true
   ... hasInstanceConfig true

   ```

1. GitHubからhelix-authentication-extensionを取得します。
   ```bash
   git clone https://github.com/perforce/helix-authentication-extension
   ```

1. パッケージを作成します。
   ```bash
   cd helix-authentication-extension
   p4 extension --package loginhook
   ls -al loginhook.p4-extension
   ```

   パッケージの作成に成功した場合は、`loginhook.p4-extension`が出力されます。
   ```
   Extension packaged successfully.
   -r--r--r--. 1 p4misc p4misc 10584  5月 30 17:30 loginhook.p4-extension
   ```

1. 作成されたパッケージをインストールします。
   ```bash
   p4 extension --install loginhook.p4-extension -y
   ```

   パッケージのインストールに成功した場合は、以下のログが出力されます。
   ```
   Extension 'Auth::loginhook' version '1.0' installed successfully.
   Perform the following steps to turn on the Extension:
   
   # Create a global configuration if one doesn't already exist.
   p4 extension --configure Auth::loginhook
   
   # Create an instance configuration to enable the Extension.
   p4 extension --configure Auth::loginhook --name Auth::loginhook-instanceName
   
   For more information, visit:
   https://www.perforce.com/manuals/v20.1/extensions/Content/Extensions/Home-extensions.html
   ```

1. Helix Coreを再起動します。
   ```bash
   p4 admin restart
   ```

1. グローバル設定をエクスポートします。
   ```bash
   p4 extension --configure Auth::loginhook -o > global_config.txt
   ```

1. グローバル設定を編集します。
   編集例:
   ```
   ExtP4USER: super
   ExtConfig:
        Auth-Protocol:
                saml
        Service-URL:
                https://auth-svc.example.com:3000/
   ```
   - `ExtP4USER`には、拡張機能の実行ユーザを表します。
   - `Auth-Protocol`には、使用するプロトコルに応じて`saml`または`oidc`と記述します。
   - `Service-URL`には、Helix Authentication ServiceのURLを記述します。

1. グローバル設定をインポートします。
   ```bash
   p4 extension --configure Auth::loginhook -i < global_config.txt
   ```

   記述内容に誤りがなければ、以下のログが出力されます。
   ```
   Extension config loginhook saved.
   ```

1. インスタンス設定をエクスポートします。
   ```bash
   p4 extension --configure Auth::loginhook --name loginhook-a1 -o > instance_config.txt
   ```
   - `--name`に与える文字列`loginhook-a1`は、別の分かりやすい名称にしても構いません。

1. インスタンス設定を編集する。
   編集例:
   ```
   ExtConfig:
           enable-logging:
                   true
           name-identifier:
                   nameID
           non-sso-groups:
                   admins
           non-sso-users:
                   super
           user-identifier:
                   email
   ```

   - `enable-logging`をtrueにすると、*P4ROOT*`/server.extensions.dir`に`log.json`という名称のログが出力されます。
     *P4ROOT*は、Helix Coreのメタデータが配置されているディレクトリのパスになります。
   - `name-identifier`には、IdP側のユーザを一意に識別することのできるフィールド名を記述します。例えばSAMLでは`nameID`を指定します。
   - `non-sso-groups`には、SSOの対象外とするHelix Coreのグループを列挙します。
   - `non-sso-users`には、SSOの対象外とするHelix Coreのユーザを列挙します。
   - `user-identifier`には、Helix Core側のユーザを一意に識別することのできるフィールド名を記述します。
     - Helix Coreのユーザ仕様に現れる`user`, `email`, `fullname`のどれかを設定します。
       Helix Coreのユーザ仕様の例:
       ```
       User:   super
       Email:  super@localhost
       FullName:       super
       ```
     - `name-identifier`で得られる値と一致する必要があり、例えば`email`を指定します。

1. インスタンス設定をインポートする。
   ```bash
   p4 extension --configure Auth::loginhook --name loginhook-a1 -i < instance_config.txt
   ```

   記述内容に誤りがなければ、以下のログが出力されます。
   ```
   Extension config loginhook-a1 saved.
   ```

1. Extensionに適用されている既存の設定の一覧を確認する。
   ```bash
   p4 extension --list --type configs
   ```

   以下のように、`type global-extcfg`、`type auth-check-sso`、`type auth-pre-sso`の3種類の設定が登録されていることが確認できます。
   ```
   ... config loginhook
   ... extension Auth::loginhook
   ... uuid 117E9283-732B-45A6-9993-AE64C354F1C5
   ... revision 1
   ... owner super
   ... type global-extcfg
   
   ... config loginhook-a1
   ... extension Auth::loginhook
   ... uuid 117E9283-732B-45A6-9993-AE64C354F1C5
   ... revision 1
   ... owner super
   ... type auth-check-sso
   ... arg auth
   
   ... config loginhook-a1
   ... extension Auth::loginhook
   ... uuid 117E9283-732B-45A6-9993-AE64C354F1C5
   ... revision 1
   ... owner super
   ... type auth-pre-sso
   ... arg auth
   ```

1. 登録されているExtensionの一覧を確認する。
   ```bash
   p4 extension --list --type=extensions
   ```

1. 登録されているHelix Core Extensionの一覧を確認します。
   ```bash
   p4 extension --list --type=extensions
   ```

   以下のようなエントリ`extension Auth::loginhook`が出力されます。
   ```
   ... extension Auth::loginhook
   ... rev 1
   ... developer Perforce
   ... description-snippet SSO auth integration
   ... UUID 117E9283-732B-45A6-9993-AE64C354F1C5
   ... version 1.0
   ... enabled true
   ... arch-dir server.extensions.dir/117E9283-732B-45A6-9993-AE64C354F1C5/1-arch
   ... data-dir server.extensions.dir/117E9283-732B-45A6-9993-AE64C354F1C5/1-data
   ... hasGlobalConfig true
   ... hasInstanceConfig true

   ```

1. SSOの対象外とするグループやユーザを設定するために、構成可能変数`auth.sso.allow.passwd`を設定します。
   `auth.sso.allow.passwd`は、Helix Coreのユーザデータベースを用いたパスワード認証を可能にします。
   ```bash
   p4 configure set auth.sso.allow.passwd=1
   ```



## Helix Authentication Serviceの導入

Helix Authentication Serviceの導入方法を記します。
参照: [Administrator's Guide for Helix Authentication Service](https://github.com/perforce/helix-authentication-service/blob/master/docs/Administrator-Guide-for-Helix-Authentication-Service-v2019.1.md)

1. GitHubからhelix-authentication-serviceを取得します。
   ```bash
   git clone https://github.com/perforce/helix-authentication-service
   ```

1. Helix Authentication Serviceをインストールします。
   ```bash
   cd helix-authentication-service
   ./install.sh
   ```
   sudoが実行できるユーザでインストールをしてください。

   インストールスクリプトが対応していない環境の場合は、次の調整で動作する場合があります。
   - 調整前:
     ```bash
     if [ -e "/etc/redhat-release" ]; then
         PLATFORM=redhat
     elif [ -e "/etc/debian_version" ]; then
         PLATFORM=debian
     else
         # Exit now if this is not a supported Linux distribution
         die "Could not determine OS distribution"
     fi
     ```
   - 調整後: (Amazon Linux 2の場合)
     ```bash
     if [ -e "/etc/redhat-release" ]; then
         PLATFORM=redhat
     elif [ -e "/etc/debian_version" ]; then
         PLATFORM=debian
     elif [ -e "/etc/system-release" ]; then
         PLATFORM=redhat
     else
         # Exit now if this is not a supported Linux distribution
         die "Could not determine OS distribution"
     fi
     ```

1. 連携するIdPの設定を行います。
   ```bash
   vi ecosystem.config.js
   ```

   SAMLで連携する場合は、以下を参考にして設定します。
   | 名前 | 説明 | デフォルト |
   | ----- | ----- | ----- |
   | IDP_CERT_FILE | 着信する SAML応答の署名を検証するために使用される、ID プロバイダの公開証明書を含むファイルのパス。これは必須ではないが、セキュリティの追加レイヤーとして機能する。 | なし |
   | SAML_IDP_SSO_URL | IdP Single Sign-On サービスの URL | なし |
   | SAML_IDP_SLO_URL | IdP シングルログアウトサービスの URL | なし |
   | SAML_SP_ISSUER | SAML IdP として Helix 認証サービスを使用するサービス・プロバイダの ID プロバイダ。 | urn:example:sp |
   | SAML_IDP_ISSUER | IDプロバイダのエンティティ識別子。これは必須ではありませんが、提供されると、着信ログアウト要求/応答に対して IdP 発行者が検証されます。 | - |
   | IDP_CONFIG_FILE | 認証サービスに接続する SAML サービスプロバイダを定義する設定ファイルのパス。 | 注意：認証サービスが SAML ID プロバイダとして動作している場合、認証サービスは auth サービス・インストール内の構成ファイルから設定の一部を読み取る。既定では、このファイルは saml_idp.conf.js という名前で、IDP_CONFIG_FILE 環境変数によって識別される。このファイルは、Node.js の require() |
   | SAML_SP_AUDIENCE | AudienceRestriction アサーションのサービス・プロバイダのオーディエンス値。 | なし |
   | SAML_AUTHN_CONTEXT | authn コンテキストは、ユーザが IdP で認証する方法を定義します。通常、ほとんどのシステムではデフォルト値で動作しますが、この値を変更する必要があるかもしれません。例えば、Azure では、特定のケースでは urn:oasis:names:tc:SAML:2.0:ac:classs:Password に設定する必要があるかもしれません。 | urn:oasis:names:tc:SAML:2.0:ac:classes:PasswordProtectedTransport |
   | SAML_NAMEID_FIELD | nameID がない場合に使用されるユーザ・プロファイルのプロパティの名前です。 | 注意: Node はファイルの内容をメモリにキャッシュするため、設定ファイルを変更するにはサービスを再起動する必要があります。 |
   | SAML_NAMEID_FORMAT | SAML ID プロバイダから期待される希望の NameID 形式。デフォルトは urn:oasis:names:tc:SAML:1.1:nameid-format:unspecified で、SAML 仕様で定義されている任意の形式に設定することができます。 | 注意: 指定されていない場合、サービスは電子メールとサブを試行し、それらが失敗した場合、サービスは一意の識別子を生成します。この値は、ユーザ・データの一意のキーとして使用される。ID プロバイダが返す生のユーザ・プロファイルを見るには、デバッグ・ロギングを有効にし（以下の DEBUG エントリを参照）、ログ出力で「レガシー設定 nameID」を確認する。 |
   | SVC_BASE_URI | Helix Authentication ServiceのNode.jsのURL | - |

1. サービスを再起動してIdPの設定を適用します。
   ```
   pm2 startOrReload ecosystem.config.js
   ```
