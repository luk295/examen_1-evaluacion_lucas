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
### 9.1 Primero creo as carpetas para a zona e para as configuracións.































