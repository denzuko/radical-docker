#!/bin/env make

## Defaults
TARGET          := bbs

## TEMPLATES
define MANPAGE
.TH $(TARGET) 7
.SH NAME
$(TARGET) \- Experiments in launching a Modern-ish RetroBBS in docker.
.SH SERVICES
.TP
.BR
\fB$(TARGET)\fR \- \fIRetroBBS with ssh, http, and fidonet support.\fR
.SH DESCRIPTION
.B $(TARGET)
Enigma based RetroBBS with telnet support in docker.
.SH GOALS
Deploy a distributed fidonet enabled BBS groupware site with modern BBS extentions/codebase.
which can interface with external chat channels.
.SH ENVIRONMENT VARIABLES
\fIN/A\fR
.SH PORTS
.TP
.BR 7080 =\fITCP\fR
ENiGMA½ comes with a built in content server for supporting both HTTP and HTTPS
.TP
.BR 7022=\fITCP\fR
ENiGMA½ SSH login server, first login creates a user added to sysops group
.SH SOURCES
https://github.com/denzuko/radical-docker/view/master/bbs/Makefile
.SH AUTHORS
\fBDwight Spencer\fR \fI@denzuko@mastodon.social\fR

endef
export MANPAGE

define COMPOSEYAML
---
version: "3.7"

## DSL Config
x-influxlog: &syslog
  logging:
    driver: syslog
    options:
      # syslog-address: "udp://{{.Node.Hostname}}:6154"
      # syslog-address: "udp://{{.Swarm.NodeAddr}}:6154"
      syslog-address: "udp://127.0.0.1:6154"
      syslog-format: rfc5424micro
      tag: "{{.ImageName}}/{{.Name}}/{{.ID}}"

x-replicas: &replicated
  replicas: 1

x-global: &global
  mode: global

x-worker: &worker
  placement:
    constraints: [node.role == worker]

x-manager: &manager
  placement:
    constraints: [node.role == manager]

x-restart: &on-failure
  restart_policy:
    condition: on-failure
    max_attempts: 3
    delay: 5s

  rollback_config:
    parallelism: 2
    delay: 10s
    monitor: 10s
    order: start-first
    failure_action: continue

  update_config:
    parallelism: 2
    delay: 10s
    monitor: 10s
    order: start-first
    failure_action: rollback

x-stop-grace: &stopperiod
  stop_grace_period: 30s

x-default-healthcheck: &heathcheck-pass
  test: ["CMD", "exit 0"]
  interval: 25s
  timeout: 5s
  retries: 3

x-traefik-off: &traefik-off
  traefik.enabled: "false"

x-traefik-on: &traefik-on
  traefik.enabled: "true"
  traefik.frontend.errors.network.backend: "error"
  traefik.frontend.errors.network.query: "/"
  traefik.frontend.errors.network.status: "404,500-511"
  traefik.frontned.entrypoints: 'http,https,ws,wss'
  traefik.frontend.priority: "5"
  traefik.port: "80"

x-default-route: &default-route
  <<: *traefik-on
  traefik.frontend.rule: "HostRegexp:{catchall:.*}"
  traefik.frontend.priority: "1"

x-vhost-route: &vhost-route
  <<: *traefik-on
  traefik.frontend.priority: "10"

x-orbiter-disable: &orbiter-off
  orbiter: "false"
  <<: *traefik-off

x-orbiter-enable: &orbiter-on
  orbiter: "true"
  orbiter.up: "3"
  orbiter.down: "1"
  <<: *traefik-on

services:

  $(TARGET):
    deploy:
      <<: [*worker, *on-failure]
      restart_policy:
        condition: any
        delay: 72h
      labels:
        <<: *orbiter-off
    ports:
      - 7080:8888/tcp
      - 7022:22/tcp
    image: davestephens/enigma-bbs:latest
...
endef
export COMPOSEYAML

## Rules:

.PHONY: all services config clean help distclean

all: config services

help: $(TARGET).man
        @- cat $< | man /dev/stdin

$(TARGET).man:
        @- echo "$$MANPAGE" > $@

docker-compose.yml:
        @- echo "$$COMPOSEYAML" > $@

services: docker-compose.yml
        @- docker compose up -d

config: docker-compose.yml

clean: docker-compose.yml
        @- docker compose down

distclean: clean
        @- rm $(TARGET).man docker-compose.yml
