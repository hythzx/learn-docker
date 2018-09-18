```
version: '3'
services:
    jhipster-elasticsearch:
        image: jhipster/jhipster-elasticsearch:latest
        networks:
            - ms_net
        volumes:
            - log-data:/usr/share/elasticsearch/data
    jhipster-logstash:
        image: jhipster/jhipster-logstash:latest
        networks:
            - ms_net
        environment:
            - INPUT_TCP_PORT=5000
            - INPUT_UDP_PORT=5000
            - INPUT_HTTP_PORT=5001
            - ELASTICSEARCH_HOST=jhipster-elasticsearch
            - ELASTICSEARCH_PORT=9200
            - LOGSTASH_DEBUG=false
        ports:
            - 5000:5000
            - 5000:5000/udp
            - 5001:5001
        volumes:
            - log-conf:/usr/share/logstash/pipeline/

    jhipster-console:
        image: jhipster/jhipster-console:latest
        networks:
            - ms_net
        ports:
            - 5601:5601
        environment:
           - ELASTICSEARCH_URL=http://jhipster-elasticsearch:9200

    jhipster-import-dashboards:
        image: jhipster/jhipster-import-dashboards:latest
        environment:
            - ELASTICSEARCH_URL=http://jhipster-elasticsearch:9200
        networks:
            - ms_net

    jhipster-curator:
        image: jhipster/jhipster-curator:latest
        networks:
            - ms_net
        environment:
            - ES_HOST=jhipster-elasticsearch
            - ES_PORT=9200
            - UNIT_COUNT=14
            - UNIT=days

    jhipster-alerter:
        image: jhipster/jhipster-alerter
        networks:
            - ms_net
        environment:
            - ES_HOST=jhipster-elasticsearch
            - ES_PORT=9200
        volumes:
            - elastalert:/opt/elastalert/

    jhipster-zipkin:
        image: jhipster/jhipster-zipkin:latest
        networks:
            - ms_net
        environment:
            - ES_HOSTS=http://jhipster-elasticsearch:9200
            # Change localhost:5601 by the URL by which your client access Kibana
            - "ZIPKIN_UI_LOGS_URL=http://localhost:5601/app/kibana#/discover/d0682f20-e0e9-11e7-9c68-0b9a0f0c183c?_g=(refreshInterval:(display:Off,pause:!f,value:0),time:(from:now-30d,mode:quick,to:now))&_a=(columns:!(X-B3-TraceId,app_name,level,message),filters:!(('$state':(store:appState),meta:(alias:!n,disabled:!f,index:c7b73f10-e0e4-11e7-9c68-0b9a0f0c183c,key:logger_name,negate:!t,params:(query:metrics,type:phrase),type:phrase,value:metrics),query:(match:(logger_name:(query:metrics,type:phrase)))),('$state':(store:appState),meta:(alias:!n,disabled:!f,index:'logstash-*',key:X-B3-TraceId,negate:!f,params:(query:{traceId},type:phrase),type:phrase,value:{traceId}),query:(match:(X-B3-TraceId:(query:{traceId},type:phrase))))),index:'logstash-*',interval:auto,query:(language:lucene,query:''),sort:!('@timestamp',asc))"
        ports:
            - 9411:9411
volumes:
    log-data:
        driver: local
    log-config:
        driver: local
    elastalert: 
        driver: local
    log-conf:
        driver: local
    rabbitmq_data:
        driver: local
    mysql-conf:
        driver: local
    mysql-data:
        driver: local
    redis_conf:
        driver: local
    mongo_data:
        driver: local
    mongo_config:
        driver: local
networks:
  ms_net:
    driver: overlay
```