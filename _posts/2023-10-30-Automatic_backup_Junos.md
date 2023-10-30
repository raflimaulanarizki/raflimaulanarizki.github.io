---
layout: post
title: Backup Automatically Junos OS
date: 2023-10-30
pin: 
tags: juniper network
splash_img_source: /assets/img/mikrotik-openvpn.png
splash_img_caption: 
---

Konfigrasi Backup Automatic menggunakan FTP server, 
sebelumnya saya sudah membuat konfigurasi Backup manual berikut linknya 

## Set Clock
Pastikan Waktu sudah sesuai, karena format file menggunakan Tanggal dan waktu.

```bash
#show time 
user@host> show system uptime 
user@host# set system time-zone Asia/Jakarta #Set Timezone*

#jika belum sesuai, bisa di set manual.
user@host> set date 202310131309* 
```

## Backup Manual

```bash
save SW.HOME_13Okt23
file list
file copy SW.JUNOS.13okt ftp://admin@172.16.10.10/switch/
```

## Set Event-Options

Membuat Scheduler, Action and Destination.

### Generate Event

Create Schedule (Event), [test](https://www.juniper.net/documentation/us/en/software/junos/automation-scripting/topics/ref/statement/generate-event-edit-event-options.html)

```bash
user@host# set event-options generate-event Backup-Daily_event time-of-day "00:00:05 +0700"
#Command
Backup-Daily_event : Nama Event
time-of-day        : Daily event / setiap hari
"00:00:05 +0700"   : Waktu executed event
```

set policy Backup-Daily_police events Backup-Daily_event
set policy Backup-Daily_police then upload filename /config/juniper.conf.gz destination ftp_server

set destinations ftp_server archive-sites [ftp://admin@172.16.10.10/switch/](ftp://admin@172.16.10.10/switch/) password test112233

set generate-event Backup-Daily_event time-of-day "13:05:00 +0700"

delete destinations ftp_server archive-sites
set destinations ftp_server archive-sites [ftp://admin@172.16.10.10/switch/](ftp://admin@172.16.10.10/switch/) password test112233

set destinations ftp_server archive-sites [ftp://admin:abc123@172.16.10.10/switch/](ftp://admin:test112233@172.16.10.10/switch/)

```bash
event-options {
    generate-event {
        Backup-Daily_event time-of-day "00:00:10 +0700";
    }
    policy Backup-Daily_police {
        events Backup-Daily_event;
        then {
            upload filename /config/juniper.conf.gz destination ftp_server;
        }
    }
    destinations {
        ftp_server {
            archive-sites {
                "ftp://admin@172.16.10.10/switch/" password "$9$DDkfz9A0Ihr0BeWLXbwHqmP5Fn6ABIcAtEyreW82go"; ## SECRET-DATA
            }
        }
    }
}
```