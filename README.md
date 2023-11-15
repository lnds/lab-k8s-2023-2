# Laboratorio Kubernetes

Este laboratorio está basado en la aplicación movies que hemos usado en las tareas 2 y 3.


## Requisitos

Para ejecutar este laboratorio debes tener instalado `mini-kube`.

Encuentra las instrucciones para instalarlo acá: https://minikube.sigs.k8s.io/docs/start/

Además debes tener una cuenta en dockerhub: https://hub.docker.com


# Paso 1

Construye una imagen para `movies-api` y publicala en tu cuenta en `docker-hub`:

```
cd movies-api
docker build -t TU_USUARIO/movies-api .
docker tag TU_USUARIO/movies-api TU_USUARIO/movies-api:v1
docker push TU_USUARIO/movies-api:v1
cd ..
```

Revisa que tu imagen aparece en el sitio de docker-hub.

## Paso 2

Luego construye una imagen para `movies-front` y publícala en tu cuenta en `docker-hub`:

```
cd movies-front
docker build -t TU_USUARIO/movies-front .
docker tag TU_USUARIO/movies-front TU_USUARIO/movies-front:v1
docker push TU_USUARIO/movies-front:v1
cd ..
```

Revisa que tu imagen aparece en el sitio de docker-hub.

## Paso 3

Vamos a iniciar minikube

```
cd k8s
minikube start
```

Abre otro terminal y ejecuta el dashboard:

```
minikube dashboard
```

Este comando abrirá tu navegador y te mostrará el dashboard de k8s.

## Paso 4

Revisa el contenido de la carpeta `k8s/base`.

Encontrarás dos archivos, el primero es `movies-app-namespace.yaml` que contiene lo siguiente:

```
apiVersion: v1
kind: Namespace
metadata:
  name: movies-app
  labels:
    name: movies-app
```

Esto define el namespace `movies-app` que es donde residirán todos los objetos que crearemos.

El otro archivo es `movies-app-storage-class.yaml`, que define un storage class que será usado para crear un volumen persistente para nuestra base de datos posteriormente.

```
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: fast
provisioner: k8s.io/minikube-hostpath
parameters:
  type: pd-ssd
```

Luego para crear estos objetos debes localizarte en el directorio `k8s` y ejecutar:

```
kubectl apply -f base
```

Entra al dashboard y selecciona el namespace `movies-app` en el combobox. Haz click en `Storage Classes` (debajo de `Config and Storage`) y verifica que aparece `fast` y `standard`.

También puedes obtener esta información en la consola del siguiente modo:

```
kubectl get storageclass --namespace=movies-app
```

Si no quieres tener que escribir el namespace en cada comando `kubectl` puedes cambiar el contexto así:

```
kubectl config set-context --current --namespace=movies-app
```


## Paso 5

Ahora vamos a crear la base postgres, esta se define en los archivos bajo la carpeta `postgres`.

El archivo `postgres-secrets.yaml` contiene la clave usada en la base de datos, que se encuentra codificada en base64, que es la forma que tiene K8s para codificar los secrets. El valor que está almacenado se puede cambiar, siempre que lo codifiques en base64.

El archivo `postgres-service` define el servicio postgres, que será usado posteriormente por migrations y los servicios. Acá se define el port que usará el servicio, entre otros parámetros.

Por último el archivo `postgres-statefullset.yaml` crea un `StatefulSet` que es la forma de crear objetos persistentes, en particular, acá configuramos los volúmenes persistentes donde quedará la base de datos.

Aplicamos estos objetos con el siguiente comando:

```
kubectl apply -f postgres
```

## Paso 6

Vamos a ejecutar la migración de datos con flyway usando un Job.

El job ya se encuentra configurado en el archivo `migrations/migration-job.yaml`. Este archivo requiere un objeto `config-map` que vamos a crear con el siguiente comando:

```
kubectl create configmap migration-config --from-file=../sql_migrations -o yaml --namespace=movies-app --dry-run=client > migration/migration-config-map.yaml
```


Este comando va a crear el archivo `migration/migration-config-map.yaml`. 

Revisa este archivo.
Fíjate como hemos "copiado" el contenido de los archivos de migración flyway dentro de la sección `data:` de este archivo.

Aplicaremos la migración ejecutando el siguiente comando:

```
kubectl apply -f migration
```

Después de esto tenemos nuestra base de datos cargada con los datos iniciales.

Puedes verificarlo "entrando" a la base postgres. En el dashbaord selecciona `Pods` en el extremo derecho del Pods `postgres-0` aparecen tres puntos verticales, este es un menú, ahi selecciona la opción `Exec`, esto abre un shell en el pod y puedes ingresar a la base de datos haciendo:

```
psql -U postgres
```

Entrando en la base de datos puedes realizar consultas para ver los datos que poblamos con la migración.

Puedes hacer lo mismo en la linea de comandos del siguiente modo:

```
kubectl exec --stdin --tty  postgres-0 -- /bin/bash
root@posrgres-0:/# psql -U postgres
```


## Paso 7

Ahora vamos a crear los deployments para nuestros dos imágenes que creamos en el paso 1.

Debes modificar los archivos `k8s/movies-api-deployment.yaml` y `k8s/movies-front-deployment.yaml`.

Busca la linea que contiene el texto: `image: TU_USUARIO/movies-front:v1` en `k8s/movies-front-deployment.yaml` y reemplaza TU_USUARIO por el nombre de tu usuario en docker-hub.


Busca la linea que contiene el texto: `image: TU_USUARIO/movies-api:v1` en `k8s/movies-api-deployment.yaml` y reemplaza TU_USUARIO por el nombre de tu usuario en docker-hub.

Luego ejecuta:

```
kubectl apply -f deployment
```


Para probar debemos habilitar `INGRESS` del siguiente modo:

```
minikube addons enable ingress
```

Luego ejecuta:

```
minikube tunnel
```

Con esto habilitas el tunnel para exponer los servicios a través de la dirección IP `127.0.0.1`.

Si todo sale bien deberías poder acceder a la aplicación en tu navegador en la dirección `http://localhost:8080`.
