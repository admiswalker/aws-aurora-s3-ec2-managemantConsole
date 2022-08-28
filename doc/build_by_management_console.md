# マネジメントコンソールを仕様した作成手順

Management Console から，EC2 と S3 と Aurora を立ち上げる．

## EC2 の作成

## NAT gateway の作成
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
- SSH

今は作らない

→ ここで踏み台を用意しないと，接続できない．
→ はじめから EC2 は踏み台用として用意することにする．

### インスタンスの起動
- 名前: example_ec2_2208_private
- OS: Amazon Linux
- instance type: t3.micro
- storage type: gp3

- ネットワーク
    - VPC: example_ec2_2208
    - サブネット: example_ec2_2208_private
    - パブリックIPの自動割り当て: 無効
    - 既存のセキュリティグループを適用する: example_ec2_2208_private_ec2

- 注意
    - 適当に作ると public subnet に作成される
    - パブリックサブネット
        - https://tech-dive.xyz/2019/07/06/%E3%83%91%E3%83%96%E3%83%AA%E3%83%83%E3%82%AF%E3%82%B5%E3%83%96%E3%83%8D%E3%83%83%E3%83%88%E3%81%A8%E3%83%97%E3%83%A9%E3%82%A4%E3%83%99%E3%83%BC%E3%83%88%E3%82%B5%E3%83%96%E3%83%8D%E3%83%83%E3%83%88/

## S3 の作成と EC2 からの接続

## Aurora の作成と EC2 からの接続

