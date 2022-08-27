# マネジメントコンソールを仕様した作成手順

Management Console から，EC2 と S3 と Aurora を立ち上げる．

## EC2 の作成

## NAT gateway の作成

### VPC の作成
- 名前：example_ec2_2208
- 手動：192.168.0.0/24


- [AWS NAT構成の作り方(NATゲートウェイ編)](https://qiita.com/TK1989/items/5d9bd5d49708c02dff3f)
- [NAT ゲートウェイを設定してみる - EC2](https://qiita.com/leomaro7/items/52147ee88c6da11048e2)

### サブネットの作成
- 名前： example_ec2_2208_public
- AZ: 指定なし
- IPv4 CIDR ブロック：192.168.0.0/25





### インスタンスの起動
- 名前：example_ec2_2208
- OS: Amazon Linux
- instance type: t3.micro
- storage type: gp3

- 注意
    - 適当に作ると public subnet に作成される
    - パブリックサブネット
        - https://tech-dive.xyz/2019/07/06/%E3%83%91%E3%83%96%E3%83%AA%E3%83%83%E3%82%AF%E3%82%B5%E3%83%96%E3%83%8D%E3%83%83%E3%83%88%E3%81%A8%E3%83%97%E3%83%A9%E3%82%A4%E3%83%99%E3%83%BC%E3%83%88%E3%82%B5%E3%83%96%E3%83%8D%E3%83%83%E3%83%88/

## S3 の作成と EC2 からの接続

## Aurora の作成と EC2 からの接続

