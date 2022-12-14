# Dockerのメモ

目次
 1. <a href="#1">目的</a>
 2. <a href="#2">Dockerの概念</a>  
    <a href="#2.1">2.1 サーバ仮想化</a>  
    <a href="#2.2">2.2 Dockerとは</a>  
 3. <a href="#3">実行環境とDockerの環境構築</a>
 4. <a href="#4">Dockerの使い方</a>    
    <a href="#4.1">4.1 Hello World!</a>  
    <a href="#4.2">4.2 イメージとコンテナ</a>    
    <a href="#4.3">4.3 Dockerfile</a>    
    <a href="#4.4">4.4 docker-compose</a>  

参考文献

<div id="1">

## 1. 目的
研究やシステム開発, 趣味などで「とりあえず動作する環境が欲しい」「軽くソフトウェアやプログラミング言語に触りたい」という需要が増えてきている. また筆者に次のような要望・依頼を数件いただいている.
- Dockerの使い方を教えてほしい
- 仮想化技術を用いて開発環境を整えたい
- 複数コンテナを組み合わせて開発を行う方法が知りたい

この要望に対して仮想化技術, 特にDockerを用いて環境を整えることで難しい環境構築にコストをかけずに開発を始めることができる.
そこで仮想化技術の概念について理解し, Dockerを用いて環境構築を行うことを目的として本リポジトリを作成する. 読者のレベルとしては基本的なUnixコマンドが使えること, ネットワークに関する基礎的な知識を有していることを想定している.  
2章では仮想化技術としてサーバ仮想化について説明した後, Dockerとは何か簡単に説明する. 3章では本ドキュメントの実行環境と, Docker自体の環境構築に参考となるサイトの紹介を行う. 4章では実際にDockerを用いて開発環境を整える方法を紹介する. 環境構築ではチュートリアルイメージと公式イメージを動作させた後にDockerfileを用いて環境構築を行う方法と, docker-composeを用いて複数のイメージを組み合わせた環境を構築する方法について説明する.

<div id="2">

## 2. Dockerの概念
本章ではDockerの概念としてサーバ仮想化とよびDockerの概要について説明する.

<div id="2.1">

### 2.1 サーバ仮想化
仮想化技術とは物理構成(ハードウェア)の機能を論理構成(ソフトウェア)によって実現する技術である[1]. 仮想化技術の一つにサーバ仮想化と呼ばれる, 1台の物理サーバ上で複数の仮想的なサーバを動作指せる技術がある. サーバ仮想化にはホスト型仮想化, ハイパバイザ型仮想化, コンテナ型仮想化の3種類がある.  
ホスト型仮想化は次に示すような構成でサーバ仮想化を行う方法である. ホスト型仮想化ではハードウェア上で動作するホストOSに, 仮想化ソフトウェアをインストールする. そして仮想化ソフトウェア上でゲストOSを動作させ, ゲストOSでアプリケーションが動作する. 

![img](./img/vm_host.png)

ホスト型仮想化でサーバ仮想化を行った環境の例を次に示す. この例ではハードウェアに搭載されているWindows10上に仮想化ソフトウェアとしてVirtualBox[1]をインストールしている. VirtulBoxではCentOSとUbuntuの2つのゲストOSが動作している. CentOSではRubyとSQLite, UbuntuではPythonとNginxのアプリケーションが動作している.

![img](./img/vm_host_ex.png)

ホスト型仮想化のメリットは既存のマシンを利用してサーバ仮想化が行えること, 仮想化に必要なソフトウェアが扱いやすいことがあげられる. 一方デメリットとしては後述するハイパーバイザー型の仮想化ソフトウェアに比べて仮想サーバの動作が遅いこと, ホストOSを動作させるために物理リソースが必要であることがあげられる. リソース面, 動作速度面から本番環境に適しているとは言えないがテスト用の環境であれば問題なく使うことができる.  
次にハイパーバイザー型仮想化について説明する. ハイパーバイザー型仮想化は次に示すような構成でサーバ仮想化を行う方法である. 仮想化のためのOSのような役割としてハイパーバイザーがあり, ハイパーバイザー上でゲストOSが動作する. 

![img](./img/hypervisor.png)

ハイパーバイザー型でサーバ仮想化を行った環境の例を次に示す. この例はWindowsでWindows Subsystem for Linux(WSL2)を利用するために, Hyper-Vと呼ばれるハイパーバイザー上にWindows10とUbuntuの2つのゲストOSを配置している[2][3].

![img](./img/hypervisor_ex.png)

ハイパーバイザー型仮想化のメリットはホストOSが不要でハードウェアを直接制御できること, システム全体としてリソースの使用効率がいいこと, 管理するサーバの台数削減が可能であることがあげられる. 一方でデメリットとして仮想化環境の高度な管理を実現するツールが標準装備されていないことがある, ハードウェアのスペックが低い場合は処理能力不足になることがある, 頻繁にアップデートが必要なものがあるため運用保守の観点にコストがかかることがあげられる.  
最後にコンテナ型仮想化について説明する. コンテナ型仮想化は次に示すような構成でサーバ仮想化を行う方法である. ハードウェアに搭載されたOS上にコンテナ管理ソフトウェア[4]をインストールし, コンテナと呼ばれる環境でアプリが動作する. ここまでの説明ではホスト型仮想化とほぼ同じであるように感じるが, 重要なのはコンテナの役割である. コンテナはアプリケーション実行環境を仮想化するためのもので, プロセッサやメモリの消費やストレージの負荷が軽量である. このため複数のコンテナを同時に動かすことができ, 起動や停止も高速で行える. 一方でデメリットとして複数のホストでのコンテナ運用が煩雑になること, カーネルを個別に変更できないこと, 新しい技術であるために学習コストが高いことが上げられる.

![img](./img/container.png)

コンテナ型仮想化の例を次に示す. この例ではWindows10上にDockerと呼ばれるコンテナ管理ソフトウェアをインストールしている. そしてコンテナ管理ソフトウェアで3つのコンテナが動作する. コンテナ内ではPython2.7, MySQL, Python3.8の3つのアプリケーションが実行されている.

![img](./img/container_ex.png)

<div id="2.2">

### 2.2 Dockerとは
前節で唐突に登場したが, Docker[5]はコンテナ管理ソフトウェアである. Dockerを用いることで1章の目的で筆者が受けた需要・要望に答えることができる.　「とりあえず動作する環境が欲しい」「軽くソフトウェアやプログラミング言語に触りたい」という需要は, Dockerを用いると配布されたベースイメージを使って簡単に軽量な環境を構築し, 必要が無くなればその環境を簡単に破棄するで満足することができる.  「仮想化技術を用いて開発環境を整えたい」「難しい環境構築にコストをかけたくない」という要望は, メジャーな言語やソフトウェアのベースイメージが配布されているので仮想化技術で開発環境を簡単に誰でも整えられる, Dockerfileに開発環境の設定を記述して配布すればどのOSでも同じ環境を構築することで満たすことができる. すなわち最初の開発者がDockerfileと開発に必要な一連のファイルを揃えておけば開発の引継ぎや, 初期段階での環境構築が簡単に行うことができる. 「複数コンテナを組み合わせて開発を行う方法が知りたい」という要望はdocker-compose.ymlに複数コンテナを組み合わせる設定を記述して, 実行することでコンテナを組み合わせることができるため満たすことができる. 複数コンテナを組み合わせるのはアプリケーション開発でフロントエンド, バックエンド, データベースをそれぞれ別のコンテナで構築するような場合に有用である.

<div id="3">

## 3. 実行環境とDockerの環境構築
Dockerの使い方で使用する環境と, Dockerの環境構築に参考になる文献を紹介する. ここではWindows11上にインストールしたDocker Desktop for Windows[6]を用いる. コマンド操作はWSL2から行うものとする. 現在ではWSL2上にDockerの環境を構築することができるので参考文献[7]の方法でDockerをインストールすることができる.

<div id="4">

## 4. Dockerの使い方
本章ではDockerの使い方として, Hello World! イメージとコンテナ, Dockerfile, docker-composeについて説明する. 

<div id="4.1">

### 4.1 Hello World!
DockerでHello Worldイメージを取得してコンテナを実行してみる. イメージやコンテナについては次節で説明する. まずDockerを起動して次にコマンドを実行して, Dockerのバージョンを確認する. バージョンが表示されれば, Dockerが起動できている. 
```
$ docker --version
Docker version 20.10.12, build e91ed57
```

Dockerを起動できていることを確認したら, 次にコマンドを実行してhello-worldする. 「Hello from Docker!」と表示されれば成功である.
```
$ docker run hello-world
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
2db29710123e: Pull complete 
Digest: sha256:18a657d0cc1c7d0678a3fbea8b7eb4918bba25968d3e1b0adebfa71caddbc346
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/

```

<div id="4.2">

### 4.2 イメージとコンテナ
イメージとコンテナの概念について説明する. イメージは開発環境を作るベースの雛形のようなもので, このイメージを使って実際に開発を行うコンテナを作成する. コンテナはイメージから実際に生成された開発環境であり, 1つのイメージから複数の環境を立てることができる. またここでは紹介しないが, コンテナからイメージに戻して, 新しいイメージを作ることもできる. メジャーなソフトウェアや言語, OSのイメージはDockerHubで提供されており参考文献[8]から検索をすることができる.
イメージとコンテナの関係は次のような図で表される.

![img](./img/docker-image-container.png)

Ubuntuイメージを取得して, イメージからコンテナを作成してみる.
DockerHubで提供されているイメージをローカルにコピーすることをdocker pullという. 試しにUbuntuのDockerイメージをdocker pullしてみる. DockerHubで「ubuntu」と検索をかけると次のようにUbuntuのイメージが提供されていることがわかる. このような公式が提供しているイメージを公式イメージと呼ぶ.

![img](./img/ubuntu-official-image.png)

ubuntuイメージのページにdocker pullコマンドが記述されているので, これをそのまま実行してみる. 
```
$ docker pull ubuntu
Using default tag: latest
latest: Pulling from library/ubuntu
cf92e523b49e: Pull complete 
Digest: sha256:35fb073f9e56eb84041b0745cb714eff0f7b225ea9e024f703cab56aaa5c7720
Status: Downloaded newer image for ubuntu:latest
docker.io/library/ubuntu:latest
```

ubuntuイメージがdocker pullできたか確認する. 次にコマンドを実行するとローカルにあるイメージの一覧が表示される. REPOSITORYがubuntuのものがあればdocker pullが成功している.
```
$ docker images
REPOSITORY             TAG       IMAGE ID       CREATED         SIZE
ubuntu                 latest    216c552ea5ba   2 weeks ago     77.8MB
hello-world            latest    feb5d9fea6a5   13 months ago   13.3kB
```

docker pullしたubuntuイメージを起動して, コンテナを作成する. 次のコマンドを実行するとubutnuイメージからコンテナを作成してbashに入ることができる. コマンドの意味については後で述べる.
```
docker run -i -t ubuntu bash

root@7b80b49a78ac:/# cat /etc/issue
Ubuntu 22.04.1 LTS \n \l

root@7b80b49a78ac:/# exit
exit
```

このようにして, DockerHubからイメージを取得して, 開発環境となるコンテナを立てることができた. ここからはイメージからコンテナを生成して開発を行うときに用いるコマンドについて説明する.  
まず, コンテナの状態確認を行うコマンドであるdocker psについて説明する. 次のコマンドを実行すると実行中のコンテナ一覧を見ることができる. 
```
docker ps
```
例えばUbuntuコンテナを実行しているときに, docker psコマンドを実行すると次のような出力が得られる. 出力は左からコンテナID, 実行中コマンド, コンテナ生成に使用したイメージ, 生成された時間, コンテナの状態, コンテナ名である. STATUSがUPになっていればコンテナが実行中の状態である.
```
$ docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED          STATUS          PORTS     NAMES
17d884f94333   ubuntu    "bash"    11 seconds ago   Up 10 seconds             competent_leakey
```

-aオプションをつけることで状態に関わらず, ローカル環境に保有している全てのコンテナを確認することができる.  
```
$ docker ps -a
CONTAINER ID   IMAGE                  COMMAND                  CREATED          STATUS                       PORTS                                       NAMES
af0cd66863c5   ubuntu                 "bash"                   10 seconds ago   Up 9 seconds                                                             keen_cartwright
2b7fae17c714   hello-world            "/hello"                 7 days ago       Exited (0) 7 days ago                                                    confident_cannon
```

次に, docker runコマンドの詳細について説明する. オプションを考えなくてもなんとなく実行できるでしょう～？という人向けに, 最初にあるあるの失敗をやっておく. 次のコマンドを実行してubuntuイメージからubuntuコンテナを生成できるか試してみる. ubuntuコンテナのbashが帰ってくるんでしょ？とウキウキして実行すると, 何も起きなかったかのように通常のbashが帰ってくる. docker ps -aでコンテナ状態を確認すると, 確かにコンテナが生成された記録があるが, exit状態になっている. 実はコンテナはプロセスを実行するためのものであるため, Root Processと呼ばれるプロセスが終了すると, プロセスも終了するようになっている. 今の失敗例ではコンテナのRoot Processとしてubuntuのbashが立ち上がるが, 標準入力もターミナルも設定していないためすぐにbashが終了する. このためコンテナが生成された記憶があるが, 何も起きなかったように見えるのである.
```
$ docker run ubuntu
```

では初めにubuntuコンテナを生成したときは何故bashがすぐに終了しなかったのだろうか? その答えが-i -tというオプションである. オプション内容は後述するがdocker runでコマンドを間違えると起動しない！接続できない！ということがよくある. そんなときはまずdocker runコマンドを確認してみよう.  
いよいよ本題のdocker runコマンドのオプションについて説明する. まず-i, -tオプションについて説明する. -iはコンテナに標準入力をアタッチすることを意味する. -tはコンテナにターミナルをアタッチする(ttyを利用する)ことを意味する. この2つのオプションを用いて次のコマンドを実行すると, ubuntuコマンドに標準入力とターミナルがアタッチされてubuntuのbashが返ってくる. -iと-tはよく一緒に使うため-itと記述することが多い.
```
docker run -i -t ubuntu
```

次に--rmオプションについて説明する. このオプションはコンテナの終了時にコンテナを削除するというオプションである. 一時的に何かの環境をテストして, それが終わったら跡形もなくコンテナを削除するときに用いる. --rmオプションなしでdocker run ubuntuコマンドを実行すると, その都度実行したコンテナが残るが次のコマンドを実行するとコンテナ終了時にコンテナが削除される.
```
docker run --rm -it ubuntu
```

--nameオプションについて説明する. --nameオプションなしでdocker runを実行した場合, keen_cartwrightやconfident_cannonのようなランダムな名前がコンテナに割り当てられる. 同じイメージから複数のコンテナを立てるような環境構築が必要な場合や, バージョンを分けたい場合, 開発用とデプロイ用でコンテナを分ける場合にはコンテナ名を割り振っておくとわかりやすい. そこで--nameオプションを次のように用いる. この例では生成したubuntuコンテナにdev_ubuntuという名前をつけている.
```
docker run --rm -it --name dev_ubuntu ubuntu
```

-dオプションについて説明する. -dオプションはコンテナを作成した後, Root Processの標準入出力からデタッチするオプションである. -itオプションでRoot Processに標準入出力をアタッチすると, デタッチは行われていないためコンテナのプロンプトが返ってくる. 一方で-dオプションをつけた場合はRoot Processに標準入出力をアタッチした後でデタッチが行われるためホストのプロンプトが返ってくる.  
このほかポート番号を設定するコマンドや, リソースを制限するコマンドを代表とする多くのオプションがある. 必要な時に参考文献[9]を参照してほしい.

最後に生成したコンテナを停止・再起動・削除する方法とイメージを削除する方法について説明する. コンテナやイメージのライフサイクルに関するコマンドは次図のようになっている.

![img](./img/docker-lifecycle.png)

docker runコマンドで新規にコンテナを生成して, 実行している状態であるとする.
```
$ docker run -d -it --name dev_ubuntu ubuntu
``` 
この状態でdockerコンテナを停止したいときは次のコマンドを実行する. すなわちdocker stopコマンドで停止したいコンテナ名もしくはコンテナIDを指定することでコンテナを停止できる.
```
docker stop dev_ubuntu
```
1回生成したコンテナは次のコマンドで再起動することができる. docker startコマンドで再起動したいコンテナ名もしくはコンテナIDを指定する.
```
docker start dev_ubuntu
```
コンテナが不要になった場合には, 次のコマンドでコンテナを削除できる. 
```
docker rm dev_ubuntu
```
イメージが不要になった場合は次のコマンドでイメージを削除できる. 削除したいイメージから生成されたコンテナがある場合にはエラーになるためコンテナを先に削除すること.
```
docker rmi ubuntu
```

<div id="4.3">

### 4.3 Dockerfile
前節では公式イメージからコンテナを生成して開発環境を構築する方法について説明した. 本節では公式イメージをカスタムして, 自分の要望にあった開発環境を構築する方法について説明する.  
Pythonで開発を行うことを考えてみる. このとき開発環境を構築する過程で次のような操作を行うことがあるだろう.

- apt updateやinstallを行う
- pipでライブラリをインストールする
- 開発に用いるディレクトリ・ファイルを構成する.
- ホストのファイルをコンテナに送信する.
- 構築したコンテナのPythonインタプリタで作業を行う.

これらの作業をDockerfileと呼ばれるものに記述して, そのDockerfileを実行するとカスタムされたイメージを作成することができる. 先のpythonの例でDockerfileを作成してみる. ここで作成するDockerfileおよびコンテナ構築に必要なファイルは/src/dev_pythonに設置してある. ここでは次の要件を満たす環境を構築する.
- apt updateが行われている.
- python3.8が動作する.
- 作業ディレクトリが/devである.
- numpy, pandas, matplotlibの3つのライブラリが使える.
- git, vim, unzipが使える.

これらの要件を満たすようにDockerfileを構成してみる. /src/dev_python/Dockerfileを次に示す. またpip installするライブラリを/src/dev_python/requirements.txtにまとめて記述しておく. Dockerfileの内容について説明する. まずFROMでイメージを作成するためのベースとなるイメージであるベースイメージを記述する. ここではpython3.8とした. 細かいバージョンをpython3.8:latestのように指定することもできる. 次にWORKDIRでコンテナの作業ディレクトリを指定する. docker run -itでコンテナを起動したときにWORKDIRで指定したディレクトリでプロンプトが立ち上がる.  
RUNコマンドはコンテナ上で実行されるコマンドを記述することができる. pubキーの設定は飛ばしてaptで説明すると, apt updateやapt installがコンテナ上で実行されるということである. ここではapt updateを行った後にgit, vim, unzipをインストールしている. 次にpip installを行う. まずCOPYコマンドでホストのrequirements.txtをコンテナの/tmpにコピーする. COPyコマンドでは「COPY ホストのファイルパス コンテナのファイルパス」のようにしてコピーするファイルを指定する. 次にRUNコマンドでpip installを行っている. コピーしたrequirements.txtに記述されたライブラリをまとめてインストールしている. requirements.txtにはnumpy, pandas, matplotlibの3つが記述されている. Dockerfileの最後にCMDコマンドが記述されている. このコマンドはイメージ作成時に実行されるのではなく, コンテナ実行時に自動実行されるコマンドを指定するためのものである. docker run -it ubuntuを実行したときにbashが返ってきたが, これはCMDで/bin/bashが指定されているためである. ここでも同じように/bin/bashを指定している. もしpythonインタプリタが返って来てほしいような場合はCMD ["python3"]のように記述する.

/src/dev_python/Dockerfile
```Dockerfile
# ベースイメージ
FROM python3.8

# 作業ディレクトリの指定
# 作業ディレクトリがない場合は生成される
WORKDIR /dev

# pubキーの追加
RUN apt-key adv --fetch-keys https://developer.download.nvidia.com/compute/cuda/repos/ubuntu1804/x86_64/3bf863cc.pub
RUN apt-key adv --fetch-keys https://developer.download.nvidia.com/compute/cuda/repos/ubuntu1804/x86_64/7fa2af80.pub

# apt実行
RUN apt update \
    && apt install -y \
    git \
    vim \
    unzip

# ホストのファイルをコンテナにコピー
COPY requirements.txt /tmp/

# pip install実行
RUN pip install --no-cache-dir -r /tmp/requirements.txt

# CMDはコンテナ生成時ではなくdocker run時に実行される
CMD ["/bin/bash"]
```

/src/dev_python/requirements.txt
```txt
numpy
pandas
matplotlib
```

/src/dev_pythonでDockerfileを実行してイメージを作成する. 次に示すdocker buildコマンドを実行する. docker buildコマンドはDockerfileのパス, -tオプションでイメージ名を指定している.

```
$ docker build ./ -t dev_python3.8
[+] Building 93.4s (12/12) FINISHED                                                                                                                                                                                                    
 => [internal] load build definition from Dockerfile                                                                                                                                                                              0.0s
 => => transferring dockerfile: 785B                                                                                                                                                                                              0.0s
 => [internal] load .dockerignore                                                                                                                                                                                                 0.0s
 => => transferring context: 2B                                                                                                                                                                                                   0.0s
 => [internal] load metadata for docker.io/library/python3.8:latest                                                                                                                                                               0.0s
 => [1/7] FROM docker.io/library/python3.8                                                                                                                                                                                        0.1s
 => [internal] load build context                                                                                                                                                                                                 0.0s
 => => transferring context: 66B                                                                                                                                                                                                  0.0s
 => [2/7] WORKDIR /dev                                                                                                                                                                                                            0.0s
 => [3/7] RUN apt-key adv --fetch-keys https://developer.download.nvidia.com/compute/cuda/repos/ubuntu1804/x86_64/3bf863cc.pub                                                                                                    0.8s
 => [4/7] RUN apt-key adv --fetch-keys https://developer.download.nvidia.com/compute/cuda/repos/ubuntu1804/x86_64/7fa2af80.pub                                                                                                    0.9s 
 => [5/7] RUN apt update     && apt install -y     git     vim     unzip                                                                                                                                                         88.7s 
 => [6/7] COPY requirements.txt /tmp/                                                                                                                                                                                             0.0s 
 => [7/7] RUN pip install --no-cache-dir -r /tmp/requirements.txt                                                                                                                                                                 2.2s 
 => exporting to image                                                                                                                                                                                                            0.6s 
 => => exporting layers                                                                                                                                                                                                           0.5s 
 => => writing image sha256:933aa0089ddd29c1155c599dff111ca4b81efb08f2d593545e79bec8000f5c73                                                                                                                                      0.0s
 => => naming to docker.io/library/dev_python3.8  
```

イメージが作成できたから, コンテナを生成して要件にあった環境構築が行えているか確認する. CMDで/bin/bashを指定したからdocker runコマンドを実行するとbashが返ってくるはずである. 実際に実行してみると確かにbashが返ってきた. さらに起動時のディレクトリが/devになっている. これはWORKDIRで設定した通りになっている.
```
$ docker run -it dev_python3.8
root@c5aeecb78ce3:/dev# 
```

pythonが起動するか確認する. 次のコマンドを実行する. 確かにpythonが実行できることが確認できた.
```
root@c5aeecb78ce3:/dev# python3
Python 3.8.10 (default, Nov 26 2021, 20:14:08) 
[GCC 9.3.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> print("hello world!")
hello world!
>>> exit()
```

pythonのライブラリがインストールされているか確認する. 次のコマンドを実行する. 必要な部分だけ示すが, 確かにnumpy, pandas, matplotlibがインストールされていることが確認できた.
```
root@c5aeecb78ce3:/dev# pip list
Package                      Version
---------------------------- -------------------
matplotlib                   3.5.1
numpy                        1.22.2
pandas                       1.4.1
```
gitがインストールされていることも確認しておく. 次のコマンドを実行する. 確かにapt installが実行されていることが確認できた.

```
root@c5aeecb78ce3:/dev# git --version
git version 2.25.1
```

このように, 公式イメージをベースイメージとして自分のやりたいことにあった環境をDockerfileを用いて構築することができる. この部分については実際に手を動かして自分好み, または必要な環境構築を行ってみてほしい.

<div id="4.4">

### 4.4 docker-compose
工事中

## 参考文献

[1]アイティーエム, 仮想化技術について解説ホスト・ハイパーバイザー・コンテナの違いとは？  
https://www.itmanage.co.jp/column/virtualization-server-integration/


[2]atmarkit, 図解で理解できる（はず）Microsoftの仮想化技術――Windows上で稼働するLinux、動かしているのはどのテクノロジー？（その2）  
https://atmarkit.itmedia.co.jp/ait/articles/1710/24/news010.html

atmarkit, 完全なLinuxがWindows 10上で稼働する？　「WSL 2」とは  
[3]https://atmarkit.itmedia.co.jp/ait/articles/1906/14/news019.html

[4]大滝みや子, 岡嶋祐史, "令和04年【春期】【秋期】応用情報技術者合格教本", pp.195, 株式会社技術評論社, 2021

[5]Docker https://www.docker.com/

[6]Docker Desktop https://www.docker.com/products/docker-desktop/

[7]WSL2 Ubuntu に Docker をインストールする https://zenn.dev/fehde/articles/ea0e8a0a0a1de4

[8]DockerHub https://hub.docker.com/

[0]Docker run リファレンス  https://docs.docker.jp/engine/reference/run.html#env