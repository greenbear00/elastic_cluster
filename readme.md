## elk stack

참고: https://jistol.github.io/docker/2019/03/27/docker-compose-elasticsearch-cluster/)
- cluster로 구축
```
$ docker-compose -f docker-compose.yml up
$ docker-compose -f docker-compose.yml down
```
    
- 접속 확인
```
- elastic:
    $ curl http://localhost:9200
    $ curl -XGET "http://es01:9200/crimes2-*/_search?size=1&pretty=true" (cluster 구축 확인)
        ip         heap.percent ram.percent cpu load_1m load_5m load_15m node.role master name
        172.18.0.3           63          97   9    0.20    0.83     1.06 dilmrt    -      es02
        172.18.0.4           59          97   9    0.20    0.83     1.06 dilmrt    -      es03
        172.18.0.2           40          97   9    0.20    0.83     1.06 dilmrt    *      es01
- kibana: http://localhost:5601
- logstash: http://localhost:9600/
```
    
## logstash test
- config에 logstash.yml, pipelines.yml 작성
- pipeline에 실제 pipelines와 연동해서 pipeline 코드 작성
- 필요한 데이터는 data 폴더를 바라보고 작성
- 참고: https://koocci-dev.tistory.com/20, http://jason-heo.github.io/elasticsearch/2016/02/28/logstash-offset.html
- 참고: https://www.elastic.co/guide/en/logstash/current/index.html

1. csv 파일 로딩(예제: csv_test.conf) 
참고: csv filter: https://www.elastic.co/guide/en/logstash/current/plugins-filters-csv.html

```
    - input{ file{ path=> ..., start_position=>..., sincedb_path=>....} }
        - start_position은 파일을 읽을때 최초 동작되고 beginning과 end만 존재. stream처리를 하려면 end가 필요하고, 그렇지 않으면 beginning
        - sincedb_path는 logstash가 파일 읽을시 offset 처리를 위해 동작하는데 offset(/var/logs/syslog로 설정할경우, 여기에 offset이 없으면 start_position을 바라보고 동작, 단 /dev/null로 되면 offset을 저장하지 않기에 무조건 처음부터 동작하게 됨)
        - sincedb_write_interval은 offset 저장주기를 관리
    "hits" : [
    {
        "_index" : "logstash-csv-2021.01.22",   # 실제 index
        "_type" : "_doc",                       # 7.0부터는 type 개념이 없어짐
        "_id" : "hDixJ3cBLM_lLGR_5xtq",         # unique한 ID
        "_score" : 1.0,                         # doc에 대한 점수
        "_source" : {                           # 실제 _id에 대한 documents
        "host" : "5a7ca3c63ce5",                # ...
        "manufacturer" : "computer associates",
        "id" : "b0006zf55o",
        "@timestamp" : "2021-01-22T01:25:05.289Z",  # 데이터에 대한 시간 값
        "description" : "oem arcserve backup v11.1 win 30u for laptops and desktops",
        "title" : "ca international - arcserve lap/desktop oem 30pk",
        "path" : "/usr/share/logstash/tmpdata/products1.csv",   # ...
        "message" : """b0006zf55o,ca international - arcserve lap/desktop oem 30pk,oem arcserve backup v11.1 win 30u for laptops and desktops,computer associates,0 
""",
        "price" : "0 ",
        "@version" : "1"    # ...
        }
    }, .... ]
```

## elastic data 확인
- dsl 문법: https://esbook.kimjmin.net/05-search
