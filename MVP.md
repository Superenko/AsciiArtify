# MVP: Розгортання go-demo-app через ArgoCD

> Демонстрація повного GitOps-циклу: ArgoCD автоматично відслідковує репозиторій
> [go-demo-app](https://github.com/den-vasyliev/go-demo-app) та розгортає зміни на Kubernetes кластері.

---

## 1. Налаштування ArgoCD Application

### 1.1 Створити namespace

```bash
kubectl create namespace demo
```

### 1.2 Задеплоїти Application маніфест

```bash
kubectl apply -f - <<EOF
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: go-demo-app
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/den-vasyliev/go-demo-app
    targetRevision: HEAD
    path: helm
    helm:
      parameters:
        - name: api-gateway.image.tag
          value: "1.14.2"
  destination:
    server: https://kubernetes.default.svc
    namespace: demo
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
      - CreateNamespace=true
EOF
```

### 1.3 Перевірити стан

```bash
# Стежити за запуском podів
kubectl get pods -n demo -w

# Перевірити статус в ArgoCD
argocd app get go-demo-app
```

## 2. Доступ до застосунку

```bash
# Прокинути порт до сервісу
kubectl port-forward -n demo svc/ambassador 8088:80
```

### Тест API

```bash
# Базовий запит
curl localhost:8088

# Конвертація зображення в ASCII-арт
curl -F 'image=@photo.png' localhost:8088/img/
```

---


### Відео-демонстрація роботи (YouTube)

[![Демонстрація AsciiArtify на YouTube](https://img.youtube.com/vi/D0NYzsU4hRg/0.jpg)](https://youtu.be/D0NYzsU4hRg)

---

### Команди для відтворення синхронізації вручну

```bash
# Переглянути поточний стан
argocd app get go-demo-app

# Примусова синхронізація (якщо auto-sync вимкнено)
argocd app sync go-demo-app

# Спостерігати за оновленням подів
kubectl get pods -n demo -w

# Перевірити фінальний стан
argocd app wait go-demo-app --health
```

---

## Результат

| Компонент | Статус |
|---|---|
| Kubernetes кластер (kind) | ✅ Running |
| ArgoCD | ✅ Installed & Configured |
| go-demo-app | ✅ Deployed via GitOps |
| Auto-sync | ✅ Enabled (prune + selfHeal) |
| ASCII Art API | ✅ Accessible via port-forward |

---

## Структура репозиторію

```
AsciiArtify/
├── doc/
│   ├── Concept.md    # порівняння інструментів k8s
│   ├── POC.md        # розгортання ArgoCD
│   └── MVP.md        # цей файл
└── assets/
    ├── demo-app.gif  # демо роботи застосунку
    └── demo-sync.gif # демо автосинхронізації
```

---

