### 1. Explica métodos para 'abrir' unha consola/shell a un contenedor en execución.

`Facendo docker attach + a máquina.`
`facendo docker exec -it [nome da maquina] sh`

### 2. No contenedor anterior (en execución), qué opciones tes que ter usado ó arrincalo para poder interactuar coas súas entradas e salidas 

tty:true

### 3. Cómo sería un ficheiro docker-compose para que dous contenedores se comuniquen entre si nunha rede só deles?

Para se crea unha rede na que se vexan os dous contenedores, hay que crear o arquivo desta maneira:
```
services:
	maquina1:
		configuracións...
		
		networks:
			*nome_rede*:
				ipv4_address: dirección IP  [.01


	maquina 2:
		configuracións...
		
		networks:
			*nome_rede*:
				ipv4_address: dirección ip [.02]

networks:
	*nome_rede*:
	  driver: bridge
	  ipam:
	   driver:default
	   config:
	   	-subnet: a rede
	   	-ip_range: o rango das ip dentro da rede
	   	-gateway: dirección hacia o ruter

```

En services "créanse" os contenedores, nos que se especifica a rede que van a usar ambos. Neste caso se lles indica que usen a mesma rede que creo no plano xeneral da xerarquía, chamada networks.

Nesta "NETWORKS" situada no nivel superior se crea a rede que van a compartir ambos contenedores: nome,configuración, ips, modo (bridge) etc... 

O docker compose crea a rede e logo se lles asigna individualmente a cada contenedor.

O driver bridge indica adaptador ponte. Como outra máquina virtual.



### 4. Qué tes que engadir ó ficheiro anterior para que un contenedor teña unha IP fixa?
```
		networks:
			*nome_rede*:
				ipv4_address: 192.168.0.1
```
Neste extracto do arquivo asígnolle unha dirección fixa coa IPv4 192.168.0.1


### 5. Qué comando de consola podo usar para sabe-las ips dos contenedores anteriores?

*docker network inspect nome_rede*



### 6. Cál é a funcionalidade do apartado "ports" en docker compose?

Establecer a dirección de comunicación entre as máquinas virtuais e a máquina local

Exemplo:
```
	55:53/upd 
	para que a máquina local escoite no porto 55 e a virtual no 53.
```
Unha das vantaxes do docker é que cada contenedor pode escoitar no seu respectivo porto neste caso o 53. O que sifnifica que no mesmo porto da máquina local podo controlar varias máquinas en distintos portos. 


### 7. Para qué serve o rexistro CNAME? Pon un exemplo

O rexisttro CNAME é como un apodo dado ao alias dunha IP. Úsase para asociar novos subdominios con domininios xa existentes de rexistro no A.

### 8. Cómo podo facer para que a configuración dun contenedor DNS non se borre se creo outro contenedor?

Podes parar o contenedor con docker stop ou mellor gardar a configuración nos arquivos locais e logo montar as carpetas cos seus volumes ao iniciar o docker compse.


### 9. Engade unha zoa tendaelectronica.int no teu docker DNS que teña:
```
- www á IP 172.16.0.1
- owncloud sexa un CNAME de www
- un rexistro de texto có contido "1234ASDF"
Comproba que todo funciona có comando "dig"
Mostra nos logs que o servicio funciona ben usando a saída da terminal ó levantar o compose ou có comando "docker logs [nomeContenedorOuID]"
(o apartado 9 realízase na máquina virtual)
```
### Primero creo as carpetas para a zona e para as configuracións.

### 9.1, 9.2, 9.3 Creo o docker-compose e os ficheiros de zona e configuracións .

`docker compose:`
```
services:
  bind:
    image: internetsystemsconsortium/bind9:9.18
    container_name: dns_examen

    tty: true
    ports:
      - 57:53/udp
      - 57:53/tcp

    volumes:

      - ./configuracion:/etc/bind/
      - ./zonas:/var/lib/bind/
    
    networks:
      primeira_evaluacion:
        ipv4_address: 172.16.0.1
      
  cliente:
    image: alpine  
    container_name: cliente_examen
    tty: true
    
    dns:
      - 172.16.0.1
    networks:
      primeira_evaluacion:
        ipv4_address: 172.16.0.250

networks:
  primeira_evaluacion:
    driver: bridge
    ipam:
      driver: default
      config:
        - subnet: 172.16.0.0/16
          ip_range: 172.16.0.0/24
          gateway: 172.16.0.254


```

`zonas`:
```
 $TTL 38400	; 10 hours 40 minutes
@		IN SOA	db.tendaelectronica.int. some.email.address. (
				10000002   ; serial
				10800      ; refresh (3 hours)
				3600       ; retry (1 hour)
				604800     ; expire (1 week)
				38400      ; minimum (10 hours 40 minutes)
				)

www		IN A		172.16.0.1
texto   	IN TXT          "1234ASDF"
#alias	IN CNAME		db.tendaelectronica.int.

```

`configuracion`:
```
zone "db.tendaelectronica.int." {
        type master;
        file "/var/lib/bind/db.tendaelectronica.int.";
        allow-query {
                any;
                };
        };

options {
        directory "/var/cache/bind";

        forwarders {
                8.8.8.8;
                1.1.1.1;
         };
         forward only;

        listen-on { any; };
        listen-on-v6 { any; };

        allow-query {
                any;
        };
};

```


### Vexo a creación da rede e que os contenedores están no mesma rede:

`docker network inspect examen_1-evaluacion_lucas_primeira_evaluacion` 

```
[
    {
        "Name": "examen_1-evaluacion_lucas_primeira_evaluacion",
        "Id": "2bc7e08042266c999313427026deb1fd36ff9bdcbe4c96d6613c70a2f69c873c",
        "Created": "2024-11-15T19:38:26.1697876+01:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "172.16.0.0/16",
                    "IPRange": "172.16.0.0/24",
                    "Gateway": "172.16.0.254"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {
            "13c9813c9f510d9a93ddfc0be571208e9b5c3c19314fffefba270796fecac207": {
                "Name": "cliente_examen",
                "EndpointID": "c761d5b6fae71501b318a8df2e93dc251cacfb20196cf6f9bb4c4da2ca1f8717",
                "MacAddress": "02:42:ac:10:00:fa",
                "IPv4Address": "172.16.0.250/16",
                "IPv6Address": ""
            },
            "afd10d3fb5e077c94d50d16b0888a75481ae0fea17890c771b6e9bf0a43ac70a": {
                "Name": "dns_examen",
                "EndpointID": "855270736f64e4ca7e3397cab7fa4964bc7ce6c046169ba8973704f37341a4ff",
                "MacAddress": "02:42:ac:10:00:01",
                "IPv4Address": "172.16.0.1/16",
                "IPv6Address": ""
            }
        },
        "Options": {},
        "Labels": {
            "com.docker.compose.network": "primeira_evaluacion",
            "com.docker.compose.project": "examen_1-evaluacion_lucas",
            "com.docker.compose.version": "2.29.2"
        }
    }
]
```





### Entro no cliente para ver que funciona. 
`docker exec -it cliente_examen sh`

Fago `ip a` :
```
luk@luk-VirtualBox:~/examen_1-evaluacion_lucas$ docker exec -it cliente_examen sh
/ # ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
27: eth0@if28: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue state UP 
    link/ether 02:42:ac:10:00:fa brd ff:ff:ff:ff:ff:ff
    inet 172.16.0.250/16 brd 172.16.255.255 scope global eth0
       valid_lft forever preferred_lft forever
/ # 

```


### Probo que vense entre eles:

```
/ # ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
27: eth0@if28: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue state UP 
    link/ether 02:42:ac:10:00:fa brd ff:ff:ff:ff:ff:ff
    inet 172.16.0.250/16 brd 172.16.255.255 scope global eth0
       valid_lft forever preferred_lft forever
/ # ping 172.16.0.1
PING 172.16.0.1 (172.16.0.1): 56 data bytes
64 bytes from 172.16.0.1: seq=0 ttl=64 time=0.290 ms
64 bytes from 172.16.0.1: seq=1 ttl=64 time=0.221 ms
64 bytes from 172.16.0.1: seq=2 ttl=64 time=0.142 ms
^C
--- 172.16.0.1 ping statistics ---
3 packets transmitted, 3 packets received, 0% packet loss
round-trip min/avg/max = 0.142/0.217/0.290 ms
/ # 
```

### Para elo primero debo instalar as bind-tools:

`apk update && apk add bind-tools`

Non me responde. Así que modifico a rede agregando a dirección de google só para instalar e actualizar (momentaneamente):

```
  cliente:
    image: alpine  
    container_name: cliente_examen
    tty: true
    
    dns:
      - 8.8.8.8 
      - 172.16.0.1
      


```

### Comprobo facendo dig

```
/ # dig @172.16.0.1

; <<>> DiG 9.18.27 <<>> @172.16.0.1
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: SERVFAIL, id: 58639
;; flags: qr rd ra; QUERY: 1, ANSWER: 0, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 00b93c77dbf010f70100000067379dee668c105637aef180 (good)
;; QUESTION SECTION:
;.				IN	NS

;; Query time: 20 msec
;; SERVER: 172.16.0.1#53(172.16.0.1) (UDP)
;; WHEN: Fri Nov 15 19:15:58 UTC 2024
;; MSG SIZE  rcvd: 56

/ # 

```






















