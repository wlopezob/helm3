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