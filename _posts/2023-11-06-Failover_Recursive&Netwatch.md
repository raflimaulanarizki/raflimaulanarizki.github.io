---
layout: post
title: Failover Recursive & Netwatch - Mikrotik
date: 2023-11-06
pin: 
tags: mikrotik network
splash_img_source: /assets/img/2023-11-06-Failover_Recursive&Netwatch/topo.png
splash_img_caption:
---

Failover adalah fitur yang digunakan untuk menjaga konektivitas network tetap stabil, jika terjadinya gangguan/down pada link Primary maka Failover ini akan memindahkan Traffic ke link Backup.

Masalah Failover

⇒ Failover terjadi dikarenakan router **tidak dapat ping** ke **default gateway** bukan ping ke internet. **lalu** **masalahnya** pada saat ping ke internet (8.8.8.8) bermasalah dan ip default gateway tetap terkoneksi atau bisa di ping maka **Failover tidak akan berkerja.**

Ada dua cara yang dapat mengatasi Masalah tersebut, berikut saya jelaskan

## 1. Failover Recursive

Failover ini sama dengan Failover biasa menggunakan “Distance” dan “Check gateway”. bedanya di ip **gatewaynya**. 

Failover Recursive memiliki Interval yang lumayan lama sekitar 10 - 20 sequence (5-10 detik)

Sebelum membuat failover, create terlebih dahulu route untuk pengecekan internet. route ini di isikan dst network to ip public (8.8.8.8/1.1.1.1) dengan gateway networknya.

Setting network dari topologi diatas.

### Configuration

1. Setting Basic Configuration
2. Membuat Route untuk cek Internet
    
    ```bash
    /ip route
    add check-gateway=ping distance=1 dst-address=1.1.1.1/32 gateway=192.168.125.1
    add check-gateway=ping distance=1 dst-address=8.8.8.8/32 gateway=192.168.1.1
    ```
    
3. Membuat Default route to internet
    
    ```bash
    /ip route
    add check-gateway=ping distance=1 gateway=1.1.1.1 dst-address=0.0.0.0/0 target-scope=30
    add check-gateway=ping distance=2 gateway=8.8.8.8 dst-address=0.0.0.0/0 target-scope=30
    ```
    
    - Note :
        - Pastikan **“target-scope”** sama atau lebih dari yang ditentukan.
        jika di bawah maka gateway akan unreachable
            
            ![recursive.png](/assets/img/2023-11-06-Failover_Recursive&Netwatch/recursive.png)
            
        - Gateway diisikan dengan ip route yang sudah di buat sebelumnya.
    - Cara kerja :
        - Route cek internet akan mengecek dst.address misal 8.8.8.8 dengan ping “check-gateway” terus menerus.
        - pada saat pengecekan terjadi down maka otomatis Rule 0.0.0.0/0 gateway 8.8.8.8 akan nonaktif dan berpindah ke backup route.

## 2. Failover Netwatch

Failover Netwatch pada dasarnya Sama dengan **Failover biasa dan Recursive**, perbedaan di **intervalnya** saja atau **waktu perubahan** **routenya** jauh lebih cepat di bandingkan Recursive dan bisa di custom.

Tools Netwatch ini akan memonitoring hosts dengan interval dan timeout yang sudah ditentukan. jika hosts tersebut down maka status dari netwatch akan down.

Netwatch ini kita akan memasukan script jadi :

- Jika Netwatch down yakni host yang di monitoing, maka kita akan disable route Utama
- Jika Netwatch Up maka enabla ekmbali route utamanya

Setting network dari topologi diatas.

### Configuration

1. Setting Basic Configuration
2. Membuat Default route to internet
    
    ```bash
    /ip route
    add check-gateway=ping comment=Main distance=1 gateway=192.168.125.1 dst-address=0.0.0.0/0 
    add check-gateway=ping comment=Backup distance=2 gateway=192.168.1.1 dst-address=0.0.0.0/0
    ```
    
    ![routenetwatch.png](/assets/img/2023-11-06-Failover_Recursive&Netwatch/routenetwatch.png)
    
3. Set Configure Netwatch
    
    <img src="/assets/img/2023-11-06-Failover_Recursive&Netwatch/netwatch1.png" alt="netwatch1" style="width: 40%;">
    Hosts : IP Internet yang di monitoring <br>
    Interval : Berapa lama untuk menentukan down atau tidaknya, set ke 5 sektiar (3-5 detik) <br>
    Timeout : Timeout Hosts. <br>

    ![netwatch2.png](/assets/img/2023-11-06-Failover_Recursive&Netwatch/netwatch2.png)
    ⇒ Set disable route dengan comment “Utama” 
    
    Otomatis Route backup akan menerusakan traffic internet jika status DOWN.
    
    ![netwatch3.png](/assets/img/2023-11-06-Failover_Recursive&Netwatch/netwatch3.png)
    ⇒ Set enable route dengan comment “Utama” 
    
    Route Utama akan kembali menerusakan traffic internet jika status UP.
    

## Thank You