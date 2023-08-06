---
layout: post
title: Mikrotik - OpenVPN
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

### Syarat Konfigurtasi OpenVPN
- Mikrotik
- IP Public
- Internet









