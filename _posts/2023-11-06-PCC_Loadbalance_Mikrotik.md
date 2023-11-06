---
layout: post
title: PCC Load Balance - Mikrotik
date: 2023-11-06
pin: 
tags: mikrotik network
splash_img_source: /assets/img/pcc-mikrotik/topo1.png
splash_img_caption: 
---

Load Balance adalah metode untuk membagi traffic ke 2 jalur atau lebih. pembagian traffic ini dilakukan secara balance agar traffic berjalan optimal dan bandwidth dari ISP bisa dimaksimalkan.

Load Balance ini juga dapat **menghindari Overload** pada salah satu jalur dan dapat menjadi **backup** jika ada Jalur down/bermasalah.

## Per Connection Classifier (PCC)

PCC Bekerja dengan memecah traffic data yang melewati router kedalam beberapa stream, stream tersebut terdiri dari jalur yang berbeda.

### Soal PCC

Sebuah perusahaan dengan topologi di atas memiliki internet dari 2 ISP dengan **bandwidth yang sama**, Perusahaan ini ingin kedua ISP tersebut dipakai sebagai Load Balance dan juga sebagai Backup jika salah satu ISP bermasalah. 

Perusahaan ini juga ingin kedua Link Internet dapat digunakan secara bersamaan dan dibagi secara otomatis.

Berikan konfigurasi dari topologi di atas menggunakan Load Balance PCC dan backup Fail Over. 

### Configuration

1. Setting Basic Configuration IP Address, Vlan, Dhcp, dll.
2. Setting NAT to Out interface Eth1 dan Eth2
    
    ```bash
    /ip firewall nat
    add action=masquerade chain=srcnat comment="to;ISP-1" out-interface=ether1-ISP1
    add action=masquerade chain=srcnat comment="to;ISP-2" out-interface=ether2-ISP2
    ```
    
3. Create address list  (LAN) Network Local
    ⇒ list network local yang akan di load balance.
    
    ```bash
    /ip firewall address-list
    add address=192.168.10.0/30 list=LAN
    add address=192.168.20.0/30 list=LAN
    ```
    
4. Setting Mangle
    1. Rule (Accept) koneksi antar local network <br>
    ⇒ Berfungsi agar network yang ingin berkomunikasi antar network local **tidak melalui proses filter koneksi PCC**. 
    Jika tidak dibuatkan rule accept, maka **semua network** akan **melalui proses PCC** terlebih dahulu, itu **dapat mempengaruhi traffic** local.
        
        ```bash
        /ip firewall mangle
        add action=accept chain=prerouting dst-address-list=LAN
        ```
        
    2. Rule Traffic Input Output
    ⇒ Berfungsi untuk memastikan traffic masuk maupun keluar berada pada jalur/route yang sama.
        
        ```bash
        /ip firewall mangle
        #Traffic Masuk dari ISP
        add action=mark-connection chain=input in-interface=ether1 new-connection-mark=\
            Koneksi-ISP1 passthrough=yes
        add action=mark-connection chain=input in-interface=ether2 new-connection-mark=\
            Koneksi-ISP2 passthrough=yes
        
        #Traffic keluar
        add action=mark-routing chain=output connection-mark=Koneksi-ISP1 \
            new-routing-mark=To-ISP1 passthrough=yes
        add action=mark-routing chain=output connection-mark=Koneksi-ISP2 \
            new-routing-mark=To-ISP2 passthrough=yes
        ```
        
        Note: Jika **“in-interface”** adalah PPPoE maka select **PPPoE client Interface** tidak Port Mikrotiknya
        
    3. Rule Load Balance 2 ISP, Bandwidth sama
    ⇒ Berfungsi untuk **membagi atau menyeimbangkan** traffic, kedua ISP akan bekerja secara bersamaan dan membagi tugas. Load Balance juga meminimalisir akan terjadinya **Overload traffic** pada salah satu ISP. 
        
        ```bash
        /ip firewall mangle
        #Load Balance Traffic
        add action=mark-connection chain=prerouting connection-mark=no-mark dst-address-list=!LAN dst-address-type="" new-connection-mark=\
            Koneksi-ISP1 passthrough=yes per-connection-classifier=both-addresses-and-ports:2/0 src-address-list=LAN
        add action=mark-connection chain=prerouting connection-mark=no-mark dst-address-list=!LAN dst-address-type="" new-connection-mark=\
            Koneksi-ISP2 passthrough=yes per-connection-classifier=both-addresses-and-ports:2/1 src-address-list=LAN
        
        #Menandai Traffic dari Source LAN to dst another LAN agar diproses melalui PCC
        add action=mark-routing chain=prerouting connection-mark=Koneksi-ISP1 dst-address-list=!LAN new-routing-mark=To-ISP1 passthrough=yes \
            src-address-list=LAN
        add action=mark-routing chain=prerouting connection-mark=Koneksi-ISP2 dst-address-list=!LAN new-routing-mark=To-ISP2 passthrough=yes \
            src-address-list=LAN
        ```
        
        Cara Kerja :
        
        - Traffic packet dengan src network “LAN” dan Dst **selain** network “LAN” (Internet) maka akan mengeksekusi rule tersebut. Lalu Traffic akan di proses menggunakan **Metode dan Algoritma PCC** untuk Balance Traffic.
        setelah packet di proses, Packet akan di tandai dengan “Mark Connection”
        - Tanda “Mark Connection” akan di jadikan “Mark Routing” untuk di buatkan route khusus untuk Load Balance
        
        Note :
        
        - Konfigurasi PCC
            
            Tiknik PCC ini menggunakan algoritma Connection splitting untuk melakukan load balance.
            
            ![Re (1).png](PCC%20Load%20Balance%20-%20Mikrotik%2002519ea386a14e0d9bc5fa9dfcc46517/Re_(1).png)
            
            - Divisor (pembagi) : nilai yang menentukan berapa banyak jalur untuk membagi traffic. Contoh: jika ada 2 ISP (bandwidh sama) maka Divisor 2.
            - Remainder (Sisa) : hasil operasi pembagian traffic (Sequence number), jika Divisor nya 2, maka akan mendapatkan Sisa 0 dan 1.
        - PCC dengan **Bandwidth Berbeda**
            
            ![Untitled](PCC%20Load%20Balance%20-%20Mikrotik%2002519ea386a14e0d9bc5fa9dfcc46517/Untitled%201.png)
            
            Jika bandwidth berbeda, maka Bandwidth yang lebih besar harus di bagi untuk menyamakan rule dengan bandwidth yang kecil.
            
        
        ### Configuration Firewall Mangel
        
        ![Untitled](/assets/img/pcc-mikrotik/confpcc.png)
        
5. Create IP Route Network
⇒ Berfungsi untuk merutekan Rule yang telah di buat dengan set “Routing-mark” pada routing table tersebut.
    
    ```bash
    /ip route
    add check-gateway=ping distance=1 gateway=10.10.10.1 routing-mark=To-ISP1
    add check-gateway=ping distance=1 gateway=172.16.2.1 routing-mark=To-ISP2
    add distance=1 gateway=10.10.10.1
    add distance=2 gateway=172.16.2.1
    ```
    
6. Configuration Complete,
⇒ Untuk melihat berhasil atau tidak, bisa dilihat dari rule mangel apakah **Packet dan Bytes** bertambah dan lihat traffic pada kedua ISP di bagi dua atau tidak pada saat **SPEEDTEST.**

## Thank You