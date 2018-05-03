# aws-sagemaker
AWS Sagemakerの使用例とできること、工夫すべき点を示します


## 目次
- AWS sagemakerとは
  - Amazon Linux
- Jupyterを使う
- インスタンスのスケール調整
- ディスクの制限をS3で回避する
- ディスクの制限をEFSで回避する
  - セキュリティグループの扱い
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
