Title: Asignación de un dominio personal a GCP API Gateway
Date: 2022-04-22 15:50
Category: Cloud
Tags: google,gcp,api,gateway,dns
Slug: gcp-dominio-api-gateway
Authors: José L. Patiño-Andrés
Summary: Explicación de cómo asignar un nombre de dominio personalizado a una instancia de GCP API Gateway

[GCP API Gateway](https://cloud.google.com/api-gateway/docs) es un producto de
Google Cloud que ofrece un servicio de gateway que podemos utilizar como punto
de entrada único a todas las APIs de nuestros microservicios desplegados en GCP.

Una vez creado y configurado, GCP API Gateway se hace disponible a través de una
URL generada automáticamente por Google, como por ejemplo
`https://megacorp-api-gateway-ayxsu52z.ew.gateway.dev`. Esto puede no ser muy
conveniente si queremos ofrecer nuestra Gateway al público, probablemente bajo
un dominio de nuestra propiedad como podría ser simplemente
`https://api.megacorp.com`.

Esta referencia indica cómo podemos hacer ésto a través de la herramienta de
consola de GCP `gcloud`.

## 1. Generar el certificado SSL para para nuestro dominio

```bash
gcloud compute ssl-certificates create <NOMBRE_CERTIFICADO> \
       --description="Megacorp API Gateway SSL certificate" \
       --domains="gateway.megacorp.com" \
       --global
```

Donde `NOMBRE_CERTIFICADO` es cualquier nombre que queramos, por ejemplo 
`megacorp-gateway-ssl-certificate`.

Podemos comprobar que el certificado ha sido creado con el siguiente comando:

```bash
gcloud compute ssl-certificates list --global
```

Nótese que en principio el certificado aparecerá con la marca `PROVISIONING`.
Ésto indica que GCP está aprovisionando el certificado y **todavía no está
disponible**. Téngase ésto en cuenta en caso de terminar el tutorial y recibir
errores del tipo
`curl: (35) error:14094410:SSL routines:ssl3_read_bytes:sslv3 alert handshake failure`
en las primeras pruebas. En ese caso, habrá que esperar un tiempo (normalmente
unos minutos, pero puede ser desde media hora a varias horas) a que se
sincronice.

## 2. Crear el NEG para el API Gateway

El [NEG](https://cloud.google.com/load-balancing/docs/negs) (Network Endpoint 
Group / grupo de extremos de red) es un objeto interno de configuración de
Google Cloud que especifica un grupo de servicios de Backend. Necesitamos crear
uno para nuestra API Gateway:

```bash
gcloud beta compute network-endpoint-groups create <NOMBRE_NEG> \
       --region=europe-west1 \
       --network-endpoint-type=serverless \
       --serverless-deployment-platform=apigateway.googleapis.com \
       --serverless-deployment-resource=<ID_API_GATEWAY>
```

Donde:
- `NOMBRE_NEG` es cualquier nombre que queramos, por ejemplo
`megacorp-api-gateway-neg`.
- `ID_API_GATEWAY` es el ID del Gateway que hayamos asignado en GCP y que
podemos encontrar en el GCP Dashboard de API Gateway, seleccionando la API
que queremos y luego la pestaña "GATEWAYS" y viendo los detalles del Gateway si
es necesario. Por ejemplo ésto podría ser `megacorp-api-gateway`.

## 3. Crear el Servicio de Backend

Un [Backend Service](https://cloud.google.com/load-balancing/docs/backend-service)
o servicio de backend es otro objeto de configuración de GCP que define la
distribución de tráfico de red para Cloud Load Balancing.

Primero debemos crear un Backend Service, y luego agregarle un Backend.

### Crear Backend Service

```bash
gcloud compute backend-services create <NOMBRE_BACKEND_SERVICE> --global
```

Donde de nuevo `NOMBRE_BACKEND_SERVICE` puede ser lo que nosotros queramos. Por
ejemplo `megacorp-api-gateway-backend-service`.

### Agregar nuevo Backend

```bash
gcloud compute backend-services add-backend <NOMBRE_BACKEND_SERVICE> \
       --global \
       --network-endpoint-group=<NOMBRE_NEG> \
       --network-endpoint-group-region=europe-west1
```

Donde:
- `NOMBRE_BACKEND_SERVICE` es el nombre que le hayamos dado antes al Backend 
  Service, en nuestro caso el ejemplo era `megacorp-api-gateway-backend-service`.
- `NOMBRE_NEG` es el nombre que le habíamos asignado antes al NEG, en nuestro
  caso el ejemplo era `megacorp-api-gateway-neg`.

## 4. Crear las URL Maps

[URL Maps](https://cloud.google.com/load-balancing/docs/url-map-concepts) es de
nuevo un objeto de configuración interno de GCP que enruta las peticiones
enrtrantes hacia los correspondientes servicios de Backend.

```bash
gcloud compute url-maps create <NOMBRE_URL_MAP> \
       --default-service=<NOMBRE_BACKEND_SERVICE>
```

Donde:
- `NOMBRE_URL_MAP` puede ser el nombre que queramos, por ejemplo
  `megacorp-api-gateway-url-map`.
- `NOMBRE_BACKEND_SERVICE` es el nombre que le hayamos dado antes al Backend 
  Service, en nuestro caso el ejemplo era `megacorp-api-gateway-backend-service`.

## 5. Crear el proxy HTTPS

Puesto que vamos a utilizar HTTPS con certificados (punto 1 de esta guía), hay
que crear el [Proxy](https://cloud.google.com/load-balancing/docs/target-proxies) 
HTTPS necesario.

```bash
gcloud compute target-https-proxies create <NOMBRE_PROXY> \
       --ssl-certificates=<NOMBRE_CERTIFICADO> \
       --url-map=<NOMBRE_URL_MAP>
```

Donde:
- `<NOMBRE_PROXY>` puede ser el nombre que queramos, por ejemplo
  `megacorp-api-gateway-https-proxy`.
- `NOMBRE_CERTIFICADO` es el nombre que le hayamos dado al certificado SSL. En
  nuestro ejemplo era `megacorp-gateway-ssl-certificate`. 
- `<NOMBRE_URL_MAP>` es el nombre que le hayamos dado al URL Map. En nuestro
  ejemplo era `megacorp-api-gateway-url-map`.

## 6. Crear reglas de reenvío/filtrado

Al Proxy HTTPS hay que asignarle reglas paras que entienda qué debe hacer con el
tráfico entrante. En nuestro caso para API Gateway, inicialmente, sólo se
necesita una:

```bash
gcloud compute forwarding-rules create <NOMBRE_REGLA> \
       --target-https-proxy=<NOMBRE_PROXY> \
       --global \
       --ports=443
```

Donde:
- `<NOMBRE_REGLA>` puede ser el nombre que queramos, por ejemplo
  `megacorp-api-gateway-forwarding-rule`.
- `<NOMBRE_PROXY>` es el nombre que le hayamos dado al Proxy HTTPS. En nuestro
  ejemplo era `megacorp-api-gateway-https-proxy`.

### Comprobar la creación de la regla

Comprobamos que la regla se ha creado correctamente con el siguiente comand:

```bash
gcloud compute forwarding-rules list
```

Esto nos debería dar una salida que para nuestro ejemplo en esta guía debería
ser así:

```bash
NAME                                     REGION        IP_ADDRESS      IP_PROTOCOL  TARGET
megacorp-api-gateway-forwarding-rule                   34.123.123.123  TCP          megacorp-api-gateway-https-proxy
```

Es importante anotar la dirección IP, `34.123.123.123`, porque se necesita para
el paso final.

## 7. Configurar GCP DNS Records

Éste último paso debemos hacerlo en GCP Dashboard. Se trata de ir a "Network
Services", "Cloud DNS", seleccionar la zona donde se encuentra alojado el
dominio que queremos usar para GCP API Gateway y configurar dicho dominio
**añadiendo la IP señalada**.

Una vez finalizado este proceso, se debería poder acceder a nuestra API Gateway
sin problemas a través de `https://gateway.megacorp.com`.
