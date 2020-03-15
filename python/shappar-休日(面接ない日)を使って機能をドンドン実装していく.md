## 休日(面接ない日)を使って機能をドンドン実装していく (2020/3/15)

### リポジトリ [Shappar](https://github.com/Hirochon/Shappar)

### 今日やったこと
1. 研究室配属希望調査票
    - 今まで集めてきた情報を元に優先順位付！
    - 記入！
    - PDFでそうしん〜。
2. ShapparのSES送信制限解除！
    - 申請しました→ [コチラを参考](https://docs.aws.amazon.com/ja_jp/ses/latest/DeveloperGuide/request-production-access.html)
    - 無事申請とおりました〜[Issue](https://github.com/Hirochon/Shappar/issues/103)
3. 投票詳細機能の実装！
    - 昨日の知見を活かして、Userモデルを取得！(※PostIDと対応する投票済みのユーザーを取得してます)
    - `users = get_user_model().objects.filter(poll_user__post__id=pk)`
    - 取得するフィールドが特殊だったので、新たにserializerを定義。
    - あとの整形はfor文やif文使いまくって、もうぶち込んでます！
    - [pull request](https://github.com/Hirochon/Shappar/pull/107)
4. 投稿削除機能を実装！
    - DELETEメソッドはRESTAPIでは初めてでしたが、あっさり実装できましたw
    - [pull request](https://github.com/Hirochon/Shappar/pull/107)
5. アカウント登録時のメールの内容を変更いたしました
    - allauthやはり元のコードが分かりにくい&記事少ない
    - 公式リファレンスで設定方法見つつ、試行錯誤〜
    - [pull request](https://github.com/Hirochon/Shappar/pull/107)
6. ECSのタスク定義での環境変数を使ってデプロイ成功なるか！？
    1. まずは大切なKey達の保管から始めた
    2. パラメーターストアを使った環境変数を設定しました！　コノキジバリサンコウナッタ→[ECSでごっつ簡単に機密情報を環境変数に展開できるようになりました！ ｜ Developers.IO](https://dev.classmethod.jp/cloud/aws/ecs-secrets/)
    3. 『環境変数を`.env`ファイルとして準備しなくても良いのかな〜』と思ってたけど、何故か上手く行った！？
    4. パラメータストアに保管している環境変数達のシングルコーテーションをめっちゃ取ってやった(何かしらのバグが出てた)
7. ついでにとっておきの`Linuxコマンド`が知れた
    - envコマンドで環境変数を出力しつつ、その出力内容をファイルに入れる！
    - `env > .env`　コチラの記事参考→ [Linux：出力結果やテキストをファイルに書き出す方法 | raining](http://raining.bear-life.com/linux/出力結果やテキストをファイルに書き出す方法)
8. GitHubで自動Projects作成&移動機能を身につけるw
    - Issueを上げたタイミングでToDoリストに出してくれる機能があった！
    - IssueをクローズしたタイミングでDoneへ移動してくれるやつがいた！
    - PullRequestも然り
