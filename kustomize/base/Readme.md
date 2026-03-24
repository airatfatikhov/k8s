# Kustomize: Полное руководство по управлению конфигурациями Kubernetes

[![License: Apache-2.0](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)
[![Kustomize Version](https://img.shields.io/badge/Kustomize-v5.0+-green)](https://kustomize.io/)
[![Kubernetes](https://img.shields.io/badge/Kubernetes-1.20+-blue)](https://kubernetes.io/)

Полное руководство по **Kustomize** — нативному инструменту для кастомизации манифестов Kubernetes. Включает установку, примеры использования, интеграцию с GitOps и лучшие практики.

---

## 📑 Содержание

1. [Что такое Kustomize?](#-что-такое-kustomize)
2. [Установка](#-установка)
3. [Основные концепции](#-основные-концепции)
4. [Структура проекта](#-структура-проекта)
5. [Базовые примеры](#-базовые-примеры)
   - [Base конфигурация](#base-конфигурация)
   - [Overlays (наложения)](#overlays-наложения)
   - [Патчи (Patches)](#патчи-patches)
6. [Генераторы ресурсов](#-генераторы-ресурсов)
   - [ConfigMapGenerator](#configmapgenerator)
   - [SecretGenerator](#secretgenerator)
7. [Продвинутые возможности](#-продвинутые-возможности)
8. [Интеграция с GitOps](#-интеграция-с-gitops)
9. [Полезные команды](#-полезные-команды)
10. [Лучшие практики](#-лучшие-практики)
11. [Частые ошибки](#-частые-ошибки)

---

## 🚀 Что такое Kustomize?

**Kustomize** — это инструмент для кастомизации конфигураций Kubernetes, который:

| Характеристика | Описание |
| :--- | :--- |
| **Нативный** | Встроен в `kubectl` (команда `kubectl apply -k`) |
| **Без шаблонов** | Не использует шаблонизацию (в отличие от Helm) |
| **Декларативный** | Конфигурация описывается в YAML |
| **Наследование** | Позволяет создавать базовые конфигурации и переопределять их |
| **GitOps-friendly** | Идеально подходит для ArgoCD, Flux и других инструментов |

### Kustomize vs Helm

| Критерий | Kustomize | Helm |
| :--- | :--- | :--- |
| **Шаблонизация** | Нет (чистый YAML) | Да (Go templates) |
| **Сложность** | Низкая | Средняя/Высокая |
| **Пакетный менеджер** | Нет | Да (Charts) |
| **Встроен в kubectl** | Да | Нет |
| **Лучше для** | Кастомизации существующих манифестов | Упаковки и распространения приложений |

---

## 🛠 Установка

### Проверка версии (встроен в kubectl)

```bash
kubectl version --client
kustomize version
````

## Быстрый старт (Hello World)

Создадим простую структуру:

````
my-app/
├── deployment.yaml
├── service.yaml
└── kustomization.yaml
````

### 1. deployment.yaml
````yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
 ````       

 ### 2. service.yaml
 ````yaml
 apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: my-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
````

### 3. kustomization.yaml

````yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - deployment.yaml
  - service.yaml
 ````

 ### Применение:

 ````
 # Просмотр итогового YAML
kubectl kustomize .

# Применение в кластер
kubectl apply -k . 