---
title: SAM使ってRESTFulAPIを作る
tags: ["AWS", "Serverless", "SAM"]
date: 2019-06-20
---


## SAMについて

AWSがserverlessアプリケーション用に作ってる構成管理用のフレームワーク

[awslabs/serverless-application-model](https://github.com/awslabs/serverless-application-model)

CloudFormation通してリソースのデプロイをやってくれる

※単にLambda＋APIGatewayでRESTfullAPI作りたいならManagementConsoleでボタンポチポチでリソース作ったほうが絶対早い。

ただAWSアカウントまたいで移行するとかになると↑は死ぬ。なぜなら手順書なぞ作ってないから。これってなんのサービスで設定したんだっけって絶対なる。てゆーかなって死にそうになった。

結論はIaCって偉大。
ただCFnのtemplate作ってるときもそうだったけどGUIで作って、AWSのCfnのリファレンスとにらめっこしながら作ったリソースの設定値を見比べながらやってた。

いつかCFnのほうが早いっすよとか言いたい。

まだSAMのtemplateはLambda関連のサービスはだいたいのテンプレで作れるようになってるからマシ。[example](https://github.com/awslabs/serverless-application-model/tree/master/examples/2016-10-31)ばっか見てた


## 前提

dockerはLocal実行しないならいらんかった気がする。記憶曖昧
- python(2.7 || 3.6~)
- aws-cli
- docker

```
boilerplate ) python --version
Python 3.7.1
boilerplate ) aws --version
aws-cli/1.16.167 Python/3.7.1 Darwin/18.6.0 botocore/1.12.157
boilerplate ) docker -v
Docker version 18.09.2, build 6247962
```

### aws-sam-cliのインストール
pip使う。ちな[公式ページ](https://docs.aws.amazon.com/ja_jp/serverless-application-model/latest/developerguide/serverless-sam-cli-install.html)

```
pip install --user aws-sam-cli
```

PATHにバイナリの場所追加しとく。なお私はbashをデフォルトshellにしてfish起動してるので、デフォshell変えてる人は各種dotファイルに追加
```
boilerplate ) cat ~/.bash_profile 
if [ -f ~/.bashrc ] ; then
. ~/.bashrc
fi

export PATH=$HOME/.nodebrew/current/bin:$HOME/.pyenv/shims:$HOME/.local/bin:$PATH
exec fish

# profileの再読込
boilerplate ) bash
bash-3.2$ source ~/.bash_profile 
Welcome to fish, the friendly interactive shell
boilerplate ) sam --version
SAM CLI, version 0.16.1
```

### aws credential
setupしとく。この時単にlocalstackとかでlocalでlambdaのデバックしたいとかならdummyでいい。deployまで行きたいならデプロイの先のAWSのcredentialを入れる

なお、アクセスキー払い出しのユーザーRoleはいつもめんどくさくてAdminRoleを使ってしまう。なにが必要か調べるのめんどくさいんだよね。。。

多分CloudFormationとS3のPolicy当てればいいんだけど、CloudFormationのFullAdminって実質AdminRoleやろ（適当）
```
boilerplate ) aws configure --profile sam
AWS Access Key ID [None]: hoge  
AWS Secret Access Key [None]: fuga
Default region name [None]: ap-northeast-1
Default output format [None]: json
```

## プロジェクト作成

`sam-init`でテンプレ作ってくれる。正直必要なのは`template.yaml`とlambdaFunctionくらいなので自分で作ったほうが早いとおもいます。

Runtimeはnodejs10.xを使う。Pythonとかいい人はPythonつかってね。
```
git ) sam init -r nodejs10.x -n sample-sam-api
[+] Initializing project structure...

Project generated: ./sample-sam-api

Steps you can take next within the project folder
===================================================
[*] Invoke Function: sam local invoke HelloWorldFunction --event event.json
[*] Start API Gateway locally: sam local start-api

Read sample-sam-api/README.md for further instructions

[*] Project initialization is now complete
git ) cd sample-sam-api/
git ) sam init -r nodejs10.x -n sample-sam-api
[+] Initializing project structure...

Project generated: ./sample-sam-api

Steps you can take next within the project folder
===================================================
[*] Invoke Function: sam local invoke HelloWorldFunction --event event.json
[*] Start API Gateway locally: sam local start-api

Read sample-sam-api/README.md for further instructions

[*] Project initialization is now complete
git ) cd sample-sam-api/
sample-sam-api ) tree
.
├── README.md
├── event.json
├── hello-world
│   ├── app.js
│   ├── package.json
│   └── tests
│       └── unit
│           └── test-handler.js
└── template.yaml

3 directories, 6 files
```

### とりあえず動かす
`sam local start-api　--profile sam`でとりあえず動かす。
localhost:3000/helloにGETすれば、必要なDockerImageをpullしてコンテナ動かしてくれる。

実行が終わるとコンテナは削除される。実際のLambdaと同じ動作

### 実装

これでHelloWorld終わりなのでhello-worldは削除しちゃってsrcとか作っちゃってinitする

とりあえずざっくり作る




## 完走した感想(激ウマ)
正直ベンダーロックイン嫌うなら[severless Framework](https://serverless.com)使ったほうがいい。serverlessでやろうとしてベンダーロックイン嫌うって意味分かんないけど。

ちなみにserveless FrameworkもPlugin入れればLocal実行できる。
こっちならAzureとかGCPでも行けるし。結局yaml書き直すことになるがな
