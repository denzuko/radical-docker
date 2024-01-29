.PHONY: all services config clean

define COMPOSEYAML
---
version: '3'

services:
  bridge:
    stdin_open: true
    ports:
      - 6667:6667/tcp
    volumes:
      - ./data/config:/var/lib/bitlbee
    networks:
      default:
        aliases:
          - bridge.local
          - bitlbee.local
    deploy:
      restart_policy:
        condition: always
    image: ezkrg/bitlbee-libpurple:latest
  bouncer:
    stdin_open: true
    ports:
      - 6697:6697
      - 8443:6697
    volumes:
      - ./data/spool/znc-data:/znc-data
    networks:
      default:
        aliases:
          - bouncer.local
    deploy:
      restart_policy:
        condition: always
    links:
      - bridge
    depends_on:
      bridge:
        condition: service_started
    image: znc:slim
  bot:
    stdin_open: true
    deploy:
      restart_policy:
        condition: always
    environment:
      SERVER: "bouncer.local +6697 secret"
      NICK: "sentinal"
      CHANNELS: '#test'
    volumes:
      - ./data/spool/eggdrop/data:/home/eggdrop/eggdrop/data
      - ./data/spool/eggdrop/scripts:/home/eggdrop/eggdrop/scripts2
    ports:
      - 3333:3333/tcp
    links:
      - bridge
      - bouncer
    depends_on:
      bridge:
        condition: service_started
      bouncer:
        condition: service_started
    image: eggdrop:latest
...
endef
export COMPOSEYAML

all: config services

docker-compose.yml:
        @- echo "$$COMPOSEYAML" > $@

data/spool/znc-data: config

services: docker-compose.yml data/spool/znc-data
        @- docker compose up -d

config:
        @-  docker run -it -v ./data/spool/znc-data:/znc-data znc --makeconf

clean: docker-compose.yml
        @- docker compose down
