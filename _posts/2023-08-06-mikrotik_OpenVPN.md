---
layout: post
title: OpenVPN - Mikrotik
date: 2023-08-06
pin: 
tags: mikrotik network
splash_img_source: /assets/img/mikrotik-openvpn.png
splash_img_caption: 
---
Ada salah satu perusahan, yang mana perusahaan ini memberlakukan pekerjaan untuk karyawannya secara hybrid WFO dan WFA. Setelah itu, perusahaan ini juga memiliki sebuah aplikasi private yang hanya bisa di buka menggunakan network kantor.

Masalah ini muncul pada saat karyawan sedang bekerjaa WFA/WFH, dikarenkan aplikasi pada perusahaan tersebut hanya bisa di buka menggunakan network kantor saja. lalu bagaimana caranya agar hp/laptop di rumah bisa terkoneksi dengan network yang ada pada kantor, yakni menggunakan <b>VPN</b>.

VPN ini bermacam-macam protokolnya, mulai dari PPTP, L2TP, SSTP, OpenVPN. Tetapi pada lab kali ini protokol yang akan dipakai yakni, <b>OpenVPN</b>.

## OpenVPN
OpenVPN adalah sebuah Software open-source yang berfungsi untuk membuat Virutal Private Network (VPN) yang Secure di atas public network seperti Internet. VPN ini mengenkripsi traffic data antara device yang dimiliki dengan server VPN.

##### Syarat Setting OpenVPN
- Mikrotik
- IP Public
- Internet

##### Bagaimana Cara Setting OpenVPN di Mikrotik?
Note : Pastikan Mikrotik sudah di setting basic configuration dan sudah dipasangkan ip public pada router.

##### Certificate
- Certificate Generate
```sh
/certificate
add name=ca-template common-name=ca days-valid=3650 key-size=2048 key-usage=crl-sign,key-cert-sign
add name=server-template common-name=server days-valid=3650 key-size=2048 key-usage=digital-signature,key-encipherment,tls-server
add name=client-template common-name=client days-valid=3650 key-size=2048 key-usage=tls-client
```

- Certificate Sign
```sh
/certificate
sign ca-template name=ca-certificate
sign server-template name=server-certificate ca=ca-certificate
sign client-template name=client-certificate ca=ca-certificate
```

- Certificate Trust
```sh
/certificate
set ca-certificate trusted=yes
set server-certificate trusted=yes
```

- Certificate Export
```sh
/certificate
export-certificate ca-certificate export-passphrase=""
export-certificate client-certificate export-passphrase="12345678"
```

- Cek File Export Certificate
```sh
[admin@mikrotik] > file print where name~"cert_"
Columns: NAME, TYPE, SIZE, CREATION-TIME
#  NAME                                TYPE       SIZE  CREATION-TIME       
2  cert_export_ca-certificate.crt      .crt file  1107  aug/06/2023 18:41:47
3  cert_export_client-certificate.crt  .crt file  1139  aug/06/2023 18:41:48
0  cert_export_client-certificate.key  .key file  1858  aug/06/2023 18:41:48
```

##### OpenVPN Server di Mikrotik
- OpenVPN Enable
```sh
/interface ovpn-server server
set auth=sha1,md5 certificate=server-certificate cipher=blowfish128,aes128,aes192,aes256 enabled=yes require-client-certificate=yes
```

- OpenVPN user
```sh
/ppp secret
add local-address=172.10.1.1 name=testing remote-address=172.10.1.2 password=123
```
bisa setting profile juga, tetapi untuk pengetestan saya memakai defautl profile saja.

##### Konfigurasi file .ovpn 
- Download File 3 file Certificate yang sebelumnya di export
- Buat file client.ovpn di notepad/notepad++


```sh
client
dev tun
proto tcp
remote 103.33.13.89 1194
nobind
remote-cert-tls server
verb 3
cipher AES-256-CBC
auth SHA1
auth-user-pass
redirect-gateway def1

<ca> 
-----BEGIN CERTIFICATE-----
#file ca-certificate.crt
-----END CERTIFICATE-----
</ca>

<cert> 
-----BEGIN CERTIFICATE-----
#file client-certificate.crt
-----END CERTIFICATE-----
</cert>

<key> 
-----BEGIN ENCRYPTED PRIVATE KEY-----
#file client-certificate.key
-----END ENCRYPTED PRIVATE KEY-----
</key>
```

##### Test VPN
- Pastikan sudah menginstall OpenVPN pada Laptop/HP
- Import file client.ovpn kedalam aplikai OpenVPN
- lalu isi username, password, dan private key. setelah di isi silahkan di connect ke vpnnya
<br><br>
<img src="/assets/img/mikrotik-openvpn1.png" alt="OpenVPN Client" style="width: 40%;">