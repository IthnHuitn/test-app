# Test App

Веб-приложение для дипломного практикума в Yandex Cloud. Деплой через GitHub Actions в Kubernetes-кластер.

## Структура репозитория

```
.
├── .github/
│   └── workflows/
│       └── build-deploy.yml # CI/CD пайплайн
├── k8s/
│   ├── deployment.yaml      # Deployment с liveness/readiness пробами
│   ├── service.yaml         # ClusterIP сервис
│   └── ingress.yaml         # Ingress-маршрут через ingress-nginx
├── html
│   └── index.html           # Статичная страница 
├── Dockerfile               # Сборка образа
├── nginx.conf               # Конфигурационный файл nginx
└── README.md                # Описание репозитория
```

## CI/CD

Пайплайн запускается при коммите или пуше тега `v*` в репозиторий. Этапы:

1. **Build** — сборка Docker-образа и пуш в Yandex Container Registry.
2. **Deploy** — `kubectl apply` манифестов из `k8s/` с подстановкой переменных через `envsubst`, ожидание rollout.

### Необходимые GitHub Secrets

| Секрет | Описание |
|--------|----------|
| `YC_REGISTRY_ID` | ID реестра в Yandex Container Registry |
| `YC_SA_KEY` | JSON-ключ сервисного аккаунта YC |
| `KUBECONFIG` | Конфиг kubeconfig в base64 |
| `LB_PUBLIC_IP` | Публичный IP балансировщика / Ingress |

## Локальная сборка и пуш

```bash
# Сборка
docker build -t cr.yandex/<REGISTRY_ID>/test-app:1.0.1 .

# Логин в registry
docker login cr.yandex -u json_key -p "$(cat key.json)"

# Пуш
docker push cr.yandex/<REGISTRY_ID>/test-app:1.0.1
```

## Манифесты Kubernetes

- **Deployment** — 2 реплики, resource limits, liveness/readiness на `/health`
- **Service** — ClusterIP, порт 80
- **Ingress** — маршрут через ingress-nginx controller, host — публичный IP

## Запуск деплоя

```bash
git tag v1.0.1
git push origin v1.0.1
```

Пайплайн соберёт образ, запушит в registry и развернёт в кластере.
