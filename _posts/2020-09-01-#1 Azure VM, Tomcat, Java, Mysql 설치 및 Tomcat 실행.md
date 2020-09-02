#### Azure VM
- 설치환경 : Ubuntu 18.06 

VM의 리눅스 서버를 하나 생성한다. 이미지 크기나 CPU, RAM 등은 기본으로 세팅 되있는걸 쓴다.

사용자 id와 pw를 설정하고 VM을 생성한다.

- 네트워킹 

  기본적으로 SSH 포트가 열려있을 것이다. 여기에 추가적으로 내가 사용하는 포트는 `8080(Tomcat)`,  `3306(MySQL)`, `8443(Tomcat HTTPS)`이 필요하므로 인바운드 포트 규칙에서 추가한다.

  AllowVnetInBound, AllowAzureLoadBalancerInBound, DenyAllInBound 포트는 건드리지 않고 냅둔다.

  VM 생성과 네트워킹이 완료되면 `putty`, `xshell` 등의 프로그램을 사용해서 VM에 접속한다.
  나는 [MobaXterm]: https://mobaxterm.mobatek.net/download.html  을 사용했다



#### Java 환경 세팅

- Java

  Java는 1.8버전을 사용한다. java 11을 사용하고 싶었지만 프로젝트 사정상 1.8을 사용해야 했다. openjdk를 사용하지 않고 https://www.oracle.com/kr/java/technologies/javase/javase-jdk8-downloads.html 에서 직접 다운을 받아 리눅스에 수동 설치한다.

  

- Tomcat 

  Tomcat도 마찬가지로 수동설치한다. https://tomcat.apache.org/download-90.cgi 에서 다운받아서 사용한다. 리눅스 환경이기 대문에 tar.gz 파일을 다운받고 압축 해제한다.

  

  다운받은 파일을 VM에 넣고 압축해제한다

  ```shell
  tar -zxvf jdk-8u261-linux-x64.tar.gz jdk1.8.0_261/ 
  tar -zxvf apache-tomcat-9.0.37.tar.gz
  
  ```

  압축 해제된 폴더를 `/usr/local/lib`밑으로 옮기자. 안옮겨도 되는데 다른 사람들도 대개 이 위치에 두고, 나중에 경로 찾을때도 편하다.

  ```shell
  sudo mv jdk1.8.0_261/ apache-tomcat-9.0.37/ /usr/local/lib/
  ```

  

- Mysql 설치

  Mysql은 5.7버전을 사용한다. 이건 그냥 `sudo apt-get mysql` 로 받자. 설치 및 환경 세팅은 https://shlee0882.tistory.com/240 를 참고했다.

  - 설치

  ```shell
  sudo apt-get install mysql-server-5.7
  ```

  - root pw 설정 및 접속 ip 설정

    ```mysql
    mysql> USE mysql;
    mysql> UPDATE user SET plugin='mysql_native_password' WHERE User='root';
    mysql> FLUSH PRIVILEGES;
    mysql> exit;
    $ service mysql restart
    
    mysql> grant all privileges on *.* to 'root'@'%' identified by '0000'; 
    mysql> flush privileges; 
    mysql> select host, user, authentication_string  from user; 
    mysql> exit
    $ service mysql restart
    ```

  - 로컬PC 접속 테스트

    - bind-address 주석 해제
    - workbench에서 접속



- 환경변수 세팅

  java와 tomcat가 설치된 폴더가 아닌 곳에서 사용하기 위해서는 환경변수에 등록을 해줘야 한다.
  
  환경변수를 설정하는 위치는 `/etc/profile` 이다. 현재 `root`에서 작업중이 아니므로 `sudo` 권한으로 접속 한 뒤 파일의 맨 마지막에 아래와 같이 추가한다. 
  
  ```shell
  .
  .
  
  export JAVA_HOME=/usr/local/lib/jdk1.8.0_261
  export CLASSPATH=$JAVA_HOME/jre/lib/ext:$JAVA_HOME/lib/tools.jar
  export CATALINA_HOME=/usr/local/lib/apache-tomcat-9.0.37
  PATH=$PATH:$JAVA_HOME/bin:$CATALINA_HOME/bin
  ```
  
  profile 파일의 변경사항을 적용하기 위해 `source /etc/profile` 을 입력한 뒤 java 또는 javac 를 쳐보면 긴 글이 나올 것이다. 그렇게 되면 성공한 것이다.
  
  

#### Tomcat 접속

위의 과정을 모두 마치고 톰캣이 설치된 위치로 가서 실행을 하면 고양이 페이지가 뜰 것이다. 톰캣이 설치된 위치는 `/usr/local/lib/apache-tomcat-9.0.37/` 이다. 



해당 경로로 가면 `webapps`, `temp`, `logs`, `lib`, `conf`, `bin` 등등 다양한 폴더가 있는데 WAS를 구축하는데 모두 다 쓰인다. 우선, 톰캣의 실행파일은 **bin** 폴더 아래에 있다. 



해당 위치로 간 뒤 `./startup.sh`를 해서 실행한다. 이후 chrome에서 `http://{VM에서 할당받은 퍼블릭 ip 주소}:8080` 으로 접속했을 때 고양이 페이지가 뜨면 성공한 것이다. 



만약 페이지를 찾을 수 없는 오류가 발생하면 아래 명령어로 톰캣 서버가 제대로 떠있는지 확인한다.

- ps -ef |grep tomcat 

  뭔가 길게 나와야한다. `{username}+ 120660 113238  0 13:29 pts/0    00:00:00 grep --color=auto tomcat` 처럼 1줄만 달랑 나오면 안뜬것

- netstat -nlpt

  `tcp6       0      0 :::8080                 :::*                    LISTEN      1` 

  8080 포트가 listen 상태로 되어있어야 한다.

톰캣을 종료하려면 `./shutdown.sh` 명령어를 입력하면 된다



다음 포스트에서는 local에서 만들고 정상적으로 실행이 되는 java servlet 프로젝트를 linux에서 실행하는 과정을 포스팅할 예정이다.



