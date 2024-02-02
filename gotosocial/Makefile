#!/bin/env make

## Defaults
TARGET	      	  := gotosocial
FQDN              := example.com
GPG_PASSPHRASE	  := WinampWhipsTheLlamasAss
LE_EMAIL          := postfix@example.com
LE_ENABLED        := false
GTS_SMTP_HOST     := mx.example.com
GTS_SMTP_PORT     := 587
GTS_SMTP_USERNAME := $(LE_EMAIL)
TZ                := UTC

## TEMPLATES
define MANPAGE
.TH $(TARGET) 7
.SH NAME
$(TARGET) \- Basic Deployment of a standalone gotosocial instance.
.SH SERVICES
.TP
.BR \fBgotosocial\fR\ \- \ \fIfediverse\ headless\ instance\fR
.TP
.BR \fBbackup\fR\ \- \ \fIbackup\ service\fR
.SH DESCRIPTION
.B $(TARGET)
This is a clone of the instance setup at social.2600hackers.com which runs a headless (api first) fediverse
instance. Gotosocial works great for Semaphore, tusky, or web/rss via https://example.com/@username/feed.rss.
Included in this setup is a backup service that executes every Sunday and prunes +15 day backups.
.SH GOALS
Create a release pipeline that uses Make, man, and docker-compose to deploy gotosocial.
.SH CONFIG
When deploying use \fImake(1)\fR and append with the following environment varables as needed.
.SH ENVIRONMENT VARIABLES
.TP
.BR TZ =\fIRFC6557\fR
The fully qualified domain name for the instance. [default: $(FQDN)]
.TP
.BR FQDN =\fIRFC1035\fR
The fully qualified domain name for the instance. [default: $(FQDN)]
.TP
.BR GPG_PASSPHRASE =\fIstring\fR
Sets a GPG passphrase for encrypted backups. [default: $(GPG_PASSPHRASE)]
.TP
.BR LE_ENABLED =\fIbool\fR
Turns on letsencrypt certificates. [default: $(LE_ENABLED)]
.TP
.BR LE_EMAIL =\fIRFC5322\fR
Sets the email used for letsencrypt. [default: $(LE_EMAIL)]
.TP
.BR GTS_SMTP_HOST =\fIRFC1035\fR
Sets the outbound email server for invites, notices, and administration. [default: $(GTS_SMTP_HOST)]

.SH PORTS
.TP
.BR 80 ", " 443 =\fITCP\fR
Gotosocial http ports.
.SH SOURCES
https://github.com/denzuko/radical-docker/view/master/gotosocial/Makefile
https://docs.gotosocial.org/en/latest/getting_started/installation/container/
https://offen.github.io/docker-volume-backup/how-tos/
.SH AUTHORS
\fBDwight Spencer\fR \fI@denzuko@mastodon.social\fR

endef
export MANPAGE

define COMPOSEYAML
---
version: '3.9'

volumes:
  gts_storage: {}

services:
  gotosocial: 
    image: superseriousbusiness/gotosocial:latest 
    user: 1000:1000
    environment:
      GTS_HOST: "$(FQDN)"
      GTS_DB_TYPE: sqlite 
      GTS_DB_ADDRESS: /gotosocial/storage/sqlite.db
      GTS_LETSENCRYPT_ENABLED: "$(LE_ENABLED)"
      GTS_LETSENCRYPT_EMAIL_ADDRESS: "$(LE_EMAIL)"
      GTS_ACCOUNTS_APPROVAL_REQUIRED: "false"
      GTS_SMTP_HOST: "$(GTS_SMTP_HOST)"
      GTS_SMTP_PORT: "$(GTS_SMTP_PORT)"
      GTS_SMTP_USERNAME: "$(GTS_SMTP_USERNAME)"
      GTS_SMTP_PASSWORD: "$(GTS_SMTP_PASSWORD)"
      GTS_SMTP_FROM: "$(GTS_SMTP_FROM)"
      TZ: $(TZ)
      GTS_OIDC_ENABLED: "False"

    ports:
      - "443:8080/tcp"
      ## For letsencrypt:
      - "80:80/tcp"
    volumes:
      - gts_storage:/gotosocial/storage

    deploy:
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
      resources:
        limits:
          memory: 2G

  backup:
    deploy:
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
      resources:
        limits:
          memory: 25M
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - ./backups:/archive
      - gts_storage:/backup/cms_content-snapshot:ro
    environment:
      BACKUP_LATEST_SYMLINK: $(TARGET)-latest.tar.gz
      BACKUP_FILENAME: $(TARGET)-%s.tgz
      BACKUP_PRUNING_PREFIX: $(TARGET)-
      BACKUP_COMPRESSION: "gz"
      BACKUP_RETENTION_DAYS: '14'
      BACKUP_CRON_EXPRESSION: "0 4 * * SUN"
      GPG_PASSPHRASE: $(GPG_PASSPHRASE)
    image: offen/docker-volume-backup:latest

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

config:
  @- echo

clean: docker-compose.yml
	@- docker compose down

distclean: clean
	@- rm $(TARGET).man docker-compose.yml
