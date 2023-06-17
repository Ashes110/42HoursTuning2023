# 競技マニュアル

## 競技の開始

VM 環境での競技開始手順の説明をします。

- 秘密鍵：{動物}\_ssh_key.pem という形式のファイル（各チームに配布予定です。）
- VM のドメイン：env-{動物}.ftt2306.dabaas.net（上記秘密鍵の名前から特定できます。）

1. 配布された秘密鍵のアクセス権限を適切に設定してください。

```
$ chmod 400 {秘密鍵のパス}
```

2. 秘密鍵に対応した VM のドメインを利用して、環境にログインしてください。

```
$ ssh -i {秘密鍵のパス} azureuser@{VMのドメイン}
```

3. ホームディレクトリにある entry.sh を実行し、fork したリポジトリの URL を入力、実行してください。docker コンテナが起動し、初期データが投入されます。

```
$ bash entry.sh
forkしたリポジトリのURLを入力ください: https://github.com/your-name/42HoursTurningTheBackend.git
```

「初期化に成功しました。」という出力がされていれば、無事に VM 環境での開発準備ができています。

4. リポジトリ内に移動し、実際に評価スクリプトを動かしてみてください。

```
$ cd 42HoursTuning2023
$ bash run.sh
```

リポジトリが clone され、docker コンテナが起動・初期データが投入されます。チューニングしたコードの実行、採点を行ってください。

注意: fork 元の(Dreamarts 所属の)リポジトリを使用することはできません。例えば、以下の URL を使うことはできません。  
`https://github.com/DreamArts/42HoursTuningTheBackend.git`  
`https://github.com/DreamArts/42HoursTuningTheBackend`

## 投入されているユーザーデータ

- mysql の user テーブルに、ユーザーのデータが登録されています。
- ユーザーは 約 30 万人登録されています。
- 各ユーザーは、メールアドレス: `popy${数字}@example.com`、パスワード: `pass${数字}`でログイン可能です。
- テスト用のセッションが mysql の session テーブルに登録されており、`session_id`が`test-session-id`、`linked_user_id`が`test-user-id`となっています。
- このレコードを削除すると、負荷試験がログイン API 以外失敗するため、誤って消してしまった場合はリストアを試すか、session テーブルに再度レコードを登録してください。

## 最終スコアの提出

VM 環境で採点が成功したものは、運営に結果が送信されます。採点は何度も実施可能で、レギュレーションを違反していない一番高いスコアが最終結果で利用されます。

## スクリプトの紹介

競技に必要、または利用できそうなスクリプトを用意しています。それぞれの概要を説明します。

注意:

- すべて、スクリプト配置フォルダ内で直接実行してください。
- ファイルを削除するスクリプトが含まれます。実行環境にお気をつけください。

### 初期設定スクリプト

場所: リポジトリ外（`/home/azureuser`）

入力したリポジトリをクローンし、そのまま init.sh を実施します。
init.sh で必要なファイルを`/.da`ディレクトリからリポジトリ内にコピーし、restore_and_migration.sh を実施します。

```
$ bash entry.sh
forkしたリポジトリのURLを入力ください: https://github.com/your-name/42HoursTurningTheBackend.git
```

### 評価スクリプト

場所: `ルートディレクトリ`

リストア・コンテナ再起動・マイグレーション・e2e テスト・負荷試験・採点を順次実行します。

上記ステップを各種実行できるスクリプトは各種項目を参考にしてください。

制約及び注意:

- ローカル環境だと e2e テストがタイムアウトする可能性が高いため、評価スクリプト内ではスキップされます。ローカル環境で e2e テストを実施したい場合は個別のスクリプトを実行してください。

```
$ bash run.sh
```

### コンテナ再起動

場所: `app/`

採点時に実施される`app/`の現在の内容でイメージおよびコンテナを作成し、サービスを立ち上げます。

```
$ cd app
$ bash restart_container.sh
```

### リストア & マイグレーション

場所: `benchmarker/`

採点時に実施される MySQL データのリストア・コンテナ再起動・簡易 DB マイグレーションを順次実施します。

実行には数分かかります。

```
$ cd benchmarker
$ bash restore_and_migration.sh
```

### API テスト

場所: `benchmarker/`

採点時に実施される API テストを制約付きで実行できます。

最新の内容でテストしたい場合は、別途リストア & マイグレーションスクリプトを実行してください。

制約及び注意:

- 初期状態だと稀にタイムアウトする可能性があります。
- 評価スクリプトとは異なり、[http://localhost/]()宛にテストを実施します。
- このスクリプトの結果は OK だが採点スクリプトではテスト結果が NG の場合、後者が優先されます。

```
$ cd benchmarker
$ bash e2e.sh
```

### 負荷試験 & 採点

場所: `benchmarker/`

現在起動しているサービスに対し、採点を行います。実行には数分かかります。

最新の内容で採点したい場合は、リストア & マイグレーションスクリプトを実行してください。

```
$ cd benchmarker
$ bash run_k6_and_score.sh
```

## 簡易 DB マイグレーション機能

評価スクリプトを実行する度にデータベースの状態は戻ってしまいますが、あらかじめ登録された SQL を実行させ、テーブル等への変更を採点前に反映させることができます。

`app/`ディレクトリの`mysql/mysql_migration/`に置かれた{数字}\_\*.sql ファイルは、採点前に実行されます。
0_name.sql, 1_name.sql...という名称のファイルを置いておくことで、番号が若い順に実行されていきます。

## 競技中及び競技環境に関する注意

- 競技環境の HDD の容量はそこまで潤沢ではありません。不要なリソースは削除するようにしてください。
- リポジトリのディレクトリ構成を変えないようにしてください。スクリプトが動作しなくなる場合があります。
- `/.da`領域は変更しないでください。動作しなくなる場合があります。また、中身の持ち出しも禁止です。
- OS をシャットダウンした場合、参加者は SSH ログインできなくなります。ドリーム・アーツ社員でないと復旧できないため時間をロスすることになります。
- Azure 障害により競技環境が一時的に使用できなくなった場合でも、課題提出締め切りの延長はありません。
- OS を書き換える等により競技環境が破損し環境再構築が発生した場合も、課題提出締め切りの延長はありません。

## 環境の制約事項

競技主催者側の都合上、やむを得ず現実的なシナリオと乖離している箇所があります。これを利用しても業務アプリケーションの改善とは言い難いため、あらかじめ共有します。

- 初期データに含まれる添付ファイルは同じ内容を使い回している
  - HDD 容量制限のため。
  - 同じファイルが参照されることがありますが、基本的には別の添付ファイルとして扱ってください
- API テストは、一部のテスト用のユーザ情報を利用して実施される
  - ユーザの識別はアクセス状況から容易にわかりますが、この情報を使って API テストだけ PASS する最適化は、レギュレーション違反です

## FAQ

- Q. 何もしていないのに、スコアが下がる
  - スコアは負荷試験による実測値をベースにする性質上、同一条件でも採点結果が多少上下します。
  - ある改善前後でスコアが下がっても、効果が全くなかったと断定できない場合があるので注意してください
- Q. データの整合性はどこまで気をつければいいですか。アプリが実行中にクラッシュしたシナリオの対応など。
  - まずは最低限 API テストが通るレベルにしておいてください。
  - 例えば負荷試験中の更新内容を全てメモリのみに保管し永続化しない場合、電源断で負荷試験中のデータは全て消失するため、これは業務アプリ向けの改善とは言い難いです。
- Q. 画像の圧縮や変換はどこまで認められますか？
  - API 仕様書に則っており、API テストが通っていれば ok。
- Q. データベースのテーブルの構造を変更しても良いですか？
  - 不可です。
- Q. ライブラリやツールを VM にインストールしても構いませんか？
  - OK です。
- Q. 追加で VM のファイアウォールのポートを開放できますか？
  - できません。HTTPS,SSH のみの開放です。
- Q. nginx や mysql を別のものに変更したり、backend を別の言語で実装しても良いですか？
  - OK です。
- Q. 評価スクリプトの実行を繰り返すと、スコアがどんどん下がっていく
  - 初期の実装では、DockerEngine に負荷がかかっている可能性があります。DockerEngine の再起動が有効な可能性があります。
- Q. どこまでが改善対象ですか？
  - 採点時の負荷試験は HTTPS 接続で各 API へリクエストを送ります。nginx や mysql、backend コンテナといった API の機能を成立させるためのモジュールは改善対象です
  - 要件を満たしていれば、これらのモジュールを全く差し替えてしまうことも可能です。
  - frontend コンテナ及びそれを実行するクライアントについては負荷試験のリクエスト対象に入っていないため改善しても効果はあまり見込めません
- Q. リストアされたデータの中身を見ること / 分析することは可能ですか？
  - 可能です。お客様からは許可は取っているものとします。
- Q. API テスト時にたくさん失敗する
  - API テストではそれぞれのテストの結果を利用して後続のテストを行なっている箇所があります
  - 前提となる項目が PASS しない場合、後続のテスト項目も(仕様に適合しているかにかかわらず)失敗する場合があります
  - あまり大幅に実装を変更してまとめて API テストを実施すると問題点がわかりにくくなるため、こまめに API テストを実施することをお勧めします。
- Q. 用意されたスクリプトは使わなくてもいいですか？
  - 採点は評価スクリプトの実行結果のみ認められますが、それ以外のスクリプトは好みに合わせて使い分けてください
- Q. 意図せずレギュレーション違反をしてしまった
  - 考慮する場合があります

## その他

### インフラ情報

競技環境は Azure で構築しています。

- VM: Standard D2as v4 (2 vcpu 数、8 GiB メモリ)
- ディスク: StandardHDD S4

### ローカル環境での開発

[こちらを参照](./98_local.md)

```

```