## 一気に投稿一覧表示と投票機能を実装したよ!! (2020/2/19)

### リポジトリ: [Shappar](https://github.com/Hirochon/Shappar)

### 徹夜で積み上げ笑(なんか良い意味でとまらなかった...！笑)
1. まずやったことは、**『あれ？share_idいらなくね？』** と思って、share_idをいらない子だと思い、即消しました。笑
2. モデルで定義していないものをどうやってフィールドに乗っけて、シリアライズしようかな...
3. そう考えながらDRFの教科書(akiyakoさん作)をチラ見してると、なんと現場で使えるTips集に載っているではないですか！！！
4. 『フムフム』しながら、実装完了する。
5. 『自分が投稿する』or『自分が投稿した以外の投稿に投票する』という2種類しか、投票結果を見る手立てはないという設定→if文とリレーションシップをうまく使いながら実装
6. 🔺でvotedが誕生。viewsでfor文によってvoted=Falseの値には、-1を返して、Trueのときはちゃんと返す設計にする。だが合計点は一般公開。
7. UUIDをユーザーしか持てない値と仮定して、Publicページにてvoted_False or Trueが実現するか試す→成功。
8. そしてやってくる投票。そして気づくのだ。**『これshare_idいるくね？』**→急いで書き直す。→前に書いたので一瞬だった。
9. 元々色々フィールドを置く予定していたPollに関しては、Optionモデルの誕生によりプログラム量と激減した。
10. 投票時にはシリアライザどうしようとめちゃくちゃ迷った。
    - 我流でぶち込んでみても、繊細にぶち込んでみても無理だった。
    - ネスト形式で出力してるからムズいてか眠い！！！！
11. viewa.pyに無理やりOptionモデルを作成することで、シリアライザ外でOption専用のシリアライザをかけれた。シリアライザ2刀流！！


### 感想：めちゃめちゃ実装させた気がする。
- 眠いです。なんで徹夜したのかまだわかりません。
- こんな眠い中TILを書いている自分を布団の中で褒めてあげたいと思います。
- おやすみいいいいいいいいいいいいいいい！