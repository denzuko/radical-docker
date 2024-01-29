TARGET		:= secbot

.PHONY: all services config clean help distclean

define MANPAGE
.TH secbot 7
.SH NAME
secbot \- Experiments in launching bitlbee, znc, and eggdrop/pyirc bots. Client side can be anything thanks to bitlbee.
.SH SERVICES
.B corrupt
[\fBbridge\fR \fIbitlbee irc with libpurple\fR]
[\fBbouncer\fR \fIznc bouncer\fR]
[\fBbot\fR \fIeggdrop service\fR]
.IR file ...
.SH DESCRIPTION
.B secbot
Experiments in launching bitlbee, znc, and eggdrop/pyirc bots. Client side can be anything thanks to bitlbee.
Further bots can be added using python-irc and attached to the default user created via bouncer.
.SH PORTS
.TP
.BR 6697 ", " 8443 =\fITCP\fR
Znc irc and webadmin ports.
.BR 6667 =\fITCP\fR
bitlbee cleartext port
.BR 3000 =\fITCP\fR
eggdrop control port
.SH SOURCES
.TP
https://github.com/denzuko/radical-docker/view/master/secbot/Makefile
.SH AUTHORS
\fBDwight Spencer\fR \fI@denzuko@mastodon.social\fR

endef
export MANPAGE

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

help: $(TARGET).man
	@- cat $< | man /dev/stdin

$(TARGET).man:
	@- echo "$$MANPAGE" > $@

docker-compose.yml:
        @- echo "$$COMPOSEYAML" > $@

data/spool/znc-data: config

services: docker-compose.yml data/spool/znc-data
        @- docker compose up -d

config:
        @-  docker run -it -v ./data/spool/znc-data:/znc-data znc --makeconf

clean: docker-compose.yml
        @- docker compose down

distclean: clean
	@- rm @(TARGET).man docker-compose.yml
