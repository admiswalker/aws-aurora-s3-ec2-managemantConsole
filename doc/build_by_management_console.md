# マネジメントコンソールを仕様した作成手順

Management Console から，EC2 と S3 と Aurora を立ち上げる．

## VPC の作成

### VPC の作成
- 名前: example_ec2_2208
- 手動: 192.168.0.0/24

- 参考資料
    - [AWS NAT構成の作り方(NATゲートウェイ編)](https://qiita.com/TK1989/items/5d9bd5d49708c02dff3f)
    - [NAT ゲートウェイを設定してみる - EC2](https://qiita.com/leomaro7/items/52147ee88c6da11048e2)

### subnet の作成
#### public subnet
- 名前: example_ec2_2208_public
- AZ: ap-northeast-1a
- IPv4 CIDR ブロック: 192.168.0.0/25
#### private subnet
- 名前： example_ec2_2208_private
- AZ: ap-northeast-1a
- IPv4 CIDR ブロック: 192.168.0.128/25

## IGW の作成とアタッチ
### IGW の作成
VPC の IGW の項目から作成できる．
- 名前: example_ec2_2208
### IGW のアタッチ
個別の IGW の設定から VPC にアタッチできる．

## NAT GW の作成とアタッチ
### NAT GW の作成
VPC の NAT GW の項目から作成できる．
- 名前: example_ec2_2208
- サブネット: example_ec2_2208_public
- 接続タイプ: パブリック
- ElasticIP: [Elastic IP を割り当て] ボタンを押す

## メインルートテーブルの設定 (public)
### ルートテーブルの作成
VPC のルートテーブルの項目から作成できる．
- 名前: example_ec2_2208_public_rtb
- VPC: example_ec2_2208_public
### メインルートテーブル編集
- ルート追加
    - 送信先: 0.0.0.0/0
    - ターゲット: igw-xxxxxxxxxxxxxxxxxxxx
### カスタムルートテーブルの追加
- example_ec2_2208_public

## メインルートテーブルの設定 (private)
### ルートテーブルの作成
VPC のルートテーブルの項目から作成できる．
- 名前: example_ec2_2208_private_rtb
- VPC: example_ec2_2208_private
### メインルートテーブル編集
- ルート追加
    - 送信先: 0.0.0.0/0
    - ターゲット: nat-xxxxxxxxxxxxxxxxxxxx
### カスタムルートテーブルの追加
- example_ec2_2208_private を明示的に関連付ける

## セキュリティグループ
- 名前: example_ec2_2208_private_ec2
### インバウンドルール
- SSH: EC2 の作成時に同時に作るので，ここでは作らない

## EC2 の作成
### インスタンスの起動
- 名前: example_ec2_2208
- OS: Amazon Linux
- instance type: t3.micro
- storage type: gp3

- ネットワーク
    - VPC: example_ec2_2208
    - サブネット: example_ec2_2208_private
    - パブリックIPの自動割り当て: 無効
    - 既存のセキュリティグループを適用する: example_ec2_2208_private_ec2

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
    - Connect to an EC2 compute resource
        - EC2 Instance: example_ec2_2208
    - パブリックアクセス: なし
    - 新しい VPC セキュリティグループ
        - 新規作成
            - 新しい VPC セキュリティグループ名
                - example_2208_aurora_sg
    - RDS Proxy
        - なし
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
  mysql -u admin -p -h database-1-instance-1.c6dtejpehxyt.ap-northeast-1.rds.amazonaws.com
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

- [【初心者向け】RDS for MySQLを構築しEC2からアクセスしてみる](https://dev.classmethod.jp/articles/sales-rds-ec2-session/)

- [EC2からS3へアクセスする4つのルートとコスト](https://tech.nri-net.com/entry/access_routes_from_EC2_to_S3)
- [VPCエンドポイントを試してみた](https://qiita.com/Jerid/items/5dd072b1f88a7913f99c#s3%E3%81%AE%E4%BD%9C%E6%88%90)
