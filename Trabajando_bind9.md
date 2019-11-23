# Servidor bind9

#### Desinstala el servidor dnsmasq del ejercicio anterior e instala un servidor dns bind9. Las características del servidor DNS que queremos instalar son las siguientes

* El servidor DNS se llama pandora.iesgn.org y por supuesto, va a ser el servidor con autoridad para la zona iesgn.org.
* Vamos a suponer que tenemos un servidor para recibir los correos que se llame correo.iesgn.org y que está en la dirección x.x.x.200 (esto es ficticio).
* Vamos a suponer que tenemos un servidor ftp que se llame ftp.iesgn.org y que está en x.x.x.201 (esto es ficticio)
* Además queremos nombrar a los clientes.
* También hay que nombrar a los virtual hosts de apache: www.iesgn.org y departementos.iesgn.org
* Se tienen que resolver las direcciones ipv6 de las distintas máuinas (inventante las ficticias).
* Se tienen que configurar la zona de resolución inversa.

#### *Tarea 2 (2 puntos)(Obligatorio):* Realiza la instalación y configuración del servidor bind9 con las características anteriomente señaladas. Entrega las zonas que has definido. Muestra al profesor su funcionamiento

#### *Configuración del Servidor:*

###### Instalamos el paquete de *bind9*

~~~
sudo apt install bind9
~~~

##### *Realizamos la resolución directa*

###### Modificamos el fichero */etc/bind/named.conf.local*

~~~
zone "iesgn.org"
{
  file "db.iesgn.org";
  type master;
  allow-transfer { 10.0.0.2; };
  notify yes;
};
~~~

###### Copiamos un fichero de configuración ya creado para la resolución directa

~~~
sudo cp /etc/bind/db.local /var/cache/bind/db.iesgn.org
~~~

###### Modificamos el fichero */var/cache/bind/db.iesgn.org*

~~~
$TTL    86400
@       IN      SOA     alejandro.iesgn.org. ale95mogra.iesgn.org. (
                              4         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                          86400 )       ; Negative Cache TTL

@               IN      NS      alejandro.iesgn.org.
@               IN      MX 10   correo.iesgn.org.

$ORIGIN iesgn.org.
alejandro       IN      A       10.0.0.1
correo          IN      A       10.0.0.200
ftp             IN      A       10.0.0.201
cliente1        IN      A       10.0.0.202
cliente2        IN      A       10.0.0.203
web             IN      A       10.0.0.204
www             IN      CNAME   web
departamentos   IN      CNAME   web

; IPv6

alejandro       IN      AAAA    fe80::a00:27ff:fed9:6fc4
correo          IN      AAAA    fe80::a00:27ff:feb8:7fc4
ftp             IN      AAAA    fe80::a00:27ff:fec2:6cc4
cliente1        IN      AAAA    fe80::a00:27ff:fed7:4fc4
cliente2        IN      AAAA    fe80::a00:27ff:fed3:6fc7
web             IN      AAAA    fe80::a00:27ff:feb7:6dc5
~~~

##### *Realizamos la resolución inversa*

###### Añadimos en el fichero */etc/bind/named.conf.local*

~~~
zone "0.0.10.in-addr.arpa"
{
  file "db.0.0.10";
  type master;
  allow-transfer { 10.0.0.2; };
  notify yes;
};


zone "f.f.7.2.0.0.a.0.0.0.0.0.0.0.0.0.0.0.0.0.0.8.e.f.ip6.arpa"
{
  file "db.f.f.7.2.0.0.a.0.0.0.0.0.8.e.f";
  type master;
};
~~~

###### Copiamos un fichero de configuración ya creado para la resolución inversa y la inversa ipv6

~~~
sudo cp /etc/bind/db.empty /var/cache/bind/db.0.0.10
sudo cp /etc/bind/db.empty /var/cache/bind/db.f.f.7.2.0.0.a.0.0.0.0.0.8.e.f
~~~

###### Modificamos el fichero */var/cache/bind/db.0.0.10*

~~~
$TTL    86400
@       IN      SOA     alejandro.iesgn.org. ale95mogra.iesgn.org. (
                              4         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                          86400 )       ; Negative Cache TTL
;
@       IN      NS      alejandro.iesgn.org.
@       IN      NS      alejandro-slave.iesgn.org.

$ORIGIN 0.0.10.in-addr.arpa.
1       IN      PTR     alejandro.iesgn.org.
2       IN      PTR     alejandro-slave.iesgn.org.
200     IN      PTR     correo.iesgn.org.
201     IN      PTR     ftp.iesgn.org.
202     IN      PTR     cliente1.iesgn.org.
203     IN      PTR     cliente2.iesgn.org.
204     IN      PTR     web.iesgn.org.
~~~

###### Modificamos el fichero */var/cache/bind/db.f.f.7.2.0.0.a.0.0.0.0.0.8.e.f*

~~~
$TTL    86400
@       IN      SOA     alejandro.iesgn.org. ale95mogra.iesgn.org. (
                              3         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                          86400 )       ; Negative Cache TTL
;
@       IN      NS      alejandro.iesgn.org.

$ORIGIN 0.0.0.0.0.0.0.0.0.0.0.0.0.8.e.f.ip6.arpa.
4.c.f.6.9.d.e.f.f.f.7.2.0.0.a   IN PTR  alejandro.iesgn.org.
4.c.f.7.8.b.e.f.f.f.7.2.0.0.a   IN PTR  correo.iesgn.org.
4.c.c.6.2.c.e.f.f.f.7.2.0.0.a   IN PTR  ftp.iesgn.org.
4.c.f.4.7.d.e.f.f.f.7.2.0.0.a   IN PTR  cliente1.iesgn.org.
7.c.f.6.3.d.e.f.f.f.7.2.0.0.a   IN PTR  cliente2.iesgn.org.
5.c.d.6.7.b.e.f.f.f.7.2.0.0.a   IN PTR  web.iesgn.org.
~~~

###### Tenemos que reiniciar el bind

~~~
sudo rndc reload
~~~

###### Para que nos muestre los errores

~~~
sudo named-checkconf

sudo named-checkzone iesgn.org /var/cache/bind/db.iesgn.org
  zone iesgn.org/IN: loaded serial 3
  OK
~~~

#### *Configuración del Cliente:*

###### En el cliente tenemos que añadir en el fichero */etc/resolv.conf* lo siguiente

~~~
nameserver 10.0.0.1
~~~


#### *Tarea 3 (2 puntos)(Obligatorio):* Realiza las consultas dig/nslookup desde los clientes preguntando por los siguientes

* Dirección de vagrant.iesgn.org, www.iesgn.org, ftp.iesgn.org

~~~
dig alejandro.iesgn.org

; <<>> DiG 9.11.5-P4-5.1-Debian <<>> alejandro.iesgn.org
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 31799
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 2, ADDITIONAL: 3

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
; COOKIE: 747411bfee3e78c604a4f8335dd6ef00588627e6293cad4a (good)
;; QUESTION SECTION:
;alejandro.iesgn.org.		IN	A

;; ANSWER SECTION:
alejandro.iesgn.org.	86400	IN	A	10.0.0.1

;; AUTHORITY SECTION:
iesgn.org.		86400	IN	NS	alejandro.iesgn.org.

;; ADDITIONAL SECTION:
alejandro.iesgn.org.	86400	IN	AAAA	fe80::a00:27ff:fed9:6fc4

;; Query time: 0 msec
;; SERVER: 10.0.0.1#53(10.0.0.1)
;; WHEN: Thu Nov 21 20:09:36 GMT 2019
;; MSG SIZE  rcvd: 180
~~~

~~~
dig www.iesgn.org

; <<>> DiG 9.11.5-P4-5.1-Debian <<>> www.iesgn.org
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 2941
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 2, ADDITIONAL: 4

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
; COOKIE: 5fdd43f94f505dce84f566365dd6ef11d3f6204fba65b043 (good)
;; QUESTION SECTION:
;www.iesgn.org.			IN	A

;; ANSWER SECTION:
www.iesgn.org.		86400	IN	CNAME	web.iesgn.org.
web.iesgn.org.		86400	IN	A	10.0.0.204

;; AUTHORITY SECTION:
iesgn.org.		86400	IN	NS	alejandro.iesgn.org.

;; ADDITIONAL SECTION:
alejandro.iesgn.org.	86400	IN	A	10.0.0.1
alejandro.iesgn.org.	86400	IN	AAAA	fe80::a00:27ff:fed9:6fc4

;; Query time: 0 msec
;; SERVER: 10.0.0.1#53(10.0.0.1)
;; WHEN: Thu Nov 21 20:09:53 GMT 2019
;; MSG SIZE  rcvd: 218
~~~

~~~
dig ftp.iesgn.org

; <<>> DiG 9.11.5-P4-5.1-Debian <<>> ftp.iesgn.org
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 31489
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 2, ADDITIONAL: 4

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
; COOKIE: b57d092788cbf66ab66c48f75dd6ef209d09264d42d8830e (good)
;; QUESTION SECTION:
;ftp.iesgn.org.			IN	A

;; ANSWER SECTION:
ftp.iesgn.org.		86400	IN	A	10.0.0.201

;; AUTHORITY SECTION:
iesgn.org.		86400	IN	NS	alejandro.iesgn.org.

;; ADDITIONAL SECTION:
alejandro.iesgn.org.	86400	IN	A	10.0.0.1
alejandro.iesgn.org.	86400	IN	AAAA	fe80::a00:27ff:fed9:6fc4

;; Query time: 0 msec
;; SERVER: 10.0.0.1#53(10.0.0.1)
;; WHEN: Thu Nov 21 20:10:08 GMT 2019
;; MSG SIZE  rcvd: 200
~~~

* El servidor DNS con autoridad sobre la zona del dominio iesgn.org

~~~
dig ns iesgn.org

; <<>> DiG 9.11.5-P4-5.1-Debian <<>> ns iesgn.org
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 20084
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 4

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
; COOKIE: fc5fcd7efd2f72ad889357a25dd6ef60d0632a04e0c526f8 (good)
;; QUESTION SECTION:
;iesgn.org.			IN	NS

;; ANSWER SECTION:
iesgn.org.		86400	IN	NS	alejandro.iesgn.org.

;; ADDITIONAL SECTION:
alejandro.iesgn.org.	86400	IN	A	10.0.0.1
alejandro.iesgn.org.	86400	IN	AAAA	fe80::a00:27ff:fed9:6fc4

;; Query time: 0 msec
;; SERVER: 10.0.0.1#53(10.0.0.1)
;; WHEN: Thu Nov 21 20:11:12 GMT 2019
;; MSG SIZE  rcvd: 180
~~~

* El servidor de correo configurado para iesgn.org

~~~
dig mx iesgn.org

; <<>> DiG 9.11.5-P4-5.1-Debian <<>> mx iesgn.org
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 58950
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 2, ADDITIONAL: 6

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
; COOKIE: 929c22370ef971c97110e1ca5dd6ef6d58c4448f2ea7eb23 (good)
;; QUESTION SECTION:
;iesgn.org.			IN	MX

;; ANSWER SECTION:
iesgn.org.		86400	IN	MX	10 correo.iesgn.org.

;; AUTHORITY SECTION:
iesgn.org.		86400	IN	NS	alejandro.iesgn.org.

;; ADDITIONAL SECTION:
correo.iesgn.org.	86400	IN	A	10.0.0.200
alejandro.iesgn.org.	86400	IN	A	10.0.0.1
correo.iesgn.org.	86400	IN	AAAA	fe80::a00:27ff:feb8:7fc4
alejandro.iesgn.org.	86400	IN	AAAA	fe80::a00:27ff:fed9:6fc4

;; Query time: 0 msec
;; SERVER: 10.0.0.1#53(10.0.0.1)
;; WHEN: Thu Nov 21 20:11:25 GMT 2019
;; MSG SIZE  rcvd: 247
~~~

* La dirección IP de www.josedomingo.org

~~~
dig www.josedomingo.org

; <<>> DiG 9.11.5-P4-5.1-Debian <<>> www.josedomingo.org
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 59101
;; flags: qr rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 5, ADDITIONAL: 6

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
; COOKIE: c357db5fd9851eea25ed8e4a5dd6ef7d9cef233729e26b65 (good)
;; QUESTION SECTION:
;www.josedomingo.org.		IN	A

;; ANSWER SECTION:
www.josedomingo.org.	900	IN	CNAME	playerone.josedomingo.org.
playerone.josedomingo.org. 900	IN	A	137.74.161.90

;; AUTHORITY SECTION:
josedomingo.org.	86398	IN	NS	ns5.cdmondns-01.com.
josedomingo.org.	86398	IN	NS	ns3.cdmon.net.
josedomingo.org.	86398	IN	NS	ns2.cdmon.net.
josedomingo.org.	86398	IN	NS	ns4.cdmondns-01.org.
josedomingo.org.	86398	IN	NS	ns1.cdmon.net.

;; ADDITIONAL SECTION:
ns1.cdmon.net.		172798	IN	A	35.189.106.232
ns2.cdmon.net.		172798	IN	A	35.195.57.29
ns3.cdmon.net.		172798	IN	A	35.157.47.125
ns4.cdmondns-01.org.	86398	IN	A	52.58.66.183
ns5.cdmondns-01.com.	172799	IN	A	52.59.146.62

;; Query time: 3226 msec
;; SERVER: 10.0.0.1#53(10.0.0.1)
;; WHEN: Thu Nov 21 20:11:41 GMT 2019
;; MSG SIZE  rcvd: 322
~~~

* Una resolución inversa

~~~
dig -x 10.0.0.201

; <<>> DiG 9.11.5-P4-5.1-Debian <<>> -x 10.0.0.201
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 21638
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 2, ADDITIONAL: 4

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
; COOKIE: 56527d8aff6a60e8100d6afd5dd6ef98b70b63607709fec9 (good)
;; QUESTION SECTION:
;201.0.0.10.in-addr.arpa.	IN	PTR

;; ANSWER SECTION:
201.0.0.10.in-addr.arpa. 86400	IN	PTR	ftp.iesgn.org.

;; AUTHORITY SECTION:
0.0.10.in-addr.arpa.	86400	IN	NS	alejandro.iesgn.org.

;; ADDITIONAL SECTION:
alejandro.iesgn.org.	86400	IN	A	10.0.0.1
alejandro.iesgn.org.	86400	IN	AAAA	fe80::a00:27ff:fed9:6fc4

;; Query time: 1 msec
;; SERVER: 10.0.0.1#53(10.0.0.1)
;; WHEN: Thu Nov 21 20:12:08 GMT 2019
;; MSG SIZE  rcvd: 221
~~~

* La dirección ipv6 de pandora.iesgn.org

~~~
dig AAAA alejandro.iesgn.org

; <<>> DiG 9.11.5-P4-5.1-Debian <<>> AAAA alejandro.iesgn.org
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 63641
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 2, ADDITIONAL: 3

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
; COOKIE: c29d4202d56282708fd392655dd6efb8b18e2fd3039511f8 (good)
;; QUESTION SECTION:
;alejandro.iesgn.org.		IN	AAAA

;; ANSWER SECTION:
alejandro.iesgn.org.	86400	IN	AAAA	fe80::a00:27ff:fed9:6fc4

;; AUTHORITY SECTION:
iesgn.org.		86400	IN	NS	alejandro.iesgn.org.

;; ADDITIONAL SECTION:
alejandro.iesgn.org.	86400	IN	A	10.0.0.1

;; Query time: 1 msec
;; SERVER: 10.0.0.1#53(10.0.0.1)
;; WHEN: Thu Nov 21 20:12:40 GMT 2019
;; MSG SIZE  rcvd: 180
~~~

### Servidor DNS esclavo

#### El servidor DNS actual funciona como DNS maestro. Vamos a instalar un nuevo servidor DNS que va a estar configurado como DNS esclavo del anterior, donde se van a ir copiando periódicamente las zonas del DNS maestro. Suponemos que el nombre del servidor DNS esclavo se va llamar afrodita.iesgn.org

#### *Tarea 4 (3 puntos):* Realiza la instalación del servidor DNS esclavo. Documenta los siguientes apartados

* Entrega la configuración de las zonas del maestro y del esclavo.

##### *En el Servidor Maestro*

###### Añadimos en el fichero */etc/bind/named.conf.options*

~~~
allow-transfer { none; };
~~~

###### Añadimos en el fichero */var/cache/bind/db.iesgn.org* las siguientes lineas

~~~
@               IN      NS      alejandro-slave.iesgn.org.
alejandro-slave IN      A       10.0.0.2
~~~

###### Añadimos en el fichero */var/cache/bind/db.resolucioninversa* las siguientes lineas

~~~
@       IN      NS      alejandro-slave.iesgn.org.
2       IN      PTR     alejandro-slave.iesgn.org.
~~~ 

###### Hay que modificar el fichero */etc/bind/named.conf.local* las siguientes lineas en cada zona

~~~
  allow-transfer { 10.0.0.2; };
  notify yes;

~~~

##### *En el Servidor Esclavo*

###### Añadimos en el fichero */etc/bind/named.conf.options*

~~~
allow-transfer { none; };
~~~

###### Instalamos el paquete de bind
###### Añadimos en el fichero */etc/bind/named.conf.local* lo siguiente

~~~
zone "iesgn.org"
{
  file "db.iesgn.org";
  type slave;
  masters { 10.0.0.1; };
};

zone "0.0.10.in-addr.arpa"
{
  file "db.0.0.10";
  type slave;
  masters { 10.0.0.1; };
}
~~~

* Comprueba si las zonas definidas en el maestro tienen algún error con el comando adecuado.

~~~
sudo named-checkzone iesgn.org /var/cache/bind/db.iesgn.org
  zone iesgn.org/IN: loaded serial 6
  OK

sudo named-checkzone iesgn.org /var/cache/bind/db.0.0.10
  zone 0.0.10.in-addr.arpa"/IN: loaded serial 6
  OK
~~~

* Comprueba si la configuración de named.conf tiene algún error con el comando adecuado.

~~~
vagrant@ServidorDNS1:~$ sudo named-checkconf
vagrant@ServidorDNS1:~$ 
~~~

* Reinicia los servidores y comprueba en los logs si hay algún error. No olvides incrementar el número de serie en el registro SOA si has modificado la zona en el maestro.

* Muestra la salida del log donde se demuestra que se ha realizado la transferencia de zona.

~~~
sudo tail /var/log/syslog 
Nov 21 20:20:43 ServidorMaster named[1775]: zone 0.0.10.in-addr.arpa/IN: loaded serial 5
Nov 21 20:20:43 ServidorMaster named[1775]: zone 0.0.10.in-addr.arpa/IN: sending notifies (serial 5)
Nov 21 20:20:43 ServidorMaster named[1775]: running
Nov 21 20:20:43 ServidorMaster named[1775]: zone iesgn.org/IN: loaded serial 5
Nov 21 20:20:43 ServidorMaster named[1775]: zone iesgn.org/IN: sending notifies (serial 5)
Nov 21 20:20:43 ServidorMaster named[1775]: client @0x7fda40110a10 10.0.0.2#45329 (0.0.10.in-addr.arpa): transfer of '0.0.10.in-addr.arpa/IN': AXFR-style IXFR started (serial 5)
Nov 21 20:20:43 ServidorMaster named[1775]: client @0x7fda40110a10 10.0.0.2#45329 (0.0.10.in-addr.arpa): transfer of '0.0.10.in-addr.arpa/IN': AXFR-style IXFR ended
Nov 21 20:20:43 ServidorMaster named[1775]: managed-keys-zone: Key 20326 for zone . acceptance timer complete: key now trusted
Nov 21 20:20:43 ServidorMaster named[1775]: client @0x7fda40102280 10.0.0.2#41493 (iesgn.org): transfer of 'iesgn.org/IN': AXFR-style IXFR started (serial 5)
Nov 21 20:20:43 ServidorMaster named[1775]: client @0x7fda40102280 10.0.0.2#41493 (iesgn.org): transfer of 'iesgn.org/IN': AXFR-style IXFR ended
~~~

#### *Tarea 5 (1 punto):* Documenta los siguientes apartados

* Configura un cliente para que utilice los dos servidores como servidores DNS.

~~~
sudo cat /etc/resolv.conf
   nameserver 10.0.0.1
   nameserver 10.0.0.2
~~~

* Realiza una consulta con dig tanto al maestro como al esclavo para comprobar que las respuestas son autorizadas. ¿En qué te tienes que fijar?

##### *Master*

~~~
dig @10.0.0.1 ftp.iesgn.org

; <<>> DiG 9.11.5-P4-5.1-Debian <<>> @10.0.0.1 ftp.iesgn.org
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 49907
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 2, ADDITIONAL: 4

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
; COOKIE: aa0ceb5ae636090191bbe8e75dd6f20f593467f75896453b (good)
;; QUESTION SECTION:
;ftp.iesgn.org.			IN	A

;; ANSWER SECTION:
ftp.iesgn.org.		86400	IN	A	10.0.0.201

;; AUTHORITY SECTION:
iesgn.org.		86400	IN	NS	alejandro.iesgn.org.
iesgn.org.		86400	IN	NS	alejandro-slave.iesgn.org.

;; ADDITIONAL SECTION:
alejandro.iesgn.org.	86400	IN	A	10.0.0.1
alejandro-slave.iesgn.org. 86400 IN	A	10.0.0.2
alejandro.iesgn.org.	86400	IN	AAAA	fe80::a00:27ff:fed9:6fc4

;; Query time: 1 msec
;; SERVER: 10.0.0.1#53(10.0.0.1)
;; WHEN: Thu Nov 21 20:22:39 GMT 2019
;; MSG SIZE  rcvd: 200
~~~

##### *Slave*

~~~
dig @10.0.0.2 ftp.iesgn.org

; <<>> DiG 9.11.5-P4-5.1-Debian <<>> @10.0.0.2 ftp.iesgn.org
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 44257
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 2, ADDITIONAL: 4

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
; COOKIE: 69c0d47ca4121ec973850aa45dd6f227d9bb555f4c66dc5c (good)
;; QUESTION SECTION:
;ftp.iesgn.org.			IN	A

;; ANSWER SECTION:
ftp.iesgn.org.		86400	IN	A	10.0.0.201

;; AUTHORITY SECTION:
iesgn.org.		86400	IN	NS	alejandro-slave.iesgn.org.
iesgn.org.		86400	IN	NS	alejandro.iesgn.org.

;; ADDITIONAL SECTION:
alejandro.iesgn.org.	86400	IN	A	10.0.0.1
alejandro-slave.iesgn.org. 86400 IN	A	10.0.0.2
alejandro.iesgn.org.	86400	IN	AAAA	fe80::a00:27ff:fed9:6fc4

;; Query time: 1 msec
;; SERVER: 10.0.0.2#53(10.0.0.2)
;; WHEN: Thu Nov 21 20:23:03 GMT 2019
;; MSG SIZE  rcvd: 200
~~~

###### Te tienes que dependiendo que consulta hagas te sale primer el master o el slave

* Solicita una copia completa de la zona desde el cliente ¿qué tiene que ocurrir?. Solicita una copia completa desde el esclavo ¿qué tiene que ocurrir?

##### *Desde el cliente*

~~~
dig @10.0.0.2 iesgn.org axfr

; <<>> DiG 9.11.5-P4-5.1-Debian <<>> @10.0.0.2 iesgn.org axfr
; (1 server found)
;; global options: +cmd
; Transfer failed.

dig @10.0.0.1 iesgn.org axfr

; <<>> DiG 9.11.5-P4-5.1-Debian <<>> @10.0.0.1 iesgn.org axfr
; (1 server found)
;; global options: +cmd
; Transfer failed.
~~~

###### Desde el cliente no te devuelve nada, porque el cliente no debería de ver la zona completa

##### *Desde el slave*

###### Te muestra todas las zonas completas

~~~
dig @10.0.0.1 iesgn.org axfr

; <<>> DiG 9.11.5-P4-5.1-Debian <<>> @10.0.0.1 iesgn.org axfr
; (1 server found)
;; global options: +cmd
iesgn.org.		86400	IN	SOA	alejandro.iesgn.org. ale95mogra.iesgn.org. 5 604800 86400 2419200 86400
iesgn.org.		86400	IN	NS	alejandro.iesgn.org.
iesgn.org.		86400	IN	NS	alejandro-slave.iesgn.org.
iesgn.org.		86400	IN	MX	10 correo.iesgn.org.
alejandro.iesgn.org.	86400	IN	AAAA	fe80::a00:27ff:fed9:6fc4
alejandro.iesgn.org.	86400	IN	A	10.0.0.1
alejandro-slave.iesgn.org. 86400 IN	A	10.0.0.2
cliente1.iesgn.org.	86400	IN	AAAA	fe80::a00:27ff:fed7:4fc4
cliente1.iesgn.org.	86400	IN	A	10.0.0.202
cliente2.iesgn.org.	86400	IN	AAAA	fe80::a00:27ff:fed3:6fc7
cliente2.iesgn.org.	86400	IN	A	10.0.0.203
correo.iesgn.org.	86400	IN	AAAA	fe80::a00:27ff:feb8:7fc4
correo.iesgn.org.	86400	IN	A	10.0.0.200
departamentos.iesgn.org. 86400	IN	CNAME	web.iesgn.org.
ftp.iesgn.org.		86400	IN	AAAA	fe80::a00:27ff:fec2:6cc4
ftp.iesgn.org.		86400	IN	A	10.0.0.201
informatica.iesgn.org.	86400	IN	NS	alejandro-sub.informatica.iesgn.org.
alejandro-sub.informatica.iesgn.org. 86400 IN A	10.0.0.3
web.iesgn.org.		86400	IN	AAAA	fe80::a00:27ff:feb7:6dc5
web.iesgn.org.		86400	IN	A	10.0.0.204
www.iesgn.org.		86400	IN	CNAME	web.iesgn.org.
iesgn.org.		86400	IN	SOA	alejandro.iesgn.org. ale95mogra.iesgn.org. 5 604800 86400 2419200 86400
;; Query time: 2 msec
;; SERVER: 10.0.0.1#53(10.0.0.1)
;; WHEN: Thu Nov 21 20:28:46 GMT 2019
;; XFR size: 22 records (messages 1, bytes 634)
~~~

#### *Tarea 6 (1 punto):* Muestra al profesor el funcionamiento del DNS esclavo:

* Realiza una consulta desde el cliente y comprueba que servidor está respondiendo.
* Posteriormente apaga el servidor maestro y vuelve a realizar una consulta desde el cliente ¿quién responde?

###### Como se puede ver, aparece despues de parar el servidor DNS Master, el primero en  AUTHORITY SECTION

~~~
vagrant@ClienteDNS:~$ dig ftp.iesgn.org

; <<>> DiG 9.11.5-P4-5.1-Debian <<>> ftp.iesgn.org
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 600
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 2, ADDITIONAL: 4

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
; COOKIE: f3faf6100bf2e9ec89a1c70c5dd5aaf84fdc6510fab76970 (good)
;; QUESTION SECTION:
;ftp.iesgn.org.			IN	A

;; ANSWER SECTION:
ftp.iesgn.org.		604800	IN	A	192.168.100.202

;; AUTHORITY SECTION:
iesgn.org.		604800	IN	NS	alejandro.iesgn.org.
iesgn.org.		604800	IN	NS	alejandro-slave.iesgn.org.

;; ADDITIONAL SECTION:
alejandro.iesgn.org.	604800	IN	A	192.168.100.1
alejandro-slave.iesgn.org. 604800 IN	A	192.168.100.3
alejandro.iesgn.org.	604800	IN	AAAA	fe80::a00:27ff:fe95:4cb9

;; Query time: 1 msec
;; SERVER: 192.168.100.1#53(192.168.100.1)
;; WHEN: Wed Nov 20 21:07:04 GMT 2019
;; MSG SIZE  rcvd: 200
~~~

~~~
vagrant@ServidorDNS1:~$ sudo systemctl stop bind9.service
~~~

~~~
vagrant@ClienteDNS:~$ dig ftp.iesgn.org

; <<>> DiG 9.11.5-P4-5.1-Debian <<>> ftp.iesgn.org
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 52501
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 2, ADDITIONAL: 4

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
; COOKIE: beb100cc6cea2f01ad745ad95dd5ab5d56c96398d6fb7c9d (good)
;; QUESTION SECTION:
;ftp.iesgn.org.			IN	A

;; ANSWER SECTION:
ftp.iesgn.org.		604800	IN	A	192.168.100.202

;; AUTHORITY SECTION:
iesgn.org.		604800	IN	NS	alejandro-slave.iesgn.org.
iesgn.org.		604800	IN	NS	alejandro.iesgn.org.

;; ADDITIONAL SECTION:
alejandro.iesgn.org.	604800	IN	A	192.168.100.1
alejandro-slave.iesgn.org. 604800 IN	A	192.168.100.3
alejandro.iesgn.org.	604800	IN	AAAA	fe80::a00:27ff:fe95:4cb9

;; Query time: 1 msec
;; SERVER: 192.168.100.3#53(192.168.100.3)
;; WHEN: Wed Nov 20 21:08:45 GMT 2019
;; MSG SIZE  rcvd: 200
~~~

### Delegación de dominios

#### Tenemos un servidor DNS que gestiona la zona correspondiente al nombre de dominio iesgn.org, en esta ocasión queremos delegar el subdominio informatica.iesgn.org para que lo gestione otro servidor DNS. Por lo tanto tenemos un escenario con dos servidores DNS:

* pandora.iesgn.org, es servidor DNS autorizado para la zona iesgn.org.
* ns.informatica.iesgn.org, es el servidor DNS para la zona informatica.iesgn.org y, está instalado en otra máquina.

#### Los nombres que vamos a tener en ese subdominio son los siguientes:

* www.informatica.iesgn.org corresponde a un sitio web que está alojado en el servidor web del departamento de informática.
* Vamos a suponer que tenemos un servidor ftp que se llame ftp.informatica.iesgn.org y que está en la misma máquina.
* Vamos a suponer que tenemos un servidor para recibir los correos que se llame correo.informatica.iesgn.org.

#### *Tarea 7 (3 puntos):* Realiza la instalación y configuración del nuevo servidor dns con las características anteriormente señaladas. Muestra el resultado al profesor

##### *Configuración del servidor Maestro*

###### Añadimos al fichero */var/cache/bind/db.iesgn.org * las siguientes lineas

~~~
$ORIGIN informatica.iesgn.org.

@               IN    NS    alejandro-sub
alejandro-sub   IN    A     10.0.0.3
~~~

##### *Configuración del servidor del subdominio*

###### Añadimos en el fichero */etc/bind/named.conf.local* las siguientes lineas

~~~
zone "informatica.iesgn.org"
{
  type master;
  file "db.informatica.iesgn.org";
};
~~~

###### Copiamos el fichero *db.empty* para crear uno nuevo para el subdominio

~~~
sudo cp /etc/bind/db.empty /var/cache/bind/db.informatica.iesgn.org
~~~

###### Dentro de este fichero vamos a añadir las siguientes lineas

~~~
$TTL    86400
@       IN      SOA     alejandro-sub.informatica.iesgn.org. ale95mogra.iesgn.org. (
                              1         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                          86400 )       ; Negative Cache TTL
;
@       IN      NS      alejandro-sub.informatica.iesgn.org.
@       IN      MX  10  correo.informatica.iesgn.org.

$ORIGIN informatica.iesgn.org.

alejandro-sub   IN      A        10.0.0.3
web             IN      A        10.0.0.101
ftp             IN      A        10.0.0.203
correo          IN      A        10.0.0.102
www             IN      CNAME    web
~~~

###### Ahora reiniciamos el servicio

~~~
sudo rndc reload
~~~

#### *Tarea 8 (1 punto):* Realiza las consultas dig/neslookup desde los clientes preguntando por los siguientes:

* Dirección de www.informatica.iesgn.org, ftp.informatica.iesgn.org

~~~
dig www.informatica.iesgn.org

; <<>> DiG 9.11.5-P4-5.1-Debian <<>> www.informatica.iesgn.org
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 37884
;; flags: qr rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 1, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
; COOKIE: 6a10daedacb156136ce6df365dd6f450e7c8952ca9c03b3c (good)
;; QUESTION SECTION:
;www.informatica.iesgn.org.	IN	A

;; ANSWER SECTION:
www.informatica.iesgn.org. 86400 IN	CNAME	web.informatica.iesgn.org.
web.informatica.iesgn.org. 86400 IN	A	10.0.0.101

;; AUTHORITY SECTION:
informatica.iesgn.org.	86400	IN	NS	alejandro-sub.informatica.iesgn.org.

;; Query time: 5 msec
;; SERVER: 10.0.0.1#53(10.0.0.1)
;; WHEN: Thu Nov 21 20:32:16 GMT 2019
;; MSG SIZE  rcvd: 144
~~~

~~~
dig ftp.informatica.iesgn.org

; <<>> DiG 9.11.5-P4-5.1-Debian <<>> ftp.informatica.iesgn.org
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 44141
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
; COOKIE: 34ac55dacced5d4268ba99a65dd6f46599d57a88c579e83d (good)
;; QUESTION SECTION:
;ftp.informatica.iesgn.org.	IN	A

;; ANSWER SECTION:
ftp.informatica.iesgn.org. 86400 IN	A	10.0.0.203

;; AUTHORITY SECTION:
informatica.iesgn.org.	86400	IN	NS	alejandro-sub.informatica.iesgn.org.

;; Query time: 2 msec
;; SERVER: 10.0.0.1#53(10.0.0.1)
;; WHEN: Thu Nov 21 20:32:37 GMT 2019
;; MSG SIZE  rcvd: 126
~~~

* El servidor DNS que tiene configurado la zona del dominio informatica.iesgn.org. ¿Es el mismo que el servidor DNS con autoridad para la zona iesgn.org?

###### No es el mismo

~~~
dig ns informatica.iesgn.org

; <<>> DiG 9.11.5-P4-5.1-Debian <<>> ns informatica.iesgn.org
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 37297
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
; COOKIE: 53f31086b96d3f735e3916c75dd6f4af268f8d105e512d3b (good)
;; QUESTION SECTION:
;informatica.iesgn.org.		IN	NS

;; ANSWER SECTION:
informatica.iesgn.org.	86400	IN	NS	alejandro-sub.informatica.iesgn.org.

;; Query time: 2 msec
;; SERVER: 10.0.0.1#53(10.0.0.1)
;; WHEN: Thu Nov 21 20:33:51 GMT 2019
;; MSG SIZE  rcvd: 106
~~~

* El servidor de correo configurado para informatica.iesgn.org

~~~
dig mx informatica.iesgn.org

; <<>> DiG 9.11.5-P4-5.1-Debian <<>> mx informatica.iesgn.org
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 2440
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
; COOKIE: 8239d7e8c136f3128b3f457c5dd6f4f35c74661f4c153bff (good)
;; QUESTION SECTION:
;informatica.iesgn.org.		IN	MX

;; ANSWER SECTION:
informatica.iesgn.org.	86400	IN	MX	10 correo.informatica.iesgn.org.

;; AUTHORITY SECTION:
informatica.iesgn.org.	86332	IN	NS	alejandro-sub.informatica.iesgn.org.

;; Query time: 4 msec
;; SERVER: 10.0.0.1#53(10.0.0.1)
;; WHEN: Thu Nov 21 20:34:59 GMT 2019
;; MSG SIZE  rcvd: 129
~~~

### DNS dínamico

#### Instala un servidor DHCP que configure de forma automática a los clientes. Este servidor DHCP debe mandar a los clientes los servidores DNS que deben utilizar

#### Configura el servidor DHCP y el DNS maestro para que cada vez que se asigne o modifique una ip a un cliente se actulice de forma automática las zonas del servidor DNS

#### *Tarea 9 (5 puntos):* Documenta en redmine el proceso que has realizado para configurar un DNS dinámico. Muestra un aprueba de funcionamiento

