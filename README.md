# solar-system-app
# Розгортання Harbor, Minikube, Argo CD та додатку Solar System

## 1. Розгортання Harbor
1. Встановіть Docker та Docker Compose:
    ```bash
    sudo apt-get update
    sudo apt-get install apt-transport-https ca-certificates curl software-properties-common
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
    echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
    sudo apt-get update
    sudo apt-get install docker-ce docker-ce-cli containerd.io
    sudo usermod -aG docker ${USER}
    su - ${USER}
    ```

2. Встановіть Docker Compose:
    ```bash
    sudo curl -L "https://github.com/docker/compose/releases/download/v2.3.3/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
    sudo chmod +x /usr/local/bin/docker-compose
    ```

3. Встановіть Harbor:
    ```bash
    wget https://github.com/goharbor/harbor/releases/download/v2.8.0/harbor-online-installer-v2.8.0.tgz
    tar xvf harbor-online-installer-v2.8.0.tgz
    cd harbor
    sudo ./install.sh
    ```

4. Налаштування сертифікатів та запуск:
    ```bash
    sudo apt-get install openssl
    openssl genrsa -out harbor.key 2048
    openssl req -new -x509 -key harbor.key -out harbor.crt -days 365
    ```

## 2. Завантаження Docker образу у Harbor
1. Завантажте Docker образ `solar-system` у Harbor:
    ```bash
    docker login localhost
    docker pull siddharth67/solar-system:v9
    docker tag siddharth67/solar-system:v9 192.168.31.68/library/solar-system:v9
    docker push 192.168.31.68/library/solar-system:v9
    ```

## 3. Розгортання Minikube з 3 нодами
1. Встановіть Minikube та Kubectl:
    ```bash
    curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
    sudo install minikube-linux-amd64 /usr/local/bin/minikube
    curl -LO "https://dl.k8s.io/release/v1.28.0/bin/linux/amd64/kubectl"
    chmod +x kubectl
    sudo mv kubectl /usr/local/bin/
    ```

2. Розгорніть кластер:
    ```bash
    minikube start --nodes=3 --driver=docker
    ```

## 4. Розгортання Argo CD
1. Встановіть Argo CD:
    ```bash
    kubectl create namespace argocd
    kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
    kubectl port-forward svc/argocd-server -n argocd 8080:443
    ```

2. Отримайте початковий пароль:
    ```bash
    kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}" | base64 -d
    ```

3. Зайдіть в Argo CD за посиланням [https://localhost:8080](https://localhost:8080).

## 5. Створення Helm-чарту та налаштування Argo CD
1. Створіть Helm-чарт у GitHub:
    ```bash
    helm create solar-system
    git add solar-system/
    git commit -m "Add Helm chart for solar-system app"
    git push origin main
    ```

2. Створіть namespaces у кластері:
    ```bash
    kubectl create namespace dev
    kubectl create namespace prod
    ```

3. Налаштуйте Argo CD для розгортання додатку:
    - **App Name**: `solar-system-dev`
    - **Project**: default
    - **Repository URL**: https://github.com/ILyakhova/solar-system-app
    - **Branch**: main
    - **Path**: `solar-system`
    - **Namespace**: `dev`

4. Проксі для перевірки додатку:
    ```bash
    kubectl port-forward svc/solar-system-dev 8081:80 -n dev
    ```

Перейдіть на [http://localhost:8081](http://localhost:8081) для перевірки додатку.
