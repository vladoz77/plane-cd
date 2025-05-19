# Развертывание Plane с помощью ArgoCD

Этот репозиторий содержит конфигурацию для развертывания [Plane](https://plane.so/) с использованием ArgoCD в Kubernetes-кластере.

## Требования

- Kubernetes-кластер с установленным ArgoCD
- Настроенный `kubectl` для доступа к кластеру
- Установленный Helm (для управления чартом Plane)
- Установленный cert-manager (если используется TLS/SSL)

## Структура проекта

Конфигурация состоит из двух основных частей:

1. **AppProject**: Определяет область проекта и разрешения
2. **Application**: Определяет непосредственно развертывание Plane

## Конфигурация развертывания

### AppProject (`plane-project`)

- **Исходные репозитории**:
  - `https://github.com/vladoz77/plane-cd.git` (пользовательские конфигурации)
  - `https://helm.plane.so/` (официальный Helm-чарт Plane)
  
- **Назначения**:
  - Развертывание в пространстве имен `plane` в том же кластере
  
- **Разрешения**:
  - Разрешает все ресурсы в пространстве имен (`namespaceResourceWhitelist`)
  - Разрешает все кластерные ресурсы (`clusterResourceWhitelist`)
  - Предупреждает о "осиротевших" ресурсах

### Application (`plane`)

- **Источники**:
  - Пользовательские значения из GitHub-репозитория
  - Helm-чарт Plane (версия 1.1.1)
  
- **Политика синхронизации**:
  - Автоматическая синхронизация включена
  - Создает пространство имен, если оно не существует
  - Удаляет ресурсы в последнюю очередь при синхронизации
  
- **Конфигурация Ingress**:
  - Хост: `plane.home-local.site`
  - TLS с использованием cert-manager и `letsencrypt-issuer`

## Шаги развертывания

1. Примените AppProject:
   ```bash
   kubectl apply -f plane-project.yaml -n argocd
   ```

2. Примените Application:
   ```bash
   kubectl apply -f plane-application.yaml -n argocd
   ```

3. Мониторинг развертывания через UI ArgoCD или команду:
   ```bash
   argocd app get plane
   ```

## Настройка

Для настройки развертывания:

1. Отредактируйте `values.yaml` в GitHub-репозитории
2. Обновите `targetRevision` в Application до нужной версии чарта
3. Измените хост и аннотации Ingress при необходимости

## Доступ к Plane

После успешного развертывания Plane будет доступен по адресу:
- `https://plane.home-local.site` (при правильной настройке DNS)

## Устранение неполадок

- Проверка статуса синхронизации в ArgoCD:
  ```bash
  argocd app sync plane
  argocd app get plane
  ```

- Проверка Kubernetes-ресурсов:
  ```bash
  kubectl get all -n plane
  ```

- Просмотр логов конкретных компонентов:
  ```bash
  kubectl logs -n plane <имя-pod>
  ```

## Обслуживание

Для обновления Plane:

1. Обновите `targetRevision` в Application до нужной версии
2. ArgoCD автоматически синхронизирует изменения (если включена автоматическая синхронизация)

Для ручной синхронизации:
```bash
argocd app sync plane
```
