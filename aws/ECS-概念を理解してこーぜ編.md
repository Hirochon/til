## Amznon EC2 Container Service(ECS)の概念を理解してこーぜ！編(2020/2/23)

### 今日の積み上げ
1. [昨日](https://github.com/Hirochon/til/blob/master/thatday/様々なジャンルのタスクだったからコチラで.md)に引き続いて、コーディングテスト(期限は1週間)の仕上げにかかっていた。終わらせたところコード量はコメントアウトや改行含めだけど600行に達していた。笑
2. 🔺の感想としては課題の要求が多かった分プログラムも複雑だったので長くなった。そこからもう一手間、いくつかのプログラムのユーザ関数化を行ったらコード量は4分1くらいにはなるかなーと思った。やっぱりC言語で書いていると、標準装備の関数が少ないので文量は多くなってくるけど、高級言語の中でも機械語よりなので、自分的に何でも実装できると思っていて(気持ちの面でも)、本当に扱いやすい。
3. プログラムの解説欄があるんだけど、Qiitaで使いまくって(READMEでもよく使うw)得意になりつつあるマークダウンで装飾しつつ解説しました。笑
4. 次にShapparのデプロイ(ローンチ?)に向けて、ECSの勉強を本格的に始める。
5. 一回Qiita見つつ自分でアレコレいじってECSを立ち上げる！！　→　『思うように動かない！』ちなみに以下の記事を参考にしました〜。**※自分でいじっちゃっているので以下記事は悪くありません。**
    - [AWS ECSでDockerコンテナ管理入門（基本的な使い方、Blue/Green Deployment、AutoScalingなどいろいろ試してみた） - Qiita](https://qiita.com/uzresk/items/6acc90e80b0a79b961ce)
    - [初心者でもできる！ ECS × ECR × CircleCIでRailsアプリケーションをコンテナデプロイ - Qiita](https://qiita.com/saongtx7/items/f36909587014d746db73)
6. 今まで独学で走り続けてるので、もう恒例になりつつある挫折(てか見本通りではなく、イジってる時点で『動かないんやろなー』て悟ってた。もはや挫折じゃないよね笑)。
7. 概念から理解することに方向転換　→　1のスライドを朗読する。(2のQiitaも参考にしつつ)
    1. [AWS Black Belt Online Seminar 2016 Amazon EC2 Container Service](https://www.slideshare.net/AmazonWebServicesJapan/aws-black-belt-online-seminar-2016-amazon-ec2-container-service)
    2. [Amazon EC2 Container Service(ECS)の概念整理 - Qiita](https://qiita.com/NewGyu/items/9597ed2eda763bd504d7)
8. 概念は理解できた気はするけど、中々実際にEC2を使ってのデプロイに手こずる...
9. CloudWatch見る限りでは、『envファイルの置きどころが悪い？』『dockerのECRにpushする用のビルドしたイメージがキャッシュで作られててエラー出た？』
10. PostgreSQLはしっかりうごいてたんだよなー...笑
11. Fargate使ったECSはチョー簡単だった。笑