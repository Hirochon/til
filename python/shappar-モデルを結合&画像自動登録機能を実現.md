## モデルを結合&画像自動登録機能を実現(2020/2/14)

### リポジトリ: [Shappar](https://github.com/Hirochon/Shappar)

### 今日は何したか
0. 東京に夜行バスで来たため、疲労混じりで作業をすすめる。笑
1. DjangoのAWS系の設定をいじっていると、CloudFront無しバージョンの設定があって、位置が微妙だったので変更。
2. やはり大きく初期userモデルを変更すると、マイグレーションがおかしくなるので、migrationsを全消去。
3. マイページ問題と画像問題を解決するために思考する。
    - 問題点
        1. アカウント作成時(SignUp時)にマイページが作成されないので、シンプルにマイページへ遷移することができない。
        2. 画像のディレクトリをUUIDで扱っているため、アカウント作成時ではUUIDが生成されていないので、画像を保存することができない。
    - 解決策
        1. マイページとアカウント作成時のモデル(customuserモデル)を合体させたことにより、アカウント作成時にマイページも生成されるようになった。
            ```python:
        2. アカウント作成時はユーザに画像を選択させないことによって、デフォルト値として全てのユーザにdefault画像のディレクトリを渡すことによって解決。
            ```python:models.py
            iconimage = models.ImageField(verbose_name='アイコン画像', blank=True, null=True, default='images/customuser/iconimage/icon.png', upload_to=get_iconimage_path)
            homeimage = models.ImageField(verbose_name='ホーム画像', blank=True, null=True, default='images/customuser/homeimage/home.jpg', upload_to=get_homeimage_path)
            ```

#### 今日のハイライト
**東京マクドでプログラムをカタカタ打っていたら、横の人も急にパソコンを取り出してカタカタしだしたので、カタカタマウントを取り合っていた。**