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
#### habilitamos la creacion del ingress y añadimos el path "/"
```
ingress:
  enabled: true
  annotations: {}
    # kubernetes.io/ingress.class: nginx
    # kubernetes.io/tls-acme: "true"
  hosts:
    - host: chart-example.local
      paths: ["/"]

```
#### Modificamos el template del ingress para ingresar al app mediante el path "/" y no ingresar mediante dominio, para ello comentamos
#### - host: {{ .host | quote }} y añadimos el "-" al http, quedando: 
```
vim minetflix-chart/templates/ingress.yaml
 
  rules:
  {{- range .Values.ingress.hosts }}
  #- host: {{ .host | quote }}
    - http:
        paths:
        {{- range .paths }}
          - path: {{ . }}
            backend:
              serviceName: {{ $fullName }}
              servicePort: {{ $svcPort }}
        {{- end }}
  {{- end }}

```
#### Instalamos nuestra app como vista preliminar y observar los manifiestos, si el app ya existe en vez de install podemos usar  upgrade
```
microk8s.helm3 install --dry-run --debug minetflix-chart minetflix-chart/
```
#### Instalamos nuestra app con el nombre de  minetflix-chart
```
microk8s.helm3 install minetflix-chart minetflix-chart/
```
#### Observamos el listado de todos los recursos creados
```
microk8s.kubectl get all

NAME                                   READY   STATUS    RESTARTS   AGE
pod/minetflix-chart-599c864dcf-tk4h5   1/1     Running   0          3m10s

NAME                      TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
service/kubernetes        ClusterIP   10.152.183.1    <none>        443/TCP   3h54m
service/minetflix-chart   ClusterIP   10.152.183.26   <none>        80/TCP    3m10s

NAME                              READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/minetflix-chart   1/1     1            1           3m10s

NAME                                         DESIRED   CURRENT   READY   AGE
replicaset.apps/minetflix-chart-599c864dcf   1         1         1       3m10s

``` 
#### Listamos el ingress
```
microk8s.kubectl get ingress

NAME              CLASS    HOSTS   ADDRESS     PORTS   AGE
minetflix-chart   public   *       127.0.0.1   80      22m

```

#### Probando el app minetflix
```
curl -kL http://{miippublica}/
```
