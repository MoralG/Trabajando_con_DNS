# Servidor DNSmasq

#### Instala el servidor dns dnsmasq en pandora.iesgn.org y configúralo para que los clientes puedan conocer los nombres necesarios.

#### *Tarea 1 (2 punto)(Obligatorio):* Modifica los clientes para que utilicen el nuevo servidor dns. Realiza una consulta a www.iesgn.org, y a www.josedomingo.org. Realiza una prueba de funcionamiento para comprobar que el servidor dnsmasq funciona como cache dns. Muestra el fichero hosts del cliente para demostrar que no estás utilizando resolución estática. Realiza una consulta directa al servidor dnsmasq. ¿Se puede realizar resolución inversa?. Documenta la tarea en redmine.

##### *Configuramos en el servidor*:

###### Instalamos el paquete *dnsmasq*
~~~
vagrant@fernando:~$ sudo apt install dnsmasq
~~~

###### Modificamos el fichero */etc/hosts*
~~~
192.168.10.100 www.iesgn.org
192.168.10.110 departamentos.iesgn.org
192.168.10.120 alejandro.iesgn.org
~~~

###### Modificamos el fichero */etc/dnsmasq.conf*
~~~
strict-order
interface=eth2
~~~

###### Reiniciamos el servicio
~~~
sudo systemctl restart dnsmasq.service
~~~

##### *Configuramos el cliente*:

###### Instalamos el paquete *dnsutils*
~~~
sudo apt install dnsutils
~~~

###### Modificamos el fichero */etc/resolv.conf*
~~~
nameserver 192.168.100.1
~~~

##### *Comprobaciones*:

~~~
vagrant@ClienteDNS:~$ dig www.iesgn.org

; <<>> DiG 9.11.5-P4-5.1-Debian <<>> www.iesgn.org
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 57233
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;www.iesgn.org.			IN	A

;; ANSWER SECTION:
www.iesgn.org.		0	IN	A	192.168.10.100

;; Query time: 0 msec
;; SERVER: 192.168.100.1#53(192.168.100.1)
;; WHEN: Wed Nov 20 07:45:37 GMT 2019
;; MSG SIZE  rcvd: 58 
~~~
~~~
vagrant@ClienteDNS:~$ dig www.josedomingo.org

; <<>> DiG 9.11.5-P4-5.1-Debian <<>> www.josedomingo.org
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 1029
;; flags: qr rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 5, ADDITIONAL: 6

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
; COOKIE: 13decbea20d662d098546e065dd4eedb7043d8028585f7a3 (good)
;; QUESTION SECTION:
;www.josedomingo.org.		IN	A

;; ANSWER SECTION:
www.josedomingo.org.	473	IN	CNAME	playerone.josedomingo.org.
playerone.josedomingo.org. 815	IN	A	137.74.161.90

;; AUTHORITY SECTION:
josedomingo.org.	84505	IN	NS	ns2.cdmon.net.
josedomingo.org.	84505	IN	NS	ns4.cdmondns-01.org.
josedomingo.org.	84505	IN	NS	ns5.cdmondns-01.com.
josedomingo.org.	84505	IN	NS	ns3.cdmon.net.
josedomingo.org.	84505	IN	NS	ns1.cdmon.net.

;; ADDITIONAL SECTION:
ns1.cdmon.net.		84470	IN	A	35.189.106.232
ns2.cdmon.net.		84470	IN	A	35.195.57.29
ns3.cdmon.net.		84470	IN	A	35.157.47.125
ns4.cdmondns-01.org.	84505	IN	A	52.58.66.183
ns5.cdmondns-01.com.	84470	IN	A	52.59.146.62

;; Query time: 3 msec
;; SERVER: 192.168.100.1#53(192.168.100.1)
;; WHEN: Wed Nov 20 07:44:27 GMT 2019
;; MSG SIZE  rcvd: 322
~~~
~~~
vagrant@ClienteDNS:~$ dig www.elmundo.com

; <<>> DiG 9.11.5-P4-5.1-Debian <<>> www.elmundo.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 29220
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 2, ADDITIONAL: 3

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
; COOKIE: 0e7caef21bffd9d0a41b21295dd4ef5992bae217242d243a (good)
;; QUESTION SECTION:
;www.elmundo.com.		IN	A

;; ANSWER SECTION:
www.elmundo.com.	3600	IN	A	131.0.136.66

;; AUTHORITY SECTION:
elmundo.com.		172800	IN	NS	dws01.dwsistemas.net.
elmundo.com.		172800	IN	NS	dws00.dwsistemas.net.

;; ADDITIONAL SECTION:
dws00.dwsistemas.net.	172800	IN	A	131.0.136.2
dws01.dwsistemas.net.	172800	IN	A	131.0.136.3

;; Query time: 1202 msec
;; SERVER: 192.168.100.1#53(192.168.100.1)
;; WHEN: Wed Nov 20 07:46:34 GMT 2019
;; MSG SIZE  rcvd: 174

vagrant@ClienteDNS:~$ dig www.elmundo.com

; <<>> DiG 9.11.5-P4-5.1-Debian <<>> www.elmundo.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 2064
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;www.elmundo.com.		IN	A

;; ANSWER SECTION:
www.elmundo.com.	3598	IN	A	131.0.136.66

;; Query time: 0 msec
;; SERVER: 192.168.100.1#53(192.168.100.1)
;; WHEN: Wed Nov 20 07:46:37 GMT 2019
;; MSG SIZE  rcvd: 60
~~~
~~~
vagrant@ClienteDNS:~$ dig -x 192.168.10.100

; <<>> DiG 9.11.5-P4-5.1-Debian <<>> -x 192.168.10.100
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 34996
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;100.10.168.192.in-addr.arpa.	IN	PTR

;; ANSWER SECTION:
100.10.168.192.in-addr.arpa. 0	IN	PTR	www.iesgn.org.

;; Query time: 0 msec
;; SERVER: 192.168.100.1#53(192.168.100.1)
;; WHEN: Wed Nov 20 07:46:58 GMT 2019
;; MSG SIZE  rcvd: 83
~~~
~~~
vagrant@ClienteDNS:~$ dig alejandro.iesgn.org

; <<>> DiG 9.11.5-P4-5.1-Debian <<>> alejandro.iesgn.org
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 30585
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;alejandro.iesgn.org.		IN	A

;; ANSWER SECTION:
alejandro.iesgn.org.	0	IN	A	192.168.10.120

;; Query time: 0 msec
;; SERVER: 192.168.100.1#53(192.168.100.1)
;; WHEN: Wed Nov 20 07:47:15 GMT 2019
;; MSG SIZE  rcvd: 64
~~~