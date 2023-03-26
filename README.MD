# First excercise
## First method
```
[root@selinux ~]# systemctl status firewalld
● firewalld.service - firewalld - dynamic firewall daemon
   Loaded: loaded (/usr/lib/systemd/system/firewalld.service; disabled; vendor preset: enabled)
   Active: inactive (dead)
     Docs: man:firewalld(1)
[root@selinux ~]# nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
[root@selinux ~]# getenforce
Enforcing
[root@selinux ~]# cat /var/log/audit/audit.log | grep 4881
type=AVC msg=audit(1679833912.511:790): avc:  denied  { name_bind } for  pid=2726 comm="nginx" src=4881 scontext=system_u:system_r:httpd_t:s0 tcontext=system_u:object_r:unreserved_port_t:s0 tclass=tcp_socket permissive=0
[root@selinux ~]# grep 1679833912.511:790 /var/log/audit/audit.log | audit2why
type=AVC msg=audit(1679833912.511:790): avc:  denied  { name_bind } for  pid=2726 comm="nginx" src=4881 scontext=system_u:system_r:httpd_t:s0 tcontext=system_u:object_r:unreserved_port_t:s0 tclass=tcp_socket permissive=0

	Was caused by:
	The boolean nis_enabled was set incorrectly. 
	Description:
	Allow nis to enabled

	Allow access by executing:
	# setsebool -P nis_enabled 1
[root@selinux ~]# setsebool -P nis_enabled 1
[root@selinux ~]# systemctl restart nginx
[root@selinux ~]# systemctl status nginx
● nginx.service - The nginx HTTP and reverse proxy server
   Loaded: loaded (/usr/lib/systemd/system/nginx.service; disabled; vendor preset: disabled)
   Active: active (running) since Sun 2023-03-26 12:56:18 UTC; 11s ago
  Process: 2928 ExecStart=/usr/sbin/nginx (code=exited, status=0/SUCCESS)
  Process: 2926 ExecStartPre=/usr/sbin/nginx -t (code=exited, status=0/SUCCESS)
  Process: 2925 ExecStartPre=/usr/bin/rm -f /run/nginx.pid (code=exited, status=0/SUCCESS)
 Main PID: 2930 (nginx)
   CGroup: /system.slice/nginx.service
           ├─2930 nginx: master process /usr/sbin/nginx
           └─2932 nginx: worker process

Mar 26 12:56:18 selinux systemd[1]: Starting The nginx HTTP and reverse proxy server...
Mar 26 12:56:18 selinux nginx[2926]: nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
Mar 26 12:56:18 selinux nginx[2926]: nginx: configuration file /etc/nginx/nginx.conf test is successful
Mar 26 12:56:18 selinux systemd[1]: Started The nginx HTTP and reverse proxy server.
[root@selinux ~]# getsebool -a | grep nis_enabled
nis_enabled --> on
[root@selinux ~]# setsebool -P nis_enabled off
[root@selinux ~]# systemctl status nginx
● nginx.service - The nginx HTTP and reverse proxy server
   Loaded: loaded (/usr/lib/systemd/system/nginx.service; disabled; vendor preset: disabled)
   Active: failed (Result: exit-code) since Sun 2023-03-26 12:59:20 UTC; 10s ago
  Process: 2928 ExecStart=/usr/sbin/nginx (code=exited, status=0/SUCCESS)
  Process: 2954 ExecStartPre=/usr/sbin/nginx -t (code=exited, status=1/FAILURE)
  Process: 2953 ExecStartPre=/usr/bin/rm -f /run/nginx.pid (code=exited, status=0/SUCCESS)
 Main PID: 2930 (code=exited, status=0/SUCCESS)

Mar 26 12:59:20 selinux systemd[1]: Stopped The nginx HTTP and reverse proxy server.
Mar 26 12:59:20 selinux systemd[1]: Starting The nginx HTTP and reverse proxy server...
Mar 26 12:59:20 selinux nginx[2954]: nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
Mar 26 12:59:20 selinux nginx[2954]: nginx: [emerg] bind() to 0.0.0.0:4881 failed (13: Permission denied)
Mar 26 12:59:20 selinux nginx[2954]: nginx: configuration file /etc/nginx/nginx.conf test failed
Mar 26 12:59:20 selinux systemd[1]: nginx.service: control process exited, code=exited status=1
Mar 26 12:59:20 selinux systemd[1]: Failed to start The nginx HTTP and reverse proxy server.
Mar 26 12:59:20 selinux systemd[1]: Unit nginx.service entered failed state.
Mar 26 12:59:20 selinux systemd[1]: nginx.service failed.
```
## Second method
```
[root@selinux ~]# semanage port -l | grep http
http_cache_port_t              tcp      8080, 8118, 8123, 10001-10010
http_cache_port_t              udp      3130
http_port_t                    tcp      80, 81, 443, 488, 8008, 8009, 8443, 9000
pegasus_http_port_t            tcp      5988
pegasus_https_port_t           tcp      5989
[root@selinux ~]# semanage port -a -t http_port_t -p tcp 4881
[root@selinux ~]# semanage port -l | grep http
http_cache_port_t              tcp      8080, 8118, 8123, 10001-10010
http_cache_port_t              udp      3130
http_port_t                    tcp      4881, 80, 81, 443, 488, 8008, 8009, 8443, 9000
pegasus_http_port_t            tcp      5988
pegasus_https_port_t           tcp      5989
[root@selinux ~]# systemctl start nginx
[root@selinux ~]# systemctl status nginx
● nginx.service - The nginx HTTP and reverse proxy server
   Loaded: loaded (/usr/lib/systemd/system/nginx.service; disabled; vendor preset: disabled)
   Active: active (running) since Sun 2023-03-26 13:03:38 UTC; 2s ago
  Process: 2993 ExecStart=/usr/sbin/nginx (code=exited, status=0/SUCCESS)
  Process: 2991 ExecStartPre=/usr/sbin/nginx -t (code=exited, status=0/SUCCESS)
  Process: 2990 ExecStartPre=/usr/bin/rm -f /run/nginx.pid (code=exited, status=0/SUCCESS)
 Main PID: 2995 (nginx)
   CGroup: /system.slice/nginx.service
           ├─2995 nginx: master process /usr/sbin/nginx
           └─2997 nginx: worker process

Mar 26 13:03:38 selinux systemd[1]: Starting The nginx HTTP and reverse proxy server...
Mar 26 13:03:38 selinux nginx[2991]: nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
Mar 26 13:03:38 selinux nginx[2991]: nginx: configuration file /etc/nginx/nginx.conf test is successful
Mar 26 13:03:38 selinux systemd[1]: Started The nginx HTTP and reverse proxy server.
[root@selinux ~]# semanage port -d -t http_port_t -p tcp 4881
[root@selinux ~]# semanage port -l | grep http
http_cache_port_t              tcp      8080, 8118, 8123, 10001-10010
http_cache_port_t              udp      3130
http_port_t                    tcp      80, 81, 443, 488, 8008, 8009, 8443, 9000
pegasus_http_port_t            tcp      5988
pegasus_https_port_t           tcp      5989
[root@selinux ~]# systemctl restart nginx
Job for nginx.service failed because the control process exited with error code. See "systemctl status nginx.service" and "journalctl -xe" for details.
[root@selinux ~]# systemctl status nginx
● nginx.service - The nginx HTTP and reverse proxy server
   Loaded: loaded (/usr/lib/systemd/system/nginx.service; disabled; vendor preset: disabled)
   Active: failed (Result: exit-code) since Sun 2023-03-26 13:04:55 UTC; 13s ago
  Process: 2993 ExecStart=/usr/sbin/nginx (code=exited, status=0/SUCCESS)
  Process: 3015 ExecStartPre=/usr/sbin/nginx -t (code=exited, status=1/FAILURE)
  Process: 3014 ExecStartPre=/usr/bin/rm -f /run/nginx.pid (code=exited, status=0/SUCCESS)
 Main PID: 2995 (code=killed, signal=KILL)
   CGroup: /system.slice/nginx.service

Mar 26 13:04:55 selinux systemd[1]: Unit nginx.service entered failed state.
Mar 26 13:04:55 selinux systemd[1]: nginx.service failed.
Mar 26 13:04:55 selinux systemd[1]: Starting The nginx HTTP and reverse proxy server...
Mar 26 13:04:55 selinux nginx[3015]: nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
Mar 26 13:04:55 selinux nginx[3015]: nginx: [emerg] bind() to 0.0.0.0:4881 failed (13: Permission denied)
Mar 26 13:04:55 selinux nginx[3015]: nginx: configuration file /etc/nginx/nginx.conf test failed
Mar 26 13:04:55 selinux systemd[1]: nginx.service: control process exited, code=exited status=1
Mar 26 13:04:55 selinux systemd[1]: Failed to start The nginx HTTP and reverse proxy server.
Mar 26 13:04:55 selinux systemd[1]: Unit nginx.service entered failed state.
Mar 26 13:04:55 selinux systemd[1]: nginx.service failed.
```
## Third method
```
[root@selinux ~]# grep nginx /var/log/audit/audit.log | audit2allow -M nginx
******************** IMPORTANT ***********************
To make this policy package active, execute:

semodule -i nginx.pp
[root@selinux ~]# semodule -i nginx.pp
[root@selinux ~]# systemctl start nginx
[root@selinux ~]# systemctl status nginx
● nginx.service - The nginx HTTP and reverse proxy server
   Loaded: loaded (/usr/lib/systemd/system/nginx.service; disabled; vendor preset: disabled)
   Active: active (running) since Sun 2023-03-26 13:09:14 UTC; 8s ago
  Process: 21828 ExecStart=/usr/sbin/nginx (code=exited, status=0/SUCCESS)
  Process: 21826 ExecStartPre=/usr/sbin/nginx -t (code=exited, status=0/SUCCESS)
  Process: 21825 ExecStartPre=/usr/bin/rm -f /run/nginx.pid (code=exited, status=0/SUCCESS)
 Main PID: 21830 (nginx)
   CGroup: /system.slice/nginx.service
           ├─21830 nginx: master process /usr/sbin/nginx
           └─21832 nginx: worker process

Mar 26 13:09:14 selinux systemd[1]: Starting The nginx HTTP and reverse proxy server...
Mar 26 13:09:14 selinux nginx[21826]: nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
Mar 26 13:09:14 selinux nginx[21826]: nginx: configuration file /etc/nginx/nginx.conf test is successful
Mar 26 13:09:14 selinux systemd[1]: Started The nginx HTTP and reverse proxy server.
[root@selinux ~]# semodule -r nginx
libsemanage.semanage_direct_remove_key: Removing last nginx module (no other nginx module exists at another priority).
```
# Second excercise

```
[vagrant@client ~]$ nsupdate -k /etc/named.zonetransfer.key
> server 192.168.50.10
> zone ddns.lab
> update add www.ddns.lab. 10 A 192.168.50.15
> send
update failed: SERVFAIL
> quit
[vagrant@client ~]$ sudo -i
[root@client ~]# cat /var/log/audit/audit.log | audit2why
[root@ns01 ~]# cat /var/log/audit/audit.log | audit2why
type=AVC msg=audit(1679837050.985:1877): avc:  denied  { create } for  pid=5073 comm="isc-worker0000" name="named.ddns.lab.view1.jnl" scontext=system_u:system_r:named_t:s0 tcontext=system_u:object_r:etc_t:s0 tclass=file permissive=0

	Was caused by:
		Missing type enforcement (TE) allow rule.

		You can use audit2allow to generate a loadable module to allow this access.

[root@ns01 ~]# ls -laZ /etc/named
drw-rwx---. root named system_u:object_r:etc_t:s0       .
drwxr-xr-x. root root  system_u:object_r:etc_t:s0       ..
drw-rwx---. root named unconfined_u:object_r:etc_t:s0   dynamic
-rw-rw----. root named system_u:object_r:etc_t:s0       named.50.168.192.rev
-rw-rw----. root named system_u:object_r:etc_t:s0       named.dns.lab
-rw-rw----. root named system_u:object_r:etc_t:s0       named.dns.lab.view1
-rw-rw----. root named system_u:object_r:etc_t:s0       named.newdns.lab
[root@ns01 ~]# chcon -R -t named_zone_t /etc/named
[root@ns01 ~]# ls -laZ /etc/named
drw-rwx---. root named system_u:object_r:named_zone_t:s0 .
drwxr-xr-x. root root  system_u:object_r:etc_t:s0       ..
drw-rwx---. root named unconfined_u:object_r:named_zone_t:s0 dynamic
-rw-rw----. root named system_u:object_r:named_zone_t:s0 named.50.168.192.rev
-rw-rw----. root named system_u:object_r:named_zone_t:s0 named.dns.lab
-rw-rw----. root named system_u:object_r:named_zone_t:s0 named.dns.lab.view1
-rw-rw----. root named system_u:object_r:named_zone_t:s0 named.newdns.lab
[root@client ~]# nsupdate -k /etc/named.zonetransfer.key
> server 192.168.50.10
> zone ddns.lab
> update add www.ddns.lab. 60 A 192.168.50.15
> send
> quit
[root@client ~]# dig www.ddns.lab

; <<>> DiG 9.11.4-P2-RedHat-9.11.4-26.P2.el7_9.13 <<>> www.ddns.lab
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 11322
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL: 2

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;www.ddns.lab.			IN	A

;; ANSWER SECTION:
www.ddns.lab.		60	IN	A	192.168.50.15

;; AUTHORITY SECTION:
ddns.lab.		3600	IN	NS	ns01.dns.lab.

;; ADDITIONAL SECTION:
ns01.dns.lab.		3600	IN	A	192.168.50.10

;; Query time: 1 msec
;; SERVER: 192.168.50.10#53(192.168.50.10)
;; WHEN: Sun Mar 26 13:44:00 UTC 2023
;; MSG SIZE  rcvd: 96
[vagrant@client ~]$ uptime
 13:52:22 up 1 min,  1 user,  load average: 0.00, 0.00, 0.00
[vagrant@client ~]$ dig @192.168.50.10 www.ddns.lab

; <<>> DiG 9.11.4-P2-RedHat-9.11.4-26.P2.el7_9.13 <<>> @192.168.50.10 www.ddns.lab
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 23800
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL: 2

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;www.ddns.lab.			IN	A

;; ANSWER SECTION:
www.ddns.lab.		60	IN	A	192.168.50.15

;; AUTHORITY SECTION:
ddns.lab.		3600	IN	NS	ns01.dns.lab.

;; ADDITIONAL SECTION:
ns01.dns.lab.		3600	IN	A	192.168.50.10

;; Query time: 0 msec
;; SERVER: 192.168.50.10#53(192.168.50.10)
;; WHEN: Sun Mar 26 13:52:26 UTC 2023
;; MSG SIZE  rcvd: 96
[vagrant@client ~]$ sudo restorecon -v -R /etc/named
```
