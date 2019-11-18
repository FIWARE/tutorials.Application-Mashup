[![FIWARE Banner](https://fiware.github.io/tutorials.Application-Mashup/img/fiware.png)](https://www.fiware.org/developers)

[![FIWARE Visualization](https://nexus.lab.fiware.org/repository/raw/public/badges/chapters/visualization.svg)](https://www.fiware.org/developers/catalogue/)
[![License: MIT](https://img.shields.io/github/license/fiware/tutorials.Application-Mashup.svg)](https://opensource.org/licenses/MIT)
[![Support badge](https://nexus.lab.fiware.org/repository/raw/public/badges/stackoverflow/wirecloud.svg)](https://stackoverflow.com/questions/tagged/fiware-wirecloud)
<br/> [![Documentation](https://img.shields.io/readthedocs/fiware-tutorials.svg)](https://fiware-tutorials.rtfd.io)

<!-- prettier-ignore -->

このチュートリアルは、[FIWARE WireCloud](https://Wirecloud.rtfd.io) の紹介です。プログラミング・スキルのない
エンドユーザが Web アプリケーションを作成し、NGSI データを視覚化するためのダッシュボードを作成できるようにする
Generic Enabler の視覚化ツールです。このチュートリアルでは、WireCloud ワークスペースを作成してデータを視覚化する
ためのウィジェットをアップロードする方法について説明します。ウィジェットが設定されると、データが画面に表示されます。

このチュートリアルでは、WireCloud の GUI のみを使用した対話の例を示します。WireCloud は、プログラミング・スキルが
限られているユーザでも、あらゆるタイプのユーザが使用できるように設計されているため、チュートリアル自体には
プログラミングは含まれていません。しかし、解説は、すべての FIWARE アーキテクチャに共通のさまざまなプログラミングの
原則および標準概念を引き続き参照しています。

独自のウィジェットを開発および作成する方法をカバーする追加の資料は、後のチュートリアルの主題になります。

## コンテンツ

<details>
<summary><strong>詳細</strong></summary>

-   [マッシュアップを使った NGSI データの視覚化](#visualizing-ngsi-data-using-a-mashup)
-   [前提条件](#prerequisites)
    -   [Docker](#docker)
    -   [Cygwin](#cygwin)
-   [アーキテクチャ](#architecture)
    -   [WireCloud 設定](#wirecloud-configuration)
-   [起動](#start-up)
    -   [サインイン](#sign-in)
-   [WireCloud にリソースを追加](#adding-resources-to-wirecloud)
    -   [ウィジェットのアップロード](#upload-widgets)
    -   [ワークスペースの作成](#creating-a-workspace)
-   [マッシュアップ・アプリケーションの作成](#creating-application-mashups)
    -   [シンプル・マッシュアップの作成](#creating-a-simple-mashup)
        -   [ウィジェットとオペレータの選択](#selecting-widgets)
        -   [NSGI Browser Widget](#nsgi-browser-widget)
    -   [マッシュアップ内の複数のウィジェットの結合](#combining-multiple-widgets-within-a-mashup)
        -   [ウィジェットとオペレータの選択](#selecting-widgets-and-operators)
        -   [NGSI Source Operator](#ngsi-source-operator)
        -   [NGSI Entity to POI Operator](#ngsi-entity-to-poi-operator)
        -   [Open Layers Map Widget](#open-layers-map-widget)
-   [次のステップ](#next-steps)
    - [License](#license)


</details>

<a name="visualizing-ngsi-data-using-a-mashup"></a>

# マッシュアップを使った NGSI データの視覚化

> "One picture is worth a thousand words."
>
> — Fred R. Barnard (Printers' Ink)

スマート・ソリューションが進化するにつれて、適切な決定を下すことができるように、現在のシステムのコンテキストを分析
および理解できることが必要です。データ分析の最初のステップは、コンテキスト・データを画面に表示することです。
この段階では、どのコンテキスト・データが関連しているのか、またエンドユーザにどのように表示されるのが最もよいかは
必ずしもわかりません。柔軟なラピッド・プロトタイピングは、データの改良、強化、操作、ビジュアライゼーションの表示と調整、
そして必要に応じてさらなる情報の追加を可能にするために必要です。

**アプリケーション・マッシュアップ**は、複数のソースからのコンテンツを使用してグラフィカル・インターフェイスで単一の
新しいサービスを表示する Web アプリケーションです。ほとんどのマッシュアップは設計上、視覚的で対話式であり、多くは短命の
表現 (short-lived representations) であり、単一の問題を分析するためにのみ必要です。<sup>[1](#footnote1)</sup>

[FIWARE WireCloud](https://Wirecloud.rtfd.io) Generic Enabler は、ユーザが NGSI やその他のデータソースに基づいて新しい
アプリケーション・マッシュアップを迅速に生成するのに役立つツールです。開発をスピードアップするために、WireCloud
アーキテクチャはマッシュアップ操作を一連の単純な再利用可能なタスク (ウィジェットとオペレータ) に分割するように
定義されています。各タスクには明確に定義された入力および出力インターフェースがあり、WireCloud UI により、マッシュアップ
作成者は一連のタスクを複雑な一連のデータ処理および視覚化イベントに関連付けることができます。

大まかに言って、アプリケーション・マッシュアップのタスクは4つのカテゴリに分けられます :

-   **データ・ソース** : これらは他の場所で消費するための情報を提供するオペレータです。たとえば、NGSI Web サービスから
    特定の種類の情報を取得するオペレータです
-   **データ・ターゲット** : 上記の逆です。これらのオペレータは、外部のマイクロサービスに情報を送り出します
-   **データ・トランスフォーマ** : このタイプのオペレータは、WireCloud エコシステム内のチェーンのさらに下流にある他の
    タスクで使用可能にするためにデータを操作します。たとえば、フォームリストの値を置き換えたり、下流の入力インタフェースに      合わせて属性の名前を変更したりします
-   **ビジュアル・コンポーネント** : オンラインでブラウザにデータを表示する HTML と JavaScript の組み合わせです。
    WireCloud 内では、ビジュアル・コンポーネントはウィジェットと呼ばれています。

WireCloud の全体的な目的は、プログラミングの知識がなくても、ドラッグ・アンド・ドロップのインターフェイスを使用して
データの視覚化を作成できるようにすることです。幅広い既存のオープンソースの
[WireCloud ウィジェットとオペレータ](https://wirecloud.readthedocs.io/en/stable/widgets/)
はすでに利用可能であり、複雑な視覚化を作成するために使用することができます。

既存の[ウィジェットとオペレータのセット](https://wirecloud.readthedocs.io/en/stable/widgets/)は幅広いシナリオをカバー
しますが、あなた自身の追加のウィジェットで補完することができます。この場合、JavaScript と HTML の知識が必要です。
あなた自身のウィジェットを作成することは、この後のチュートリアルの主題になるでしょう。

マッシュアップがワイヤリングされて作成されたら、それをエンドユーザと共有することもできます。

<a name="prerequisites"></a>

# 前提条件

<a name="docker"></a>

## Docker

簡単にするために、すべてのコンポーネントは [Docker](https://www.docker.com) を使用して実行されます。**Docker** は、
さまざまなコンポーネントをそれぞれの環境に分離できるようにするコンテナ・テクノロジです。

-   Windows に Docker をインストールするには、[こちら](https://docs.docker.com/docker-for-windows/)の指示に従ってください
-   Mac に Docker をインストールするには、[こちら](https://docs.docker.com/docker-for-mac/)の指示に従ってください
-   Linux に Docker をインストールするには、[こちら](https://docs.docker.com/install/)の手順に従ってください

**Docker Compose** は、マルチコンテナ Docker アプリケーションを定義して実行するためのツールです。
[YAMLファイル](https://raw.githubusercontent.com/Fiware/tutorials.Application-Mashup/master/docker-compose.yml)
を使用して、アプリケーションに必要なサービスを設定します。これは、すべてのコンテナ・サービスを単一のコマンドで起動
できることを意味します。Docker Compose は、Docker for Windows および Docker for Mac の一部としてデフォルトでインストール
されますが、Linux ユーザは[こちら](https://docs.docker.com/compose/install/)にある手順に従う必要があります。

<a name="cygwin"></a>

## Cygwin

簡単な bash スクリプトを使ってサービスを開始します。Windows ユーザは、Windows 上の Linux ディストリビューションに
似たコマンドライン機能を提供するために [cygwin](http://www.cygwin.com/) をダウンロードするべきです。

<a name="architecture"></a>

# アーキテクチャ

このアプリケーションは、[前のチュートリアル](https://github.com/FIWARE/tutorials.IoT-Agent/) で作成された既存の
在庫管理およびセンサ・ベースのアプリケーションに **WireCloud** アプリケーション・マッシュアップを追加します。
このチュートリアルの目的は、デバイスを監視し、簡単なスーパーマーケット・ファインダーを接続できるようにすることです。
このモニタリング・ツールのマッシュアップは、PUG, Node.JS および JavaScript で書かれているチュートリアル・アプリケーション
自体にある視覚化機能の多くを複製して置き換えることができます。目的は、コードを書くことに頼らずに同等のアプリケーション
を作成することです。

**WireCloud** のユーザは、標準の [ID 管理](https://github.com/FIWARE/tutorials.Identity-Management/)コンポーネントの
**Keyrock** を使用して作成されています。全体としてシステムは4つの FIWARE コンポーネントを利用します -
[Orion Context Broker](https://fiware-orion.readthedocs.io/en/latest/),
[IoT Agent for UltraLight 2.0](https://fiware-iotagent-ul.readthedocs.io/en/latest/),
[Keyrock](https://fiware-idm.readthedocs.io/en/latest/) ID 管理 および新しく統合された
[WireCloud](https://wirecloud.readthedocs.io/en/stable/) アプリケーション・マッシュアップ・ツールです。
アプリケーションが _"Powered by FIWARE"_ の資格を得るには、Orion Context Broker の使用で十分です。

したがって、アーキテクチャ全体は次の要素で構成されます :

-   [NGSI](https://fiware.github.io/specifications/OpenAPI/ngsiv2) を使用してリクエストを受信する
    FIWARE [Orion Context Broker](https://fiware-orion.readthedocs.io/en/latest/)
-   [NGSI](https://fiware.github.io/) を使用してサウスバウンドのリクエストを受信し、それらをデバイス用の
    [UltraLight 2.0](https://fiware-iotagent-ul.readthedocs.io/en/latest/usermanual/index.html#user-programmers-manual)
    コマンドに変換する FIWARE [IoT Agent for UltraLight 2.0](https://fiware-iotagent-ul.readthedocs.io/en/latest/)
-   FIWARE [Keyrock](https://fiware-idm.readthedocs.io/en/latest/) ID 管理システム
    -   **在庫管理システム**と **WireCloud** の両方で使用
-   NGSIエンティティを表示するための FIWARE [WireCloud](https://wirecloud.readthedocs.io/en/stable/) アプリケーション・
    マッシュアップ・ツール
-   3つのデータベース

    -   [PostgreSQL](https://www.postgresql.org/) データベース :
        -   マッシュアップ状態を保持するために**WireCloud** によって使用されます
    -   [MySQL](https://www.mysql.com/) データベース :
        -   ユーザの ID, アプリケーション, ロール, パーミッションを保持するために **Keyrock** によって使用されます
    -   [MongoDB](https://www.mongodb.com/) データベース :
        -   **Orion Context Broker** によって使用され、データ・エンティティ、サブスクリプション、レジストレーション
            などのコンテキスト・データ情報を保持します
        -   デバイスの URL やキーなどのデバイス情報を保持するために **IoT Agent** によって使用されます

-   **在庫管理フロンドエンド**は以下のことを行います :
    - 店舗情報を表示します
    - 各店舗で購入できる商品を表示します
    - ユーザが製品を "購入" し、在庫数を減らすことができます
-   HTTP 経由で実行されている
    [UltraLight 2.0](https://fiware-iotagent-ul.readthedocs.io/en/latest/usermanual/index.html#user-programmers-manual)
    プロトコルを使用する [ダミー IoT デバイス](https://github.com/FIWARE/tutorials.IoT-Sensors)のセットとして機能する
    Web サーバです - 特定のリソースへのアクセスが制限されています
-  **WireCloud** では、3つの追加マイクロサービスが使用されています :
    -  [Memcache](https://memcached.org) は、汎用の分散メモリ・キャッシュ。システムです
    -  [ElasticSearch](https://www.elastic.co/products/elasticsearch) は、全文検索エンジンです
    -  [NGSI Proxy](https://github.com/conwetlab/ngsi-proxy) は、**Orion** からの通知を Web ページにリダイレクトする
       ことができるサーバです

要素間のやり取りはすべて HTTP リクエストによって開始されるため、エンティティをコンテナ化して公開ポートから実行することが
できます。

![](https://fiware.github.io/tutorials.Application-Mashup/img/architecture.png)

チュートリアルの各セクションの特定のアーキテクチャについては、以下で説明します。

<a name="wirecloud-configuration"></a>

## WireCloud 設定

```yaml
image: fiware/wirecloud
        container_name: fiware-wirecloud
        hostname: wirecloud
        ports:
            - "8000:8000"
        networks:
          default:
            ipv4_address: 172.18.1.10

        restart: always
        depends_on:
            - keyrock
            - elasticsearch
            - memcached
            - postgres-db
        environment:
            - DEBUG=True
            - DEFAULT_THEME=wirecloud.defaulttheme
            - DB_HOST=postgres-db
            - DB_PASSWORD=wirepass
            - FORWARDED_ALLOW_IPS=*
            - ELASTICSEARCH2_URL=http://elasticsearch:9200/
            - MEMCACHED_LOCATION=memcached:11211
            - FIWARE_IDM_PUBLIC_URL=http://localhost:3005
            - FIWARE_IDM_SERVER=http://172.18.1.5:3005
            - SOCIAL_AUTH_FIWARE_KEY=wirecloud-dckr-site-0000-00000000000
            - SOCIAL_AUTH_FIWARE_SECRET=wirecloud-docker-000000-clientsecret
        volumes:
            - wirecloud-data:/opt/wirecloud_instance/data
            - wirecloud-static:/var/www/static
```

`wirecloud` コンテナは単一のポートで待機している Web アプリケーション・サーバです :

-   ポート `8000` は HTTP トラフィック用に公開されているため、Web ページを表示できます

`wirecloud` コンテナは **Keyrock** に接続しており、以下に示すように環境変数によって制御されます :

| キー                      | 値                                     | 説明                                                                                  |
| ------------------------- | -------------------------------------- | ------------------------------------------------------------------------------------- |
| DEFAULT_THEME             | `wirecloud.defaulttheme`               | 表示する WireCloud テーマ                                                             |
| DB_HOST                   | `postgres-db`                          | WireCloud データベースの名前                                                          |
| DB_PASSWORD               | `wirepass`                             | WireCloud データベースのパスワード - これは Docker Secrets によって保護されるべきです |
| FORWARDED_ALLOW_IPS       | `*`                                    |                                                                                       |
| ELASTICSEARCH2_URL        | `http://elasticsearch:9200/`           | ElasticSearch サービスがリッスンしている URL                                          |
| MEMCACHED_LOCATION        | `memcached:11211`                      | Memcahe サービスが待機している場所                                                    |
| FIWARE_IDM_PUBLIC_URL     | `http://localhost:3005`                | ログイン画面を表示するための **Keyrock** の URL                                       |
| FIWARE_IDM_SERVER         | `http://172.18.1.5:3005`               | OAuth2 認証に使用される **Keyrock** の URL                                            |
| SOCIAL_AUTH_FIWARE_KEY    | `wirecloud-dckr-site-0000-00000000000` | **WireCloud** の **Keyrock** によって定義されたクライアント ID                        |
| SOCIAL_AUTH_FIWARE_SECRET | `wirecloud-docker-000000-clientsecret` | **Keyclck** によって **WireCloud** のために定義されたクライアント・シークレット       |

<a name="start-up"></a>

# 起動

インストールを開始するには、次の手順に従います :

```console
git clone https://github.com/FIWARE/tutorials.Application-Mashup.git
cd tutorials.Application-Mashup

./services create
```

> **注 :** Docker イメージの初期作成には最大3分かかることがあります

その後、リポジトリ内の [services](https://github.com/FIWARE/tutorials.Application-Mashup/blob/master/services)
Bash スクリプトを実行して、すべてのサービスをコマンドラインから初期化できます :

```console
./services start
```

それからブラウザに行き、**WireCloud** で、URL `http://localhost:8000/` を開いてください

> :information_source: **注 :** クリーン・アップして最初からやり直す場合は、次のコマンドを実行してください :
>
> ```console
> ./services stop
> ```

<a name="sign-in"></a>

## サインイン

**WireCloud** は Oauth 2 セキュリティに対して有効になっています。また、前述のように、**WireCloud** アプリケーションの
admin ユーザは既に作成されています。マッシュアップの作成を開始するには、ページの右上にある **サインイン** ボタンを
クリックし、パスワード  `test` を指定した `alice-the-admin@test.com` を使って **Keyrock** にサインインします。

![](https://fiware.github.io/tutorials.Application-Mashup/img/login.png)

> **注 :** Alice がどのようにして **WireCloud** を使用する権限を与えられているかを知りたい場合は、**Keyrock**
> 自体にログインして、Alice のアプリケーション (`http://localhost:3005/idm/applications`) を見てください。
>
> **WireCloud** アプリケーションは以下のように設定されています :
>
> -   URL: `http://localhost:8000`
> -   Callback: `http://localhost:8000/complete/fiware/`
> -   アプリケーションは `admin` と呼ばれる単一のロールを持ちます
> -   `admin` ロール は、一つの空白のパーミッションを持ちます
> -   `admin` ロールが Alice に割り当てられました

<a name="adding-resources-to-wirecloud"></a>

# WireCloud にリソースを追加

上記のように、**WireCloud** は、NGSI ソースに接続し、データを操作し、画面に何かを表示するために、ウィジェットと
オペレータに依存しています。最初のステップとして、これらのウィジェット (`*.wgt`) を **WireCloud** にアップロードする
必要があります。これをプログラムで行うことは可能ですが、このチュートリアルでは、ユーザは手動でウィジェットをアップロード
するように指示されています。

<a name="upload-widgets"></a>

## ウィジェットのアップロード

リソースをアップロードするには、**My Resources** ボタンをクリックしてください :

![](https://fiware.github.io/tutorials.Application-Mashup/img/my-resources-button.png)

一連のリソースはすでに利用可能ですが、このチュートリアルでは追加のウィジェットとオペレータが必要です。
**Upload** ボタンをクリックしてください :

![](https://fiware.github.io/tutorials.Application-Mashup/img/upload-button.png)

次に、**Select files from your computer** をクリックします。

![](https://fiware.github.io/tutorials.Application-Mashup/img/upload-widgets.png)

WireCloud ウィジェットのソースは
[WireCloud Marketplace](https://wirecloud.readthedocs.io/en/stable/user_guide/#browsing-the-marketplace) にあります。
あるいは、個々のウィジェットのバイナリ (`*.wgt`) を GitHub の利用可能なリリースから直接ダウンロードすることができます。
一般的なウィジェットとその場所のリストは、
[WireCloud のドキュメント](https://wirecloud.readthedocs.io/en/stable/widgets/) の付録にあります。

このリポジトリのルートに移動し、`widgets` ディレクトリにあるすべてのファイルを以下のように選択します :

![](https://fiware.github.io/tutorials.Application-Mashup/img/upload-components-list.png)

Upload をクリックすると、リソースが **WireCloud** に追加されます。

![](https://fiware.github.io/tutorials.Application-Mashup/img/my-resources.png)

ウィジェットの使用状況の概要を取得するには、使用可能なウィジェットをクリックするだけです。利用可能な入力と出力、
そして各ウィジェットがどのようにワイヤリングできるかについての詳細はウィジェットのドキュメントにあります。

ホームページに戻るには、戻るボタンをクリックしてください :

![](https://fiware.github.io/tutorials.Application-Mashup/img/back-button.png)

<a name="creating-a-workspace"></a>

## ワークスペースの作成

個々のマッシュアップは異なるワークスペースで作成されるため、ユーザは異なる URLs で別々のビューを提供できます。
ワークスペースを作成するには、ハンバーガー・ボタンをクリックして新しいワークスペースを選択してください。

![](https://fiware.github.io/tutorials.Application-Mashup/img/new-workspace.png)

図のようにダイアログに入力して、空のワークスペースを作成します。

![](https://fiware.github.io/tutorials.Application-Mashup/img/create-workspace.png)

ワークスペースが開きます。ブラウザバーの URL は `http://localhost:8000/<user>/<workspace>` に変わります。

![](https://fiware.github.io/tutorials.Application-Mashup/img/workspace.png)

ワークスペースは、ハンバーガー・ボタンの下にある使用可能なワークスペースのリストにも追加されます。
利用可能な任意のワークスペースをクリックで選択できます。

![](https://fiware.github.io/tutorials.Application-Mashup/img/selecting-a-workspace.png)

戻るボタンをクリックしてホームページに戻ることができます :

![](https://fiware.github.io/tutorials.Application-Mashup/img/back-button.png)

<a name="creating-application-mashups"></a>

# マッシュアップ・アプリケーションの作成

アプリケーション・マッシュアップは、ワークスペースにウィジェットを追加することによって作成できます。最低でも1つの
データソースと1つのビジュアル・コンポーネントが必要ですが、一部のウィジェットはこれらの機能を単一のブラウザ・
ウィジェットに統合しています。

より複雑なマッシュアップは、WireCloud ワイヤリングを使用してリンクされた一連のウィジェットとオペレータで構成されます。

<a name="creating-a-simple-mashup"></a>

## シンプル・マッシュアップの作成

WireCloud コンポーネントの _Hello World_ では、画面に単一のブラウザ・ウィジェットを追加してから、コンテキスト・データ
を表示するようにウィジェットを設定します。

ご想像のとおり、このチュートリアルは Context Broker からのコンテキスト・データに依存しています。IoT デバイスが
実行されていることを確認するには、新しいブラウザ・ウィンドウでチュートリアル・アプリケーション
(`http://localhost:3000/`) を開きます。パスワード `test` で `bob-the-manager@test.com` としてログインし、
デバイス・モニタをクリックしてください。Bod はランプをオンにしてマッシュアップのコンテキスト・データを取得できます。

-   ![](https://fiware.github.io/tutorials.Application-Mashup/img/devices-lamp-on.png)

<a name="selecting-widgets"></a>

### ウィジェットとオペレータの選択

**Edit** ボタンをクリックしてワークスペースを編集モードに切り替え、**Find Components** ボタンをクリックして、
ワークスペースにウィジェットまたはオペレータを追加します。

![](https://fiware.github.io/tutorials.Application-Mashup/img/wiring-view.png)

Components のサイドバーが表示されます

<a name="nsgi-browser-widget"></a>

### NSGI Browser Widget

ビジュアル・コンポーネントが必要なので、Components のサイドバーで **Widgets** を選択し、NGSI Browser
が見つかるまで下にスクロールします

-   ![](https://fiware.github.io/tutorials.Application-Mashup/img/ngsi-browser-widget.png)

**+** ボタンをクリックすると、未設定の NGSI Browser が画面に追加されます

> **注 :** ウィジェットはウィンドウの右上にある **x** をクリックすることで削除できます

次のステップはウィジェットを設定することです。Wiring ボタンをクリックしてワイヤリング表示に切り替え、
Components のサイドバーから **Widgets** を選択します。NGSI Browser までスクロールすると、利用可能な
ウィジェットがオレンジ色で表示されます

![](https://fiware.github.io/tutorials.Application-Mashup/img/ngsi-browser-wiring.png)

オレンジ色のボタンをクリックすると、設定する設定が表示されます。ハンバーガー・メニューを選択して
ドロップ・ダウンを取得し、設定をクリックして以下のように設定します :

![](https://fiware.github.io/tutorials.Application-Mashup/img/ngsi-browser-settings.png)

次に **Accept** をクリックします

ワークスペース・ビューに戻るには、戻るボタンをクリックしてください :

![](https://fiware.github.io/tutorials.Application-Mashup/img/back-button.png)

NGSI Browser は次のようにライブ・データを表示しているはずです :

![](https://fiware.github.io/tutorials.Application-Mashup/img/ngsi-browser-ui.png)

<a name="combining-multiple-widgets-within-a-mashup"></a>

## マッシュアップ内の複数のウィジェットの結合

上の例の単純なウィジェットは、入力と出力の両方を単一のコンポーネントに結合していますが、WireCloud の真の力を見ることが
できるのは、一連のオペレータとウィジェットが**結合**されたときです。単純なデータ・フローを画面上にグラフィカルに
マッピングし、マッシュアップを作成するために使用されるテクノロジについて最小限の知識を持ってマッシュアップをユーザが
作成できます。

<a name="selecting-widgets-and-operators"></a>

### ウィジェットとオペレータの選択

ワークスペースのページから、Edit をクリックしてからジグソーパズル・ボタンをクリックして、
ワイヤリング・エディタ・ビューを開きます。

![](https://fiware.github.io/tutorials.Application-Mashup/img/add-components.png)

次に、Add components をクリックしてComponents のサイドバーを表示します。**Operators** と **Widgets**
の両方の組み合わせが必要になります

**operators** タブをクリックし、**+** ボタンを押して _NSGI Source_ のインスタンスと _NGSI-Entity-to-POI_ の
インスタンスを選択します。各エントリの下に緑色のバーが表示され、それらをワイヤリング・エディタ・ビューに
ドラッグ・アンド・ドロップできます。

次に **Widgetts** タブをクリックして _Open Layers Map_ Widget のインスタンスを追加し、
それをワイヤリング・エディタ・ビューにドラッグします。

| Operators                                                                         | Widgets                                                                               |
| --------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------- |
| ![](https://fiware.github.io/tutorials.Application-Mashup/img/operators-list.png) | ![](https://fiware.github.io/tutorials.Application-Mashup/img/components-widgets.png) |

その結果、一連の接続されていないコンポーネントが画面に表示されます。

![](https://fiware.github.io/tutorials.Application-Mashup/img/osm-unwired.png)

コネクタの上にカーソルを合わせると、利用可能な入力と出力に関する詳細情報が得られます。
図のように、要素をクリックし、ドラッグして接続します。

![](https://fiware.github.io/tutorials.Application-Mashup/img/osm-wired.png)

画面にデータの流れを視覚化できることがわかります。マッシュアップでは、関心のあるポイント (_POIs_) を画面に表示します。
これを行うために、NSGI source からデータ _entities_ を受け取ります。各 _entity_ からのデータは _POI_ に変換されます。
最後に各 _POI_ が _OpenLayers Map_ に挿入されます。

<a name="ngsi-source-operator"></a>

### NGSI Source Operator

_NGSI Source_ オペレータを使用すると、Orion Context Broker に接続してそれをデータのソースとして使用できます。これは、
関心のあるエンティティの変更に関するリアルタイムの通知を取得するためのサブスクリプションを作成することによって実現
されます。これはデータソース・コンポーネントの例です。

オペレータのドキュメントの詳細は、**My Resources** 画面の実行中のアプリケーション内にあります :
`http://localhost:8000/wirecloud/home#view=myresources&subview=details&resource=CoNWeT%2Fngsi-source%2F4.0.0&tab=Documentation`

コンポーネントを設定するには、_NSGI Source_ のハンバーガー・ボタンをクリックし、Settings を選択します :

![](https://fiware.github.io/tutorials.Application-Mashup/img/ngsi-source-wiring.png)

以下のように設定を修正して、Accept をクリックします。

![](https://fiware.github.io/tutorials.Application-Mashup/img/ngsi-source-settings.png)

-   NGSI サーバは Orion Context Broker の場所である必要があります
-   JavaScript を使用して Web サービスにアクセスするときに、NGSI proxy を使用して CORS の問題を回避します。
    これは、Orion Context Broker がサーバ・サイドの API 呼び出しを介してのみアクセスされることを保証する
    単純なエンドポイントです
-   Use the FIWARE credentials of the user と Use the FIWARE credentials of the Workspace owner
    はチェックしないでください
-   FIWARE-Service と FIWARE-ServicePath は、Orion Context Broker 内で見つかったデータと一致します
-   Id pattern, Query, Monitored NGSI Attributes は空白のままにできます

<a name="ngsi-entity-to-poi-operator"></a>

### NGSI Entity to POI Operator

_NGSI Entity to POI_ オペレータは、NGSI エンティティを POI (Point of Interest) に変換します。データ操作コンポーネント
の例です。そうするために、それらのエンティティはエンティティの座標を持つ属性を含むべきです。また、このオペレータが
一般的であるという事実を考慮に入れると、このオペレータによって作成された PoIs のマーカー・バブル (marker bubbles) は、
属性と値のペアの単なる合成になります。

オペレータのドキュメントの詳細は、**My Resources** 画面の実行中のアプリケーション内にあります :
`http://localhost:8000/wirecloud/home#view=myresources&subview=details&resource=CoNWeT%2Fngsientity2poi%2F3.1.2&tab=Documentation`

コンポーネントを設定するには、_NGSI Entity to POI_ のハンバーガー・ボタンをクリックし、Settings を選択します :

![](https://fiware.github.io/tutorials.Application-Mashup/img/ngsi-to-poi-wiring.png)

下に示すように座標設定を修正して、Accept をクリックします :

![](https://fiware.github.io/tutorials.Application-Mashup/img/ngsi-to-poi-settings.png)

FIWARE データ・モデルの
[ガイドライン](https://fiware-datamodels.readthedocs.io/en/latest/guidelines/index.html#modelling-location)
によれば、地理的座標には `location` という属性を使用する必要があります。座標は GeoJSON を使ってエンコードされるべき
です。この規約はチュートリアル・データに使用されているので、ウィジェットは `Store` エンティティから場所を抽出する
方法を知っています。


<a name="open-layers-map-widget"></a>

### Open Layers Map Widget

_Open Layers Map_ ウィジェットを使用すると、ユーザは [Open Layers](https://openlayers.org/) マップに地理データを
表示できます。これはビジュアル・コンポーネントの一例です。Open Layers は無料のオープンソースの JavaScript
ライブラリで、[2-clause BSD License](https://opensource.org/licenses/BSD-2-Clause) (2条項 BSD ライセンス) の下で
リリースされています。

ウィジェットのドキュメントの詳細は、実行中のアプリケーションの **My Resources** 画面の下にあります :
`http://localhost:8000/wirecloud/home#view=myresources&subview=details&resource=CoNWeT%2Fol3-map%2F1.1.2&tab=Documentation`

コンポーネントを設定するには、_Open Layers Map_ のハンバーガー・ボタンをクリックし、Settings を選択します :

![](https://fiware.github.io/tutorials.Application-Mashup/img/osm-wiring.png)

以下のように設定を修正して、Accept をクリックします。

-   ![](https://fiware.github.io/tutorials.Application-Mashup/img/osm-settings.png)

店舗データはベルリンにあります。この都市は 52.53N 13.4E に位置しています。Initial location (初期位置) の設定
は、Long/Lat 形式で定義されています。他の設定は、地球の右側に地図を表示するのに役立ちます。

更新されたマッシュアップは、ワークスペース・タブに表示されます。必要に応じてブラウザを更新してください。

`http://localhost:8000/alice/test#view=workspace&tab=tab`

![](https://fiware.github.io/tutorials.Application-Mashup/img/osm-map-result.png)

POIs をクリックすると、各店舗から追加のデータが取得されます。

-   ![](https://fiware.github.io/tutorials.Application-Mashup/img/osm-map-on-click.png)

現在データはフォーマットされていない JSON として表示されています。これは、チュートリアル例の `Store`
コンテキスト・データのエンティティが標準の FIWARE データモデルを使用していないためです。
[Building](https://fiware-datamodels.readthedocs.io/en/latest/Building/Building/doc/spec/index.html)
などの標準データモデルが使用されていた場合、データは適切な形式でフォーマットされま、**Building**
固有のアイコンが使用されます。

<a name="next-steps"></a>

# 次のステップ

高度な機能を追加することで、アプリケーションに複雑さを加える方法を知りたいですか？ このシリーズの
[他のチュートリアル](https://www.letsfiware.jp/fiware-tutorials)を読むことで見つけることができます

---

<a name="licensse"></a>

## License

[MIT](LICENSE) © 2019 FIWARE Foundation e.V.

---

### 脚注

<a name="footnote1"></a>

-   [Wikipedia: Mashup](https://en.wikipedia.org/wiki/Mashup_%28web_application_hybrid%29) - 複数のソースからのコンテンツを使用するWebアプリケーション
