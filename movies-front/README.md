# Pasos para publicar esta imagen

Debes tener una cuenta en docker-hub. Puedes usar tu cuenta GitHub para registrarte.

Haz login desde la consola con este comando:

```
docker login
```

Luego haz un build de `movies-front` usando este comando:

```
docker build --no-cache -t USER/movies-front:v0.1.0 .
```

Reemplaza `USER` por tu nombre de usuario en docker-hub.

Ahora publica la imagen con este comando:

```
docker push USER/movies-front:v0.1.0
```
 

Verifica que la imagen queda publicada en tu cuenta en docker-hub.
