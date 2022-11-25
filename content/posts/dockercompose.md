---
title: "docker-compose.yml範例"
date: 2022-10-24T15:37:25+08:00
draft: false
tags: ["docker"]
type: post
author: "Steve Lin"
description: ""
---
# docker-compose.yml
```
version: "3.2"

services:
  elasticsearch:
    image: elasticsearch:7.10.1
    container_name: elasticsearch-local
    ports:
      - "9200:9200"
      - "9300:9300"
    environment:
      - cluster.name=docker-cluster
      - bootstrap.memory_lock=true
      - http.host=0.0.0.0
      - http.port=9200
      - transport.host=127.0.0.1
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
      - "http.cors.allow-origin=http://127.0.0.1:1358"
      - "http.cors.enabled=true"
      - "http.cors.allow-headers=X-Requested-With,X-Auth-Token,Content-Type,Content-Length,Authorization"
      - "http.cors.allow-credentials=true"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - es_data:/usr/share/elasticsearch/data
    networks:
      - esnet

  kibana:
    image: kibana:7.10.1
    container_name: kibana-local
    environment:
      SERVER_NAME: kibana-server
      ELASTICSEARCH_URL: http://elasticsearch:9200
    networks:
      - esnet
    depends_on:
      - elasticsearch
    ports:
      - "5601:5601"

  cerebro:
    image: yannart/cerebro:latest
    container_name: cerebro-073
    networks:
      - esnet
    ports:
      - "9900:9000"
    depends_on:
      - elasticsearch      
 
 
      
  dejavu:
    image: appbaseio/dejavu:latest
    container_name: dejavu
    networks:
      - esnet    
    ports:
     - "1358:1358"

volumes:
  es_data:
    driver: local      

networks:
  esnet:
    driver: bridge
```