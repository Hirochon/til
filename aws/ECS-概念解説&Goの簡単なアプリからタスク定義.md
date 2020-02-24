## ECS-概念解説とGoの簡単なアプリからタスク定義していきます！(2020/2/24)

### 自分なりのECSの機能についての解釈(※個人の見解です)
    - ECS(EC2 Container Service): そもそもECSとはDockerをEC2上で簡単に扱えるだけではなく、サービス志向な機能が色々と備わった管理ツールだと思ってる。
    - Task(タスク): インスタンス上で実行されてるコンテナさん。
    - Task Difinition(タスク定義): タスクで使われるコンテナの設定やボリュームとかを定義。もうね、使えば使うほど`docker-compose`にしか思えなくなってくる。
    - Cluster(クラスター): タスクが実行される場所。作成時にはコンテナインスタンスと共にEC2インスタンスも作成する。一緒にVPCとかも設定。
    - Service: クラスターでタスクがどれだけ実行するか決める。ロードバランサーはクラスターで作成されたインスタンスをTG先に指定しておいて、ここで紐付ける。
    - Manager: クラスターのリソースと、タスクの状態を管理。
    - Scheduler: クラスターの状態を見て、タスクを配置。

### 今日の積み上げ
1. 『ECSを扱えるように俺はなる』の続き([昨日の続き](https://github.com/Hirochon/til/blob/master/aws/ECS-概念を理解してこーぜ編.md))です。笑
2. ポートとか色々と見直して、『これどーかなー』『あれどーかなー』とか、クラスター作って、タスク定義してみたけど、一向にタスクが`RUNNING`になりません！
3. 手順をすっ飛ばしすぎたと気づき、まずは簡単なGoで作ったアプリケーションをECSで管理しようと思って、まずは以下のようにローカルで立ち上げました！
    1. まずは簡単にできると噂のGoのWebサーバーの立て方を調べる。コチラの記事にはお世話になりました→[Docker上でgolangの開発環境を整える](https://qiita.com/yasuno0327/items/be7fb992054f40b491cc)
    2. やや🔺でネタバレになってますが、もちろん環境構築はDockerで行い、Webサーバーをたてつつ、htmlを表示させる流れを想定しました！
    3. 🔺で紹介したQiita記事様が優秀で、`main.go`は↓のように丸コピで成功w。ただしDockerの設定に関しては、僕のベストプラクティスに沿っていただきました。笑

    ```golang:main.go
    package main

    import(
    "log"
    "net/http"
    "path/filepath"
    "sync"
    "text/template"
    )

    type templateHandler struct {
    once sync.Once
    filename string
    templ *template.Template
    }

    func (t *templateHandler) ServeHTTP(w http.ResponseWriter, r *http.Request) {
    t.once.Do(func() {
        t.templ = template.Must(template.ParseFiles(filepath.Join("templates", t.filename)))
    })
    t.templ.Execute(w, nil)
    }

    func main() {
    http.Handle("/", &templateHandler{filename: "index-go.html"})

    if err := http.ListenAndServe(":8080", nil); err != nil {
        log.Fatal("ListenAndServe", err)
    }
    }
    ```

    4. 見事Docker上でGoのアプリを立ち上げることができました!→[画像はコチラ](https://twitter.com/heacet43/status/1231830408582447105)
    5. ここでdocker-composeの出番は終わってしますので、勝手に作成した`docker-compose.yml`を載せておきます。笑

    ```yaml:docker-compose.yml
    version: '3'
    services:
        go_app:
            build: .
            ports:
                - "8080:8080"
            command: "go run main.go"
    ```

4. 次にECRにpush→ECSのタスク定義→ECSのクラスター作成→ECSのクラスターのGoアプリのサービス作成！→実行中のタスクが現れてくれる！という計画を立てて以下のようになる。
    1. 🔺の時点で作成したImageをAWSのタグの付け方に従いつつ、そのままECRにpushする。
    2. やっぱりここが難しいEC2のタスク定義。機能が多いんだよねー。だけど昨日の僕とは違う。何を指定すべきかわかるし(※成功したことはない)、何せ扱うコンテナが1つだけ。
    3. そしてこの時には気づかない挫折しちゃうポイントに知らず知らず引っかかる。(※コンテナ追加の環境にあるコマンド欄に`go run main.go`を書いていない(笑)。そりゃ動かんで笑)
    4. タスクが定義できたので、クラスターを作成する。
    5. サービスをさくせい！
    6. タスクを確認　→　タスクが実行されない！？！？！？
    7. わからなくて、気絶。
5. 寝転びながら考えてると、そーいえば`docker-compose.yml`では、サーバーを起動させるための`command`を設定していたことを思い出す！
6. 勢い良く復活して、コンテナのcommand欄に`go run main.go`を追加！
7. ECSを用いたサービスのデプロイ成功に喜んびまくったツイートは[コチラ](https://twitter.com/heacet43/status/1231881020577738752?s=20)
8. ついでにEC2インスタンスにSSHログインして、`docker ps`したり、`docker images`,`docker exec`...などなど、色々試しましたが、しっかりとDocker動いてましたw