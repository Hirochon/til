## shappar-Optionのシリアライザテストを追加 (2020/3/22)

## 今日やったこと
1. 某社のコーディングテスト
    - 出来は悪かったwww
2. 某社のES!!
    - やりきっっっっった！
3. 他某社のES!!!
    - 前社のESへの気持ち切り替えられず断念
4. 友達と議論しつつShappar作成！！
    - デザインはしっかりと議論します！！
    - 僕は友達がフロントを変更している間にシリアライザのテストに取り掛かる！
5. 空き時間にOptionSerializerのテストを書いてました！
    - 以下の入出力データのバリデーションを作成
    - (正常系)2methods,(異常系)3methods
    - [実際のコードはコチラ](https://github.com/Hirochon/Shappar/blob/back/apiv1/tests/test_serializers.py)
6. Makefileに単体テストコマンドを追加
    - `docker-compose run --rm shappar-back python3 manage.py test apiv1.tests.test_views --settings=config.settings.local`
    - `docker-compose run --rm shappar-back python3 manage.py test apiv1.tests.test_serializers --settings=config.settings.local`
7. 他の時間はひたすら、デザインデザインバグ探しデザインバグ探しデザインデザイン手感じでした！