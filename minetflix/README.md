# Tutoriales HELM3 - Minetflix
## Requisitos
> Instalaremos una instancia EC2 con microk8s mediante cloudformation, los pasos se encuentran en [Cloudformation-ec2-microk8s](https://github.com/wlopezob/cloudformation/tree/main/microk8s)

## Crear la imagen docker
```
git clone https://github.com/wlopezob/minetflix.git && cd minetflix
```
#### creamos la imagen localhost:32000/minetflix para hacer un push al addon registry de microk8s 
```
docker build -t localhost:32000/minetflix:latest .
docker push localhost:32000/minetflix:latest
```
#### creamos la carpeta que contendra nuestros charts y creamos nuestro chart
```
mkdir charts && cd charts
microk8s.helm3 create minetflix-chart
```

#### Observamos la estructura de las carpetas creadas
```
tree

└── minetflix-chart
    ├── charts
    ├── Chart.yaml
    ├── templates
    │   ├── deployment.yaml
    │   ├── _helpers.tpl
    │   ├── ingress.yaml
    │   ├── NOTES.txt
    │   ├── serviceaccount.yaml
    │   ├── service.yaml
    │   └── tests
    │       └── test-connection.yaml
    └── values.yaml

```
#### El archivo values.yaml contiene todos las variables y valores de nuestras plantillas
#### La carpeta templates contiene las plantillas de nuesta aplicacion
#### Modificamos el archivo deployment.yaml, para este ejemplo modificamos el nombre de la imagen y quitando **:{{ .Chart.AppVersion }}** para poder controlar nuestras versiones(se podria automatizar las versiones, queda pendiente este tema)
```
vim minetflix-chart/templates/deployment.yaml

      containers:
        - name: {{ .Chart.Name }}
          securityContext:
            {{- toYaml .Values.securityContext | nindent 12 }}
          image: "{{ .Values.image.repository }}"
```
#### Modificamos el nombre de nuestra imagen del archio value.yaml
```
vim minetflix-chart/values.yaml

image:
  repository: localhost:32000/minetflix:latest
  pullPolicy: IfNotPresent


```
#### tambien habilitamos la creacion del ingress, enabled: true
```
ingress:
  enabled: true
```