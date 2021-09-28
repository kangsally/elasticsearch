# elasticsearch

#### Elasticsearch 설치

1. elasticsearch 와 kibana 다운로드 -> 두개는 같은 버전이어야 함

   ```
   wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.14.1-linux-x86_64.tar.gz
   ```

   ```
   wget https://artifacts.elastic.co/downloads/kibana/kibana-7.14.1-linux-x86_64.tar.gz
   ```

   ** 만약 wget 설치가 안되어있다면 다음 명령어로 설치 후  진행 `sudo yum install wget`  

   

2. 압축해제

   ```
   tar xfz elasticsearch-7.14.1-linux-x86_64.tar.gz
   ```

   ```
   tar xfz kibana-7.14.1-linux-x86_64.tar.gz  
   ```

   

3.  files 폴더를 만들어서 압축파일은 따로 보관

   ```
   mkdir files
   ```

   ```
   mv *.tar.gz files/  
   ```

   

4.  elasticsearch와 kibana 폴더명을 간단한 이름으로 변경해줌

   ```
   mv elasticsearch-7.14.1 es-714
   ```

   ```
   mv kibana-7.14.1-linux-x86_64 kb-714  
   ```



5. Elastic search.yml 에서 기존 #로 주석처리 된 내용들 참고해서 필요한대로 설정

   ```
   vi config/elasticsearch.yml
   ```

   - cluster.name: '클러스터 이름'

   - node.name: '노드 이름'

   - path.data: /path/to/data   // production 환경에서는 elasticsearch가 설치된 폴더와 분리해주는 것이 좋음

   - network.host: 192.168.0.1 // 이걸 외부ip로 설정해주면 외부에서도 접근이 가능함, 아니면 local 에서만 접근 가능  -> 외부 ip로 설정하는 순간 elasticsearch는 여러가지 부트스트랩 체크를 하게 됨

   -  http.port: 9200 // production 환경에서는 default 포트가 아닌 다른 번호로 지정해주는 것이 좋음

   - discovery.seed_hosts: ["host1", "host2"]  // 다른  호스트를 찾을 때 찾을 네트워크 주소를 입력-
   - cluster.initial_master_nodes:  ["node-1",  "node-2"]  // 여러 노드들이 있을 때  첫번째 마스터 후보들이 될 노드들을 지정해줌  



6. elasticsearch 의 bin폴더에 들어가서 elasticsearch를 실행해준다.  (하기전에 10번 참고)

   ```
   bin/elasticsearch -d
   ```

   ** 실행 후 다른 터미널을 열어 `curl -XGET localhost:9200`로 실행되는지 확인 가능

   ** bin/elasticsearch 으로 실행하면 터미널을 끄는 순간 멈출 수도  있어서  -d 옵션을 데몬으로 실행하게 하면 백그라운드에서 실행됨  

   

7. elasticsearch 실행 후 log 폴더가면 log가 새로 생겨있어서 확인 가능

   ```
   tail -f logs/es-cluster-1.log  
   ```

   

8. 백그라운드에서 진행중인 elasticsearch 프로세스  찾기

   ```
   ps  -ef | grep  elasticsearch  
   ```

   

9. 백그라운드 진행중인 elasticsearch 를 멈추려면 8번에서 찾은 프로세스 id로 멈출 수  있음

   ```
   kill 2955  
   ```



10. 하지만 6번,  9번처럼 하면 매번 id를 찾아야하므로 불편함 그래서 다음과 같이 하자

    ```
    bin/elasticsearch -d -p es.pid
    ```

    ** -p옵션은 es.pid라는 파일을  생성해서 그 안에다가 아이디를 넣어준다.

    

    id 확인 방법은 es.pid 를 열어보면 됨

    ```
    cat es.pid
    ```

    ** 라인 제일 앞에 id가 나온다.

    

    다음과  같이 하면 9번과 같은  의미가 된다  -> es.pid도 자동으로 삭제됨

    ```
    kill `cat  es.pid`  
    ```

    ​

11. 10번 과정들을 실행파일(쉘 파일)로 만들어서 사용 가능

    - 시작 파일 만들기

    ```
    vi start.sh
    ```

    열려진 텍스트 편집기 안에 다음 내용을 입력 후 ```:wq``` 로 저장

    ```
    bin/elasticsearch -d -p es.pid
    ```

    

    - 끝내기 파일 만들기

    ```
    vi stop.sh
    ```

    열려진 텍스트 편집기 안에 다음 내용을 입력 후 ```:wq``` 로 저장

    ```
    kill `cat es.pid`  
    ```

    

    - 파일에 실행 권한 부여해주기 ```chmod```

    ```ls -la``` 를 하면 각 파일들 앞에 -drwxr-xr-x 와 같은 값이 있음 -> 

    {(1)종류}{(3)user}{(3)group}{(3)other(모든)} 을 나타냄 d: directory/ -: file/ r : read / w - write / x - execute : 에 해당하는 3비트씩을 십진수 (0~7) 로 입력예) 754 == 111110100 = rwxr-xr-- 
    user 는 read/write/execute, group 은 read/execute, other 는 read 가능.

    ```
    chmod 755 *.sh
    ```

    이렇게 하면 start.sh와 stop.sh 에 실행권한이 생김    
    
    
    
    


#### Elasticsearch 환경설정  



1.  jvm.options 설정

   ```
   vi config/jvm.options
   ```

   테스트 편집기에 하기 입력하면 힙메모리 설정 가능 (기본 1gb) 단, 30gb를 넘지 않도록 한다. (Xms와 Xmx는 같은 값 넣어야 함)

   ```
   -Xms512m
   -Xmx512m
   ```  

   

2. Elastic search.yml 설정 - https://esbook.kimjmin.net/02-install/2.3-elasticsearch/2.3.2-elasticsearch.yml

   ```
   vi config/elasticsearch.yml
   ```

     매 라인의 들여쓰기가 정확하지 않으면 같은 레벨에 설정되어야 할 값들이 하위 레벨 설정으로 들어가는 경우가 있dma. 또한**`<key>: <value>`**가운데에 있는 콜론**`:`**과**`<value>`**값 사이에는 반드시 공백이 있어야 하며 붙여쓰게 되면 오류가 발생하기 때문에 띄어쓰기에 항상 주의해야 함.

   

   - data 경로 바꾸기  - 현재는 es-714 폴더 하위에 data 폴더가 있음 -> data 폴더를 es-714와 동일한 계층에 생성하고 거기로 data path 바꾸는 설정  (위치는 pwd로 알 수 있다)
     항상 elasticsearch를 사용하는 user가 data, log 폴더 쓰기 권한을 가지고 있어야 한다. ls -la 로 확인 가능

     ```
     path.data: "/home/kimjmin/data"
     path.logs: "/home/kimjmin/logs"
     
     // 아래처럼 써도 됨
     path:
     	data: "/home/kimjmin/data"
     	logs: "/home/kimjmin/logs"
     ```

   

   - 하기 처럼 command 라인으로 설정 가능

     ```
     bin/elasticsearch -E node.name="node-new"
     ```

     

- network.host 값의 경우

  - network.host 값을 지정하지 않으면 같은 localhost에서 밖에 접근이 안된다. 그래서 외부에서 접근가능하게 하려면 지정해줘야 함. (+방화벽도 열어줘야 함)

  - `_site_` 변수값이면 로컬 네트워크 주소가 되는데, VM Instance에서 Internal IP가 된다, 직접 값을 넣어도 되고 그냥 변수값을 넣어도 됨 - 강의에서는 이걸로 사용을 하고 나중에 GCP 에서 firewall 설정 추가를 하면(아래 참고) 아래 _global_ 변수 설정 필요 없이 External IP로 접근할 수 있음 (아래처럼 local 도 추가하면 내부, 외부 모두 접근 가능!)

    ```
    network.host: ["_local_", "_site_"]
    ```

  - `_global_` 변수값이면 로컬 네트워크 주소가 되는데, VM Instance에서 External IP가 된다, 직접 값을 넣어도 되고 그냥 변수값을 넣어도 됨 - External IP 는 GCP가 외부에 방화벽 밖에 설정해놓은 주소

  - network.host 값을 설정하고 엘라스틱 서치를 실행하면 바로 꺼짐 -> host 값을 넣지 않았을 때는 elasticsearch 가 dev 환경이었지만 여기에 값을 넣는 순간 외부환경과 연결되기 때문에 엘라스틱 서치는 여러가지 부트스트랩 체크를 하게 된다. -> 마지막 로그를 보면 왜 실행이 안되었는지 이유를 확인할 수 있음

    - max file descriptor 바꿔주기 - 리눅스 머신 설정을 바꿔줘야함 -> 운영체제는 1프로세스 당 접근할 수 있는 파일의 개수에 한계를 두고 있는데 elaticsearch 는 사용하는 파일 개수가 굉장히 많기 때문에 이 개수 설정값을 늘려줘야함  (elasticseach doc에서 reference - system config - important system configuration - increase file descriptors 설명보기 - 2가지 방법이 있음)

      - ulimit 방식 - 한 세션에만 적용되는 temporary 설정

      - etc/security/limits.conf 파일 수정 - 영구적인 설정 방법

        ```
        ls -l /etc/security/limits.conf  // -> 권한이 루트권한으로 되어있음 그러므로 수정하려면 sudo로 해야함
        sudo vi /etc/security/limits.conf // -> 마지막 # End of file 아래에 kimjmin - nofile 65535 입력 -> 이후 시스템 재시작해야함
        ```

        

    - max virtual memory areas vm.max_map_count 바꿔주기 - 리눅스 머신 설정을 바꿔줘야함 (elasticseach doc에서 reference - system config - important system configuration - virtual memory 설명보기)

      -  한 세션에만 적용되는 temporary 설정

        ```
        sysctl -w vm.max_map_count=262144
        ```

      - etc/sysctl.conf 파일 수정 - 영구적인 설정 방법

        ```
        ls -la /etc/sysctl.conf // -> 권한이 루트권한으로 되어있음 그러므로 수정하려면 sudo로 해야함
        sudo vi /etc/sysctl.conf // -> vm.max_map_count=262144 입력 -> 이후 시스템 재시작해야함
        sudo shutdown -r // -> 재시작 명령어  (bin/elasticsearch 명령어로 실행해도 됨)
        ```

    - discovery settings are unsuitable for production use

      elasticsearch.yml 에 다음 설정값 추가

      ```
      discovery.seed_hosts: ["elastic-1"]  // terminal에 hostname 입력하면 나오는 이름 (ip 주소를 넣어도 됨)
      cluster.initial_master_nodes: ["node-1"]
      ```

    

    - 위 설정을 마치면 더이상 curl localhost:9200 으로 접근 불가능 (즉, localhost로 접근 불가능) curl 10.178.0.2:9200 이렇게 해야 접근 가능

      

    - External IP로도 접근가능하게 하려면 instance에 firewall 설정을 추가해줘야한다.

      - GCP VM Instance에서 오른쪽 connect 바로 옆 세로 ... 을 누르고 View network details 누름
      - 왼쪽 firewall 메뉴 선택
      - create a firewall rule 클릭
      - Name 입력 (ex. elastic) 
      - Target tags 입력 (ex. elastic) -> 추후 VM에 추가되는 태그 이름이 된다
      - Source IP ranges 는 0.0.0.0/0 입력 (filter IP임 -> 0.0.0.0/0 은 외부 어디서나 접근이 가능하다는 의미 만약 이걸 내 로컬에서만 접속하고 싶다하면 내 로컬 주소를 입력하면 됨)
      - Specified protocols and ports 에서 tcp port 로 입력 (ex. 9200)
      - create!
      - VM Instance (in Computer Engine) 에서 해당 인스턴스 누르고 상단 EDIT 버튼 누르고 Network tags 에 elastic 태그 추가 (위에서 생성한 태그) 후 저장



#### Elasticsearch Cluster 만들기

- 앞의 과정을 반복 또는 image로 복사하여 instance를 총 3가지를 만든다.

- node name 각각 node-2, node-3, seed host를 elastic-2, elastic-3 로,  master nodes 를 node-2, node-3로 변경

- cluster에 포함될 instance 들의 Host 주소를 관리할 별도의 파일을 만든다(ip를 설정파일에 직접 넣는 것 보다 이렇게 host 파일에 지정해주는 것이 좋음) -> 아래 설정을 마치면 이렇게 접근 가능  curl elastic-2:9200 -> instance 모두에 설정해두자! -> 이렇게 해야 elasticsearch.yml 에서 discovery.seed_hosts을 : [elasitc-1, elastic-2, elastic-3] 이렇게 쓸 수 있음 ip 주소 대신

  ``` 
  sudo vi /etc/hosts
  
  --insert--
  10.178.0.2	elastic-1
  10.178.0.3	elastic-2
  10.178.0.4	elastic-3
  ```



- elastic-2와 elastic-3를 elastic-1에서 접근 가능하도록 Firewall 설정 (elastic-1은 위에서 외부 접근 가능하도록 firewall 만듬)

  - Target tags: elastic-internal
  - Source filter: Source tags
  - Source tags: elastic-internal
  - Specified protocols and ports 에서 tcp port: 9200, 9300
  - 3가지 instance (elastic-1, elastic-2, elastic-3) 에 모두 해당 태그 추가 (elastic-2와 elastic-3는 위에 elastic태그 빼줘야 함)

- 3가지 instance (elastic-1, elastic-2, elastic-3)에 elasticsearch.yml 설정 추가

  ```
  discovery.seed_hosts: ["elastic-1", "elastic-2", "elastic-3"] // /etc/hosts에서 설정한 값
  cluster.initial_master_nodes: ["node-1", "node-2", "node-3"]
  ```

  

- 그리고 나서 3개의 instance를 실행할 때 data 폴더를 한번 rm -rf 로 지워주고 bin/elasticsearch 로 실행하자 왜냐하면 기존에 별개로 실행했던 이력이 있으면 cluster 로 안묶일 수 있기 때문



- 다른 터미널에서 cluster에 속한 노드들 확인

  ```
  curl -X GET "localhost:9200/_cat/nodes" // 클러스터에 속한 노드들 확인
  curl "http://34.64.137.113:9200/_cat/nodes?v" // title도 같이 확인 가능
  ```


#### 보안

- elasticsearch.yml 파일에 보안설정 추가

  ```
  xpack.security.enabled: true // id&pw 적용 설정
  xpack.security.transport.ssl.enabled: true   //transport layer security 적용 설정 (production 환경)
  
  + 공개키, 대칭키 입력하기 -> 이 전에 이 키를 만들어두어야 함 -> 키를 만들려면 아래 과정을 거쳐서 certification 만들어야한다.
  ```

  

- elasticsearch 에서는 certificate를 만들 수 있는 [cert util](https://www.elastic.co/guide/en/elasticsearch/reference/7.11/encrypting-communications-certificates.html) 제공

  ```
  ./bin/elasticsearch-certutil ca
  ```

  - outfile명 입력 (default는 elastic-stack-ca.p12)
  - pw 입력 -> 공개키 암호가 된다.

  - es-714폴더 아래 elastic-stack-ca.p12 라는 이름의 certificate이 생성되어 있는 것을 확인



- 다음 명령어 입력 (리눅스에서 \는 줄바꿈의 의미를 가진다)

  ```
  ./bin/elasticsearch-certutil cert \  
  --ca elastic-stack-ca.p12 \ // 생성된 cert 이름
  --dns elastic-1,elastic-2,elastic3 \ 
  --ip 10.190.0.2,10.190.0.3,10.190.0.4 \
  --out config/certs/es-cluster.p12 \     // 지정할 파일 이름
  ```

  - 이렇게 하면 위에서 만든 pw 입력하라고 함

  - 그 다음 es-cluster.p12 에서 사용할 pw 입력

  - config/certs에 es-cluster.p12 생성되어있는 것 확인

    

- elasticsearch.yml 파일에 보안설정 추가

  ```
  xpack.security.transport.ssl.keystore.path: certs/es-cluster.p12   //절대 경로 설정을 안하면 자동으로 config/ 를 찾음
  xpack.security.transport.ssl.truststore.path: certs/es-cluster.p12
  ```

  - 이 다음에 원래는 다음이 추가되어야 하나 직접 pw 설정하면 보안에 문제가 된다

    ```
    xpack.security.transport.ssl.keystore.secure_password
    xpack.security.transport.ssl.truststore.secure_password
    ```

  - 그래서 다음 명령어로 별도로 입력

    ```
    ./bin/elasticsearch-keystore create 
    ./bin/elasticsearch-keystore add xpack.security.transport.ssl.keystore.secure_password
    ./bin/elasticsearch-keystore add xpack.security.transport.ssl.truststore.secure_password
    ```

    

-> 이후 해당 인증서를 복사해서 elastic-2, elastic-3에도 저장한다. 근데 직접 복사가 안되서 내 컴퓨터에 한번 저장 후에 그걸로 저장해야함. 이렇게 하면 이제 노드끼리 보안된 통신이 가능하게 된다.

```
scp -i ~/.ssh/es-rsa kimjmin@34.64.137.113:/home/kimjmin/es-714/config/certs/es-cluster.p12 ./

다음 elastic-2와 elastic-3 에 certs 디렉토리 만들어 줌
scp -i ~/.ssh/es-rsa ./es-cluster.p12 kimjmin@34.64.193.32:/home/kimjmin/es-714/config/certs/es-cluster.p12
scp -i ~/.ssh/es-rsa ./es-cluster.p12 kimjmin@34.64.167.194:/home/kimjmin/es-714/config/certs/es-cluster.p12
```

-> 이후 elastic-2와 elastic-3에도 동일하게 security 관련 설정을 elasticsearch.yml에 해준다.

-> 이후 elasticsearch 에서 제공하는 실행파일로 user id & pw 를 설정하면 curl과 같은 Rest API 사용시에 user의 id & pw 도 같이 넣어서 요청해야 정상 응답하게 된다. (만약 패스워드 잊어버렸을 때는 data 폴더를 지우면 되긴 하는데 그러면 데이터들 다 날아가서 일반적으로 이렇게 하면 안된다) 이렇게 만든 id & password 는 cluster 전체에 적용된다 (Native Realm)

```
./bin/elasticsearch-setup-passwords interactive
```

-> 아래는 super user 로 유저별로 생성해주는 방법인데 이걸로 만든 경우에는 팀에 공유하지 않고 개인만 알고 있는 것이 좋다. 이 방법은 file realm 에 적용되어서 이렇게 등록한 node에만 이 id&password로 접속 가능하다. (File Realm)

```
bin/elasticsearch-users user add kimjmin -p password -r superuser
```


#### 데몬 실행(백그라운드 실행)

```
start.sh, stop.sh 파일 생성 및 실행 권한 부여
$ echo 'bin/elasticsearch -d -p es.pid' > start.sh
​
$ echo 'kill `cat es.pid`' > stop.sh
​
$ chmod 755 start.sh stop.sh
​
$ ./start.sh
​
$ ./stop.sh
```

