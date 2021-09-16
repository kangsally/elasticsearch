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

