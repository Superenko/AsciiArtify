# Proof of Concept (PoC): Встановлення та налаштування ArgoCD у кластері k3d

Цей документ містить покрокову інструкцію для розгортання локального Kubernetes кластера за допомогою **k3d** та налаштування GitOps-системи **ArgoCD** з доступом до її графічного інтерфейсу (UI).

![ArgoCD PoC Demo](poc-demo.gif)

> [!NOTE]  
> Дана інструкція підготовлена для демонстрації працездатності концепту (PoC) автоматизації розгортання в межах проекту **AsciiArtify**.

---

## Prerequisites

Перед початком переконайтеся, що на вашому комп'ютері встановлено:
1. **Docker Desktop** або **OrbStack** (запущений та активний).
2. **Homebrew** (для macOS).

---

## Step-by-step Deployment Guide

### Step 1. Installing tools

Install **k3d** (lightweight tool for running k3s in Docker) and **kubectl** (command-line interface for managing Kubernetes):

```bash

brew install k3d


k3d --version
kubectl version --client
```

---

### Крок 2. Створення Kubernetes кластера

```bash
k3d cluster create mycluster -p "8081:80@loadbalancer" -p "8443:443@loadbalancer"
```

* **`8081:80`** — мапінг HTTP-порту для майбутніх додатків.
* **`8443:443`** — мапінг HTTPS-порту для доступу до веб-інтерфейсу ArgoCD.

Після успішного запуску перевірте доступність кластера:
```bash
kubectl get nodes
```

---

### Крок 3. Встановлення ArgoCD

1. Створіть окремий простір імен (Namespace) для ArgoCD:
   ```bash
   kubectl create namespace argocd
   ```

2. Застосуйте офіційний стабільний маніфест для встановлення ArgoCD:
   ```bash
   kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
   ```

3. Зачекайте, поки всі поди перейдуть у статус `Running`:
   ```bash
   kubectl get pods -n argocd -w
   ```

---

### Крок 4. Налаштування доступу до ArgoCD UI

За замовчуванням сервіс `argocd-server` встановлюється як `ClusterIP` (доступний лише всередині кластера). Щоб отримати до нього доступ через прокинутий раніше порт `8443` на вашому Mac, переведіть тип сервісу на `LoadBalancer`:

```bash
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'
```


### Крок 5. Отримання пароля адміністратора (Admin Password)

Початковий пароль для користувача `admin` генерується автоматично під час встановлення та записується в Kubernetes Secret. Отримайте та розкодуйте його за допомогою наступної команди:

```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo
```

> [!WARNING]  
> Цей пароль рекомендується змінити відразу після першого входу в інтерфейс з міркувань безпеки.

---

### Крок 6. Вхід в інтерфейс ArgoCD UI

1. Відкрийте ваш улюблений веб-браузер.
2. Перейдіть за адресою:
    **[https://localhost:8443](https://localhost:8443)** (якщо ви налаштували LoadBalancer на кроці 4)  
    **[https://localhost:8080](https://localhost:8080)** (якщо використовуєте `kubectl port-forward`)
3. Браузер покаже попередження про безпеку (через самопідписаний SSL-сертифікат ArgoCD). Натисніть **Advanced** (Додатково) та оберіть **Proceed to localhost** (Перейти до localhost).
4. У формі входу введіть:
   * **Username:** `admin`
   * **Password:** *(пароль, отриманий на Кроці 5)*

---
