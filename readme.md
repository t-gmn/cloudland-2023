# Installation und Konfiguration von k3d

## Voraussetzungen: 
- docker ist installiert 
- (git) bash
- Internet

# Deployment step by step
## Step 0 - Docker

### Container starten
```bash 
docker run -d nginx:1.24.0
```
### Laufenden Container anzeigen
```bash 
docker ps
```
### In Container gehen
- Alle OS außer Windows:
```bash 
docker exec –it <container-id> /bin/sh
```
- Windows in git bash:
```bash 
winpty docker exec –it <container-id> sh
```
### Anwendungscrash simulieren, also nginx abschießen
```bash 
kill 1 #inside container
```

## Installation von [k3d](https://k3d.io/v5.5.1/)

```bash
curl -s https://raw.githubusercontent.com/k3d-io/k3d/main/install.sh | bash
```
## Prüfen, ob k3d installiert ist
```bash
k3d version
```
## k3d cluster erstellen: 
```bash
k3d cluster create cloudland-cluster --agents 2
```
## Nodes anzeigen: 
```bash
k3d node list
```

## kubectl installieren (wenn nötig): 
### Prüfen, ob kubectl vorhanden ist: 
```bash
kubectl version
```
### Wenn nicht - installieren
- [Windows](https://kubernetes.io/docs/tasks/tools/install-kubectl-windows/)
- [MacOs](https://kubernetes.io/docs/tasks/tools/install-kubectl-macos/)
- [Linux](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/)

### Prüfen, ob kubectl den cluster kennt
```bash
kubectl cluster-info
```

## Step 1 - Pod

### Pod erzeugen

```bash 
kubectl apply -f ./step1/step1-pod-nginx.yml 
```

###  Pods beobachten
```bash
kubectl get pods -w 
```

### Details zu einem Pod anzeigen
```bash
kubectl describe pod <Podname> 
```

### Logs anzeigen
```bash
kubectl logs -f <Podname> 
```

### In Container gehen
- Alle OS außer Windows:
```bash 
kubectl exec –it <Podname> -- /bin/sh
```
- Windows in git bash:
```bash 
winpty kubectl exec –it <Podname> -- sh
```

### Anwendungscrash simulieren, also nginx abschießen
```bash 
kill 1 #inside container
```

### Pod entfernen

```bash 
kubectl delete -f ./step1/step1-pod-nginx.yml 
```

## Step 2 - Deployment

### Deployment erstellen
```bash 
kubectl apply -f ./step2/step2-deployment.yml 
```
### Zusehen, wie drei Pods hochgefahren werden

```bash 
kubectl get pods -w 
```

### Hidden Champion: ReplicaSet

```bash 
kubectl get replicasets
```

### ReplicaSet ausgeben lassen

```bash
kubectl describe rs ngi
```

### Rolling-Update

im deployment:
```
image: nginx:1.25.0
```

## Step 3 - Service

### Ausgangssituation: 

```bash
curl localhost:80  # geht nicht
```
### Cluster neu konfigurieren
```bash
k3d cluster delete cloudland-cluster
k3d cluster create cloudland-cluster --agents 2 -p "8081:80@loadbalancer"
```
### Service erzeugen
```bash
kubectl apply -f step3/step3-service.yml
```
### Ingress erzeugen
```bash
kubectl apply -f step3/step3-ingress.yml
```
### Anwendung aufrufen
```bash
curl localhost:8081 #geht
```