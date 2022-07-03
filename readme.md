# DemoCA

## 概要

デモ用ルートCAと下位CAの作成用の設定ファイルとコマンドをまとめたもの。

## 証明書作成方法

### ルートCAの作成
ディレクトリを作成して証明書と鍵を発行する。
```
cd root-ca
mkdir certs db private
chmod 700 private
touch db/index
openssl rand -hex 16 > db/serial
echo 1001 > db/crlnumber

openssl req -new \
-config root-ca.conf \
-out root-ca.csr \
-keyout private/root-ca.key

openssl ca -selfsign \
-config root-ca.conf \
-in root-ca.csr \
-out root-ca.crt \
-extensions ca_ext
```
### 下位CAの作成

ディレクトリを作成して鍵と証明書発行要求を作成する。

```
cd sub-ca
mkdir certs db private
chmod 700 private
touch db/index
openssl rand -hex 16 > db/serial
echo 1001 > db/crlnumber

openssl req -new \
-config sub-ca.config \
-out sub-ca.csr \
-keyout private/sub-ca.key

```

CSRをroot-caにコピーしてroot-ca署名する。
署名した証明書をsub-caにコピーする。

```
cp sub-ca.csr ../root-ca/
cd ../root-ca
openssl ca \
-config root-ca.config \
-in sub-ca.csr \
-out sub-ca.crt \
-extensions sub_ca_ext
cp sub-ca.srt ../sub-ca/
```

### サーバ証明書の発行方法
鍵と証明書発行要求を作成し、sub-caにコピーする。  
ついでに秘密鍵のパスフレーズを解除する。
```
cd www
openssl genrsa -aes128 -out server.key 2048
openssl rsa in server.key -out server.key 
openssl req -new -key server.key -out server.csr -addext 'subjectAltName = IP:192.168.0.1,DNS:example.jp'
cp server.csr ../sub-ca/
```

コピーした証明書発行要求から署名した証明書を作成し、wwwにコピーする。
```
cd ../sub-ca
openssl ca -config sub-ca.config -in server.csr -out server.crt -extensions server_ext
cp server.crt ../www
```
