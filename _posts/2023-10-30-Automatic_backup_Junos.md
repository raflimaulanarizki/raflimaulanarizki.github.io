---
layout: post
title: Manually and Automatically Backup - Juniper
date: 2023-10-30
pin: 
tags: juniper network
splash_img_source: /assets/img/Juniper-Automatic_Backup.jpeg
splash_img_caption: 
---

Konfigrasi Backup Otomatis dan Manual menggunakan FTP server, 

## Set Clock
Pastikan Waktu sudah sesuai, karena format file menggunakan Tanggal dan waktu.

```bash
#show time 
user@host> show system uptime 
user@host# set system time-zone Asia/Jakarta #Set Timezone*

#jika belum sesuai, bisa di set manual.
user@host> set date 202310131309* 
```
# 1. Manual Backup
```bash
#Backup Configuration Juniper
user@host# save SW.HOME_13Okt23
user@host# commit

#Cek File
user@host> file list

#Upload File to FTP Server
user@host> file copy SW.JUNOS.13okt ftp://admin@172.16.10.10/switch/
```

# 2. Automatic Backup

## Event-Options
Create Scheduler (Action), Action (Policy) and Destination.

##### <b>Generate Event</b> (Schedule) [link](https://www.juniper.net/documentation/us/en/software/junos/automation-scripting/topics/ref/statement/generate-event-edit-event-options.html)

```sh
user@host# set event-options generate-event Backup-Daily_event time-of-day "00:00:05 +0700"
```
- Backup-Daily_event : Nama Event
- time-of-day        : Daily event / setiap hari
- "00:00:05 +0700"   : Waktu executed event

##### <b>Policy</b>
Action yang akan dilakukan jika event sudah sesuai dengan yang di jadwalkan
```sh
user@host# set event-options policy Backup-Daily_police events Backup-Daily_event
#Upload/backup config juniper to destination
user@host# set event-options policy Backup-Daily_police then upload filename /config/juniper.conf.gz destination ftp_server
```
- Backup-Daily_police : Nama Policy
- ftp_server          : Nama Destination, yang akan di create

##### <b>Destination</b>
Note : Harus sudah memiliki FTP Server, untuk user, ip dan path nya bisa di sesuaikan
```bash
user@host# set event-options destinations ftp_server archive-sites ftp://admin@172.16.10.10/switch/ password test112233
#or
user@host# set event-options destinations ftp_server archive-sites ftp://admin:test112233@172.16.10.10/switch/
```
- ftp_server : Nama Destination

Note: Commit Configuration

## Configuration Results
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
                "ftp://admin@172.16.10.10/switch/" password "$9$SDFASDdfsdfsdfSDsfsdFSDd"; ## SECRET-DATA
            }
        }
    }
}
```

Reference
- [supportportal.juniper.net_Junos-How-to-take-configuration-backup-automatically](https://supportportal.juniper.net/s/article/Junos-How-to-take-configuration-backup-automatically?language=en_US)