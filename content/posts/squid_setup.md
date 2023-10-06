+++
title = 'Squid_setup'
date = 2026-06-18T23:15:45+08:00
draft = false
categories = ['Operating System']
tags = ['Linux','squid']
+++



    1. yum -y update
    
    2. yum install openssl
    
    3. yum install squid
    
    4. vim /etc/squid/squid.conf
    
    5. tail -f /var/log/squid/access.log
    
    6. systemctl start squid
    
    7. systemctl status squid
    
    8. ss -nlp | grep squid | grep 3128
    
    dns_nameservers 8.8.8.8 8.8.4.4
