# Laboratorio 3 - Despliegue CI/CD en Kubernetes

**Alumno:** Alejandro Cespedes  
**Repositorio:** janoboxCL/lab3-cicd-kubernetes-alejandro-cespedes  
**Imagen Docker:** janobox/tarea-final:alejandro-cespedes  
**APP_VERSION:** 3.0.0

## Descripción

Este proyecto implementa un flujo CI/CD completo para una aplicación NestJS utilizando:

- Docker
- Docker Hub
- Kubernetes local mediante Docker Desktop
- Jenkins
- Jenkins Kubernetes Plugin
- Agentes dinámicos Kubernetes
- ConfigMap
- Secret
- Deployment
- Service

El pipeline automatiza las etapas:

1. install
2. test
3. build
4. push
5. deploy

## Arquitectura

El flujo implementado es:

GitHub  
→ Jenkins  
→ Agente Kubernetes  
→ instalación de dependencias  
→ tests  
→ build de aplicación  
→ build de imagen Docker  
→ push a Docker Hub  
→ despliegue en Kubernetes

El agente Jenkins se crea dinámicamente como un Pod Kubernetes definido en `agent.yaml`.

## Requisitos

- Docker Desktop
- Kubernetes habilitado en Docker Desktop
- kubectl
- WSL2
- Jenkins
- Kubernetes Plugin para Jenkins
- Cuenta Docker Hub
- Cuenta GitHub

## Recursos Kubernetes

Se utilizaron los siguientes nombres:

- Namespace: `ns-alejandro-cespedes`
- Deployment: `app-alejandro-cespedes`
- Service: `svc-alejandro-cespedes`
- ConfigMap: `config-alejandro-cespedes`
- Secret: `secret-alejandro-cespedes`

El Deployment utiliza dos réplicas.

La imagen desplegada es:

```text
janobox/tarea-final:alejandro-cespedes
```

## Variables de configuración

La aplicación utiliza dos variables de entorno.

Desde ConfigMap:

```text
AMBIENTE=kubernetes
```

Desde Secret:

```text
API_KEY
```

La aplicación permite comprobar ambas mediante:

```text
GET /lab
```

## Construcción manual de la imagen

Desde la raíz del proyecto:

```bash
docker build -t tarea-final:alejandro-cespedes .
```

Etiquetar la imagen:

```bash
docker tag tarea-final:alejandro-cespedes \
  janobox/tarea-final:alejandro-cespedes
```

También se utiliza el tag de versión:

```bash
docker tag tarea-final:alejandro-cespedes \
  janobox/tarea-final:3.0.0
```

Publicar:

```bash
docker push janobox/tarea-final:alejandro-cespedes
docker push janobox/tarea-final:3.0.0
```

## Despliegue manual en Kubernetes

Aplicar los manifiestos:

```bash
kubectl apply -f entrega.yaml
```

Comprobar los Pods:

```bash
kubectl get pods -n ns-alejandro-cespedes
```

Comprobar el Deployment:

```bash
kubectl get deployment app-alejandro-cespedes \
  -n ns-alejandro-cespedes
```

Comprobar el Service:

```bash
kubectl get svc svc-alejandro-cespedes \
  -n ns-alejandro-cespedes
```

## ConfigMap y Secret

Comprobar ConfigMap:

```bash
kubectl get configmap config-alejandro-cespedes \
  -n ns-alejandro-cespedes
```

Comprobar Secret:

```bash
kubectl get secret secret-alejandro-cespedes \
  -n ns-alejandro-cespedes
```

Comprobar variables dentro de la aplicación:

```bash
kubectl exec deployment/app-alejandro-cespedes \
  -n ns-alejandro-cespedes \
  -- printenv
```

## Logs

```bash
kubectl logs deployment/app-alejandro-cespedes \
  -n ns-alejandro-cespedes
```

## Prueba de la aplicación

Ejecutar:

```bash
kubectl port-forward \
  svc/svc-alejandro-cespedes \
  8080:80 \
  -n ns-alejandro-cespedes
```

En otra terminal:

```bash
curl http://localhost:8080/lab
```

Respuesta esperada:

```json
{
  "AMBIENTE": "kubernetes",
  "API_KEY": "lab3-api-key-alejandro"
}
```

## Jenkins

Jenkins se ejecuta dentro del cluster Kubernetes en el namespace:

```text
jenkins
```

El pipeline utiliza un agente Kubernetes definido en:

```text
agent.yaml
```

El agente contiene contenedores especializados para:

- Node.js / pnpm
- Docker
- kubectl
- JNLP de Jenkins

## Credenciales

Las credenciales de Docker Hub se encuentran almacenadas en Jenkins Credentials con el identificador:

```text
dockerhub-credentials
```

Las credenciales no se encuentran escritas directamente en el `Jenkinsfile`.

## Pipeline CI/CD

El `Jenkinsfile` implementa los stages obligatorios:

```text
install
test
build
push
deploy
```

### install

Instala las dependencias utilizando pnpm.

### test

Ejecuta los tests del proyecto con Jest.

### build

Compila la aplicación NestJS y construye la imagen Docker.

### push

Publica las imágenes:

```text
janobox/tarea-final:alejandro-cespedes
janobox/tarea-final:3.0.0
```

en Docker Hub.

### deploy

Ejecuta:

```bash
kubectl apply -f entrega.yaml
```

y posteriormente realiza un restart controlado del Deployment para utilizar la última imagen publicada.

## Verificación del cluster

```bash
kubectl cluster-info
kubectl get nodes
```

## Evidencias

La carpeta `evidencias/` contiene capturas correspondientes a:

- Cluster Kubernetes
- Nodo Kubernetes
- Tests
- Docker build
- Docker Hub
- Pods
- Deployment
- Service
- ConfigMap
- Secret
- Logs
- Variables de entorno
- Port-forward
- Prueba con curl
- Jenkins
- Agente Kubernetes
- Pipeline CI/CD exitoso

El log completo de la ejecución exitosa de Jenkins se encuentra en:

```text
jenkins-pipeline.log
```
