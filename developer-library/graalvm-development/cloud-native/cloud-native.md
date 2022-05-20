# GraalVMとDockerコンテナ

## 概要

この演習では、演習4で作成したSpring BootのRESTFulサービスをDockerコンテナとして作成、起動します。jar形式とnaitve image形式でそれぞれコンテナを作成し、両者の起動スピードおよびコンテナイメージのサイズを比較します。以下３種類のDockerイメージを作成します。
* Oracle JDKベースのfat jar用のDockerイメージ
* Oracle Linux7ベースのnative image用のDockerイメージ
* Distrolessベースのほぼ静的なnative image用のDockerイメージ

*所要時間: 15分*

### ■目標
* 3種類のベース・イメージを使用して、より軽量で高速起動するコンテナイメージを作成

### ■前提条件
* 演習4「GraalVMとマイクロサービスフレームワークによるRESTFulサービス開発」を実施済みであること  

## Task 1: fat jarのDockerイメージ作成
1. Oracle JavaのベースDockerイメージをダウンロードし、演習4のTask2で作成したSpring BootのjarファイルをDockerコンテナとして作成、実行します。spdemo配下に、Dockerfile.jdkという名前のDockerファイルを作成します。

    ```
    <copy>nano Dockerfile.jdk</copy>
    ```
       
    以下の内容をDokcerfile.jdkに貼り付け、ファイルを保存します。
    ```
    <copy>
    FROM container-registry.oracle.com/java/jdk:11-oraclelinux7

    EXPOSE 8080

    COPY target/demo-0.0.1-SNAPSHOT-exec.jar demo.jar
    CMD ["java","-jar","demo.jar"]
    </copy>
    ```
       
2. Dockerイメージを作成します。以下のコマンドをspdemo配下で実行します。

    事前に[container-registry.oracle.com](https://container-registry.oracle.com)よりpullしたベース・イメージをVM上にダウンロードします。
    > **Note:** 通常はOracle公式のJavaのDockerイメージをOracleコンテナレジストリからpullできますが、今回ハンズオン用VMのネットワークの制限により事前pullしたイメージを配布する形を取ります。 
    ```
    <copy>
    wget https://objectstorage.us-ashburn-1.oraclecloud.com/p/LNAcA6wNFvhkvHGPcWIbKlyGkicSOVCIgWLIu6t7W2BQfwq2NSLCsXpTL9wVzjuP/n/c4u04/b/livelabsfiles/o/developer-library/jdkimage.tar.gz
    </copy>
    ```
    ```
    <copy>
    sudo docker load < jdkimage.tar.gz
    </copy>
    ```
    
    JDKベース・イメージがロードされているのを確認します。  
    ```
    <copy>
    sudo docker images
    </copy>
    ```
    
    コンテナイメージをビルドします。  

    ```
    <copy>
    sudo docker build -f Dockerfile.jdk -t spring-jdk .
    </copy>
    ```
    ![docker in spring](images/docker-spring.png)
       
3. コンテナイメージが生成されたことを確認し、コンテナを起動します。
    ```
    <copy>
    sudo docker images
    </copy>
    ```
    ![docker in spring1](images/docker-spring1.png)
       
    ```
    <copy>
    sudo docker run --rm -p 8080:8080 spring-jdk:latest
    </copy>
    ```
    ![docker in spring2](images/docker-spring2.png)

    RESTfulサービスの起動時間を確認します。この例では1.441秒です。  

4. 別ターミナルを立ち上げ、以下のコマンドを実行し、HTTPリクエストからレスポンスが正常にリターンされることを確認します。
        
    ```      
    <copy>curl http://localhost:8080/greeting</copy>
    ```
    ![docker in spring3](images/docker-spring3.png)

5. Ctrl+CでDockerコンテナからexitします。

    > **Note:** コンテナが起動しているターミナルでSSH接続が既に切断されている場合、SSH接続を再度実行し、sudo docker ps -a　を実行し、コンテナが実行中かどうかを確認してください。

## Task 2: native imageのDockerイメージ作成

1. JDKを含まないベース・イメージとnative imageでコンテナを作成します。spdemo配下に、Dockerfile.nativeという名前のDockerファイルを作成します。

    ```
    <copy>nano Dockerfile.native</copy>
    ```
       
    以下の内容をDokcerfile.nativeに貼り付け、ファイルを保存します。
    ```
    <copy>
    FROM container-registry.oracle.com/os/oraclelinux:7-slim
    COPY target/demo app
    ENTRYPOINT ["/app"]
    </copy>
    ```
       
2. Dockerメージをビルドします。以下のコマンドをspdemo配下で実行します。

    ```
    <copy>
    wget https://objectstorage.us-ashburn-1.oraclecloud.com/p/LNAcA6wNFvhkvHGPcWIbKlyGkicSOVCIgWLIu6t7W2BQfwq2NSLCsXpTL9wVzjuP/n/c4u04/b/livelabsfiles/o/developer-library/ol7image.tar.gz
    </copy>
    ```
    ```
    <copy>
    sudo docker load < ol7image.tar.gz
    </copy>
    ```

    JDKを含まないOSのみのベース・イメージがロードされているのを確認します。  
    ```
    <copy>
    sudo docker images
    </copy>
    ```
    コンテナイメージをビルドします。

    ```
    <copy>
    sudo docker build -f Dockerfile.native -t spring-native .
    </copy>
    ```
    ![docker in spring4](images/docker-spring4.png)

       
3. コンテナイメージが生成されたことを確認し、コンテナを起動します。

    ```
    <copy>
    sudo docker images
    </copy>
    ```
    ![docker in spring5](images/docker-spring5.png)

    ```
    <copy>
    sudo docker run --rm -p 8080:8080 spring-native
    </copy>
    ```
    ![docker in spring6](images/docker-spring6.png)

4. 別ターミナルを立ち上げ、以下のコマンドを実行し、HTTPリクエストからレスポンスが正常にリターンされることを確認します。
    ```      
    <copy>curl http://localhost:8080/greeting</copy>
    ```
    ![docker in spring3](images/docker-spring3.png)
       
    RESTfulサービスの起動時間を確認します。この例では0.022秒です。JITモードより100倍速く起動できました。

5. Ctrl+CでDockerコンテナからexitします。

## Task 3: ほぼ静的なnative imageのDockerイメージ作成

1. より軽量なコンテナを作成するため、ベース・イメージをGoogleが公開しているdistrolessベース・イメージを使用します。distrolessは、パッケージマネージャやシェルを含まない、アプリケーション実行に特化したコンテナイメージです。ほぼ静的なnative imageは実行時標準Cライブラリ(glibc)のみ参照し、それ以外のすべての依存ライブラリを静的にリンクし、ビルドされます。pom.xmlに以下の部分を追加して、ぼぼ静的なnative imageとして再度ビルドします。

    spdemo配下でpom.xmlを開きます。
    ```
    <copy>nano pom.xml</copy>
    ```
    以下の```<configuration>```部分を、```<profile>```タグ-->```<build>```タグ-->```<plugin>```タグの中に追加します。  

    ```<buildArgs>```タグの中に```StaticExecutableWithDynamicLibC```というパラメータを指定します。このパラメータによりnative imageビルド時標準Cライブラリ```libC```以外の依存ライブラリを全て事前に静的にリンクします。

    ```
    <copy>
    <configuration>
        <!-- add native-image build arguments -->
        <buildArgs>
          <buildArg>-H:+StaticExecutableWithDynamicLibC</buildArg>
        </buildArgs>
    </configuration>
    </copy>
    ```
    ![docker in spring3](images/docker-spring9.png)

    Ctrl＋Xを押し、内容保存の確認メッセージに対し、"Y"を入力し、Enterを押下してソースファイルを保存します。

2. spdemo配下に、以下のコマンドを実行し、native imageを再度ビルドします。

    ```
    <copy>./mvnw -Pnative -DskipTests package</copy>
    ```

3. ビルドが正常に終了したことを確認した上、spdemo配下に、Dockerfile.native-lightという名前のDockerファイルを作成します。

    ```
    <copy>nano Dockerfile.native-light</copy>
    ```
       
    以下の内容をDokcerfile.native-lightに貼り付け、ファイルを保存します。
    ```
    <copy>
    FROM gcr.io/distroless/base
    COPY /target/demo app
    ENTRYPOINT ["/app"]
    </copy>
    ```
       
4. Dockerコンテナをビルドします。以下のコマンドをspdemo配下で実行します。

    ```
    <copy>
    wget https://objectstorage.us-ashburn-1.oraclecloud.com/p/LNAcA6wNFvhkvHGPcWIbKlyGkicSOVCIgWLIu6t7W2BQfwq2NSLCsXpTL9wVzjuP/n/c4u04/b/livelabsfiles/o/developer-library/distroless.tar.gz
    </copy>
    ```
    ```
    <copy>
    sudo docker load < distroless.tar.gz
    </copy>
    ```

    ```
    <copy>
    sudo docker build -f Dockerfile.native-light -t spring-native-light .
    </copy>
    ```
       
5. コンテナイメージが生成されたことを確認し、コンテナを起動します。
    ```
    <copy>
    sudo docker images
    </copy>
    ```
    ![docker in spring7](images/docker-spring7.png)
       
    ```
    <copy>
    sudo docker run --rm -p 8080:8080 spring-native-light:latest
    </copy>
    ```

    ![docker in spring8](images/docker-spring8.png)
    RESTfulサービスの起動時間を確認します。この例では0.026秒です。JITモードより100倍速く起動できました。

6. 別ターミナルを立ち上げ、以下のコマンドを実行し、HTTPリクエストからレスポンスが正常にリターンされることを確認します。
    ```      
    <copy>curl http://localhost:8080/greeting</copy>
    ```

7. Ctrl+CでDockerコンテナからexitします。  

    以下は3種類のDockerコンテナイメージをベースに作成したコンテナの起動時間とイメージサイズの比較です。

    | アプリ形式 | fat jar | native image | ほぼ静的なnaitve image |
    | --- | --- | --- | --- |
    | 起動時間(秒) | 1.441 | 0.022  | 0.027 |
    | コンテナイメージサイズ(MB) | 594 | 184  | 94.2  |
  
## Acknowledgements

- **Created By/Date** - Jun Suzuki, Java Global Business Unit, April 2022
- **Contributors** - 
- **Last Updated By/Date** - Jun Suzuki, April 2022
