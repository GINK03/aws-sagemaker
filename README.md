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

