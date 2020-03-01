## upstream-socket通信とかいうムズい奴 (2020/3/1)

### 今日の流れ
1. 今週は就活で立て込んでたので、たまってたやるべきこと(私生活系)を色々とやった。
2. 本日はNginxと全面戦争です。`upstream`をどないしていいか。
3. 始まりはECSで`back:8000`という書き方が通用しなかったこと。
4. よくある` unix:/run/gunicorn/ecs-test.sock`的なのを`upstream`に書けるようにsocket通信に挑戦してみました。
    1. socketファイルやserviceファイルを記述した。
    2. systemdが必須みたいだから、centos7のイメージにnginxをyumでインストールする`Dockerfile`を作成した。
    3. systemdが使えない！？！？！？！？！？
    4. どうやら`CentOS in DockerHub`曰く、systemd使えなくしてるみたいで、専用の何かが必要みたい。笑
    5. 結局以下のすごい`Dockerfile`が完成した。

    ```dockerfile:Dockerfile
    FROM centos:7
    ENV container docker

    COPY ./nginx.repo /etc/yum.repos.d/nginx.repo
    RUN yum update -y &&\
        yum install -y nginx

    COPY ./conf/ /etc/nginx/
    COPY ./html/ /etc/nginx/html/
    COPY ./gunicorn/ /etc/systemd/system/

    RUN (cd /lib/systemd/system/sysinit.target.wants/; for i in *; do [ $i == \
    systemd-tmpfiles-setup.service ] || rm -f $i; done); \
    rm -f /lib/systemd/system/multi-user.target.wants/*;\
    rm -f /etc/systemd/system/*.wants/*;\
    rm -f /lib/systemd/system/local-fs.target.wants/*; \
    rm -f /lib/systemd/system/sockets.target.wants/*udev*; \
    rm -f /lib/systemd/system/sockets.target.wants/*initctl*; \
    rm -f /lib/systemd/system/basic.target.wants/*;\
    rm -f /lib/systemd/system/anaconda.target.wants/*;
    VOLUME [ "/sys/fs/cgroup" ]
    CMD ["/usr/sbin/init"]

    EXPOSE 80
    ```

    6. 結果としてDocker上且つシンプルにソケット通信について詳しくなくて、見様見真似でやってみたが失敗。
5. うーん。`docker-compose.yml`のイメージだとうまく行ってくれたりしないかなぁ...無理でした！
6. どーーーーーーーーーーすればつながるんやああああああああああ！！！！！！！！