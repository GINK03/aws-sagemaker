# aws-sagemaker
AWS Sagemakerの使用例とできること、工夫すべき点を示します


## 目次
- AWS sagemakerとは
  - Amazon Linux
- anacondaにモジュールを追加する　
- SageMakerのOSとusernameとpermission

- インスタンスのスケール調整

- ディスクの制限をEFSで回避する
  - セキュリティグループの扱い

- ディスクの制限をS3で回避する

- Random Cut Forestの異常値検知アルゴリズム
  - アルゴリズム概要
  - 論文
  - ユースケースを実行する

## AWS sagemakerとは
EC2インスタンス等を立てることなく、JupyterNotebookやTensorFlowやSpark等を扱うことができます。  

<div align="center">
  <img width="550px" src="https://user-images.githubusercontent.com/4949982/39577840-61befd4a-4f1d-11e8-8afd-586eea29bdae.png">
</div>
<div align="center"> 図1. SageMakerのサービスに入る </div>

<div align="center">
  <img width="550px" src="https://user-images.githubusercontent.com/4949982/39578205-84670850-4f1e-11e8-9d51-a8ef71124437.png">
</div>
<div align="center"> 図2. JupyterNotebookのインスタンス作成＆起動 </div>

<div align="center">
  <img width="550px" src="https://user-images.githubusercontent.com/4949982/39578315-cc096ad6-4f1e-11e8-9574-2d5ec8bc7e82.png">
</div>
<div align="center"> 図3. フルサイズのJupyterが立ち上がります </div>

<div align="center">
  <img width="350px" src="https://user-images.githubusercontent.com/4949982/39578380-f89462e0-4f1e-11e8-8e8f-aa163ec682c3.png">
</div>
<div align="center"> 図4. 対応しているカーネルはSpark各種と、mxnet, tensorflow, anaconda </div>

## anacondaにモジュールを追加する  
Jupyterはユーザ権限で動作しているので、Jupyterの中から直接、モジュールをインストールできます  
```jupyter
%%sh 
conda install -c conda-forge lightgbm
```
```jupyter
import lightgbm as lgb # OK!
```

## SageMakerのOSとusernameとpermission
JupyterNotebookなので、terminalが使えます。  

Linuxの種類とversionは2018/5時点でこのようになっています。
```console
sh-4.2$ less /etc/system-release
Amazon Linux AMI release 2017.09
```

usernameは以下の通りでsudoする権限が与えられています  
```console
sh-4.2$ whoami
ec2-user
```

## インスタンスのスケール調整

<div align="center">
  <img width="350px" src="https://user-images.githubusercontent.com/4949982/39581668-11c47c7a-4f27-11e8-98d0-b88676e84169.png">
</div>
<div align="center"> 図5. インスタンスを停止して、編集をクリックします　</div>

<div align="center">
  <img width="350px" src="https://user-images.githubusercontent.com/4949982/39581463-97b203b2-4f26-11e8-9bd1-a8f27ed62cdf.png">
</div>
<div align="center"> 図6. インスタンスのサイズを変更できます　</div>

## ディスクの制限をEFSで回避する

AWS SageMakerは5GByteの容量制限があるために、大規模なデータを扱う際には外部ディスクとの接続が必要になります。  

[オフィシャルサイトでもその手法は紹介されて](https://aws.amazon.com/jp/blogs/news/mount-an-efs-file-system-to-an-amazon-sagemaker-notebook-with-lifecycle-configurations/)いますが、EFSというネットワーク経由のディスクをマウントする際に気をつけるべき点があります。　

- EFSとSageMakerのセキュリティグループは同じである  
　
これを気づかずに数時間とかしました。。。

<div align="center">
  <img width="450px" src="https://user-images.githubusercontent.com/4949982/39610067-c170d36a-4f88-11e8-8e99-8b705172732b.png">
</div>
<div align="center"> 図7. EFSのセキュリティグループをSageMakerのインタンスのものと同じにしておきます </div>

SageMakerも一旦インスタンスを作成してしまうと、セキュリティグループ等を変更できないので、要注意です。

Jupyterでshを有効にするか、terminalに入って、以下のコマンドでEFSをマウントできます。  
```console
$ cd SageMaker
$ mkdir efs
$ sudo mount -t nfs4 -o nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2 ${MOUNT_TARGET_IP}:/ efs
$ sudo chown -R uc2-user efs
```
複数のインスタンスでアクセスでき、後述するS3をマウントするアプローチよりだいぶ速度的にマシだったりして、選択としてありだと思います。

## ディスクの制限をS3で回避する
S3でマウントするにはgoofysというgoでマウントする仕組みが便利です。  

Linux用のバイナリがすでにコンパイルされて公開されているので、それを利用します。　　

goofysのバイナリを設置して、credencialsを設定します
```console
$ cd SageMaker
$ mkdir bin
$ cd bin 
$ wget http://bit.ly/goofys-latest
$ chmod +x goofys-latest
$ cat ~/.aws/credencials
[default]
aws_access_key_id = ${AKID1234567890}
aws_secret_access_key = ${MY-SECRET-KEY}
```
必要なバイナリ、fusermountが含まれていないので、別途インストールします
```console
$ sudo yum install fuse
```
マウントします
```console
$ /bin/goofys-latest gk-sagemaker-test-01 s3
```
これでs3で作業するには容量制限を受けませんが、遅いので、コスパ重視の設定になります

