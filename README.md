### PaaS SandBox


---

**1) Install k3s cluster**
```bash

# K3S 클러스터를 생성합니다.
$ curl -sfL https://get.k3s.io | INSTALL_K3S_VERSION=v1.25.9+k3s1 sh 
$ mkdir ~/.kube
$ cp /etc/rancher/k3s/k3s.yaml ~/.kube/config
$ sed -i 's/default/k3s/g'  ~/.kube/config

# Cert-Manager와 Dynamic Storage Class를 생성합니다.
$ kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.10.0/cert-manager.yaml
$ kubectl apply -f https://raw.githubusercontent.com/rancher/local-path-provisioner/master/deploy/local-path-storage.yaml
$ kubectl patch storageclass local-path -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'

# Epinio cli 를 설치합니다.
$ curl -o epinio -L https://github.com/epinio/epinio/releases/download/v1.8.1/epinio-linux-x86_64
$ chmod +x epinio
$ mv epinio /usr/local/bin

```
---

**2) Install epinio**
```bash

# Helm을 이용하여 Epinio를 설치합니다.
$ curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
$ helm repo add epinio https://epinio.github.io/helm-charts
$ helm repo update
$ LB_IP=$(kubectl get svc -n kube-system traefik -o jsonpath={@.status.loadBalancer.ingress} | grep -Eo '[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}')
$ echo $LB_IP
$ helm upgrade --install epinio epinio/epinio --namespace epinio --create-namespace  --set global.domain=$LB_IP.nip.io

# Epinio에 로그인 합니다.
$ epinio login https://epinio.10.214.156.38.nip.io
```
- https://epinio.10.214.156.38.nip.io

- admin / password


---
**3) Run epinio application**

```bash
$ git clone https://github.com/flytux/kw-mvn.git
$ epinio push --name kw-mvn --path .
```

---

**3) Install Acorn**

```bash
$ curl -LO https://cdn.acrn.io/cli/default_linux_amd64_v1/acorn
$ sudo mv acorn /usr/local/bin
$ acorn install
```
---

**4) Deploy Acorn App**
```bash
$ cat << EOF >> Dockerfile
# syntax=docker/dockerfile:1
FROM eclipse-temurin:17-jdk-jammy
WORKDIR /app
COPY .mvn/ .mvn
COPY mvnw pom.xml ./
RUN ./mvnw dependency:resolve
COPY src ./src
CMD ["./mvnw", "spring-boot:run"]
EOF

$ cat << EOF >> Acornfile
containers: {
  app: {
    build: "."
    ports: publish: "8080/http"
  }
}
EOF

$ acorn run .
```
- http://app-red-silence-71c1623a.528l52.alpha.on-acorn.io

---

- Crossplane for controlling cloud infrastuctures from my k8s cluster
