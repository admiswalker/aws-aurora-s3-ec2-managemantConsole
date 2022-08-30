# マネジメントコンソールを仕様した作成手順

Management Console から，EC2 と S3 と Aurora を立ち上げる．

恐らく，デフォルト VPC を作成してやるのが，一番早い．（デフォルトといいつつ，他の操作で影響を受けている場合があるので，作り直すと上手く行く）．
[削除してしまったデフォルトVPCやサブネットを再作成する](https://tracl.cloud/archives/engineerblog/aws-def-vpc-subnet-del)

## VPC の作成

### VPC の作成
- 名前: example_2208
- 手動: 192.168.0.0/23
<!--
足りない
- 手動: 192.168.0.0/24
-->

### subnet の作成
#### public subnet
- 名前: example_2208_public
- AZ: ap-northeast-1a
- IPv4 CIDR ブロック: 192.168.0.0/25
#### private subnet
- 名前： example_2208_private
- AZ: ap-northeast-1a
- IPv4 CIDR ブロック: 192.168.0.128/25
#### private subnet-2
- 名前： example_2208_private_2
- AZ: ap-northeast-1c
- IPv4 CIDR ブロック: 192.168.1.0/25

## IGW の作成とアタッチ
### IGW の作成
VPC の IGW の項目から作成できる．
- 名前: example_2208
### IGW のアタッチ
個別の IGW の設定から VPC にアタッチできる．

<!--
## NAT GW の作成とアタッチ
### NAT GW の作成
VPC の NAT GW の項目から作成できる．
- 名前: example_2208
- サブネット: example_ec2_2208_public
- 接続タイプ: パブリック
- ElasticIP: [Elastic IP を割り当て] ボタンを押す
-->

## メインルートテーブルの設定 (public)
### ルートテーブルの作成
VPC のルートテーブルの項目から作成できる．
- 名前: example_2208_public
- VPC: example_2208_public
- ルート追加
    - 送信先: 0.0.0.0/0
    - ターゲット: igw-xxxxxxxxxxxxxxxxxxxx
### カスタムルートテーブルの追加
- example_2208_public を VPC に明示的に関連付ける

## メインルートテーブルの設定 (private)
### ルートテーブルの作成
VPC のルートテーブルの項目から作成できる．
- 名前: example_2208_private
- VPC: example_2208_private
<!--
- ルート追加
    - 送信先: 0.0.0.0/0
    - ターゲット: nat-xxxxxxxxxxxxxxxxxxxx
-->
### カスタムルートテーブルの追加
- example_2208_private を VPC に明示的に関連付ける

## メインルートテーブルの設定 (private)
### ルートテーブルの作成
VPC のルートテーブルの項目から作成できる．
- 名前: example_2208_private_2
- VPC: example_2208_private
### カスタムルートテーブルの追加
- example_2208_private_2 を VPC に明示的に関連付ける

## EC2 の作成
### インスタンスの起動
- 名前: example_2208
- OS: Amazon Linux
- instance type: t3.micro
- storage type: gp3

- ネットワーク
    - VPC: example_2208
    - サブネット: example_2208_public
    - パブリックIPの自動割り当て: 有効化
    - セキュリティグループを作成する: example_2208_ec2_sg

## RDS 用のセキュリティグループ
- 名前: example_2208_rds_sg
- VPC: example_2208
- インバウンドルール
    - MySQL/Aurora
    - ソース: example_2208_ec2_sg

## サブネットグループの作成
マネジメントコンソール → RDS → サブネットグループ → [DB サブネットグループ作成] ボタンをクリック

- 名前: example_2208_rds_subnet_group
- VPC: example_2208
- サブネットを追加
    - AZ: ap-northeast-1a, ap-northeast-1c, ap-northeast-1d (チェックした AZ の subnet が次の項目で選択肢に表示される)
    - サブネット
        - example_2208_private
        - example_2208_private_2

### インバウンドルール
- SSH: EC2 の作成時に同時に作るので，ここでは作らない

## Aurora の作成と EC2 からの接続
### Aurora の作成
- データベースの作成方法: 標準作成
- エンジンタイプ: Amazon Aurora
    - エディション: Amazon Aurora MySQL 互換
    - バージョン: 5.7
- テンプレート: 
    - 開発/テスト
- 設定
    - DB クラスター識別子: database-1
    - マスターユーザ名: admin
    - pass: adminpass
- DB インスタンスクラス
    - バースト可能クラス (t クラスを含む)
        - t2.small: (vCPU: 1, memory: 2 GiB) 0.023 USD/h
        - t3.small (vCPU: 2, memory: 2 GiB): 0.0208 USD/h
- 接続
    - Don’t connect to an EC2 compute resource
        - Virtual Private Cloud (VPC): example_2208
    - サブネットグループ: example_2208_rds_subnet_group
    - パブリックアクセス: なし
    - VPC セキュリティグループ
        - 既存の選択
            - 既存の VPC セキュリティグループ
                - example_2208_rds_sg
    - アベイラビリティーゾーン
        - 指定なし
    - RDS Proxy
        - なし

## エラーメモ
- ご指定になった DB クラスター database-1-instance-1 の作成リクエストは実行されませんでした。
  The DB subnet group doesn't meet Availability Zone (AZ) coverage requirement. Current AZ coverage: ap-northeast-1a. Add subnets to cover at least 2 AZs. (Service: AmazonRDS; Status Code: 400; Error Code: DBSubnetGroupDoesNotCoverEnoughAZs; Request ID: c426a8b9-1b4a-4b51-8696-613a80539a32; Proxy: null)
- ご指定になった DB クラスター database-1-instance-1 の作成リクエストは実行されませんでした。
  No default subnet detected in VPC. Please contact AWS Support to recreate default Subnets.

### EC2 からの接続
- mysql client の install
  ```
  sudo yum install mysql
  ```
- 接続
  ```
  mysql -u admin -p -h [Endpoint of RDS]
  ```
  例: 
  ```
  mysql -u admin -p -h database-1.cluster-xxxxxxxxxxxx.ap-northeast-1.rds.amazonaws.com
  mysql -u admin -p -h database-1.cluster-c6dtejpehxyt.ap-northeast-1.rds.amazonaws.com
  ```

## S3 の作成と EC2 からの接続
### S3 bucket の作成
- bucket name: example2208
- デフォルト暗号化: 有効
### iam で ec2 に権限追加
IAM → ロール → ロールを作成
1. 信頼されたエンティティを選択
   - AWS のサービス
   - 一般的なユースケース
       - EC2
2. 許可を追加
   - AmazonS3FullAccess
3. 名前，確認，および作成
   - ロール名: example2208_s3fullaccess
### iam ロールのアタッチ
1. 対象の EC2 に入る
2. アクション → セキュリティ → IAM ロールを変更
   - IAM ロールを選択: example2208_s3fullaccess

### 動作確認
```
aws s3 ls
```

### end point の作成
vpc → エンドポイント → エンドポイントの作成
- 名前: example2208
- サービス名: com.amazonaws.ap-northeast-1.s3 (タイプ: Gateway)
- VPC: EC2 とか RDS と同じ VPC を選択する

---

- 参考資料
    - インターネットゲートウェイを介して EC2 に接続する時に参考にした資料
        - [AWS NAT構成の作り方(NATゲートウェイ編)](https://qiita.com/TK1989/items/5d9bd5d49708c02dff3f)
        - [NAT ゲートウェイを設定してみる - EC2](https://qiita.com/leomaro7/items/52147ee88c6da11048e2)
    - RDS 構築で参考にした資料
        - [【初心者向け】EC2からRDSへ接続してみた](https://blog.serverworks.co.jp/ec2-to-private-rds)
    - EC2 から S3 にアクセスする時に参考にした資料
        - [EC2からS3へアクセスする4つのルートとコスト](https://tech.nri-net.com/entry/access_routes_from_EC2_to_S3)
        - [VPCエンドポイントを試してみた](https://qiita.com/Jerid/items/5dd072b1f88a7913f99c#s3%E3%81%AE%E4%BD%9C%E6%88%90)
