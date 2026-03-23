# Kyverno: Управление политиками Kubernetes

[![License: Apache-2.0](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)
[![Kyverno Version](https://img.shields.io/badge/Kyverno-v1.11+-green)](https://kyverno.io/)

Полное руководство по внедрению, настройке и управлению политиками безопасности и конфигурации в Kubernetes с помощью **Kyverno**. Включает примеры `ClusterPolicy`, `Policy` (namespace-scoped) и интеграцию с **Kustomize** для GitOps.

## 📑 Содержание

- [Kyverno: Управление политиками Kubernetes](#kyverno-управление-политиками-kubernetes)
  - [📑 Содержание](#-содержание)
  - [🚀 Что такое Kyverno?](#-что-такое-kyverno)
  - [🛠 Установка](#-установка)
  - [Режимы работы (validationFailureAction):](#режимы-работы-validationfailureaction)
- [📝 Примеры политик](#-примеры-политик)
  - [Валидация (Validate)](#валидация-validate)
  - [Мутация (Mutate)](#мутация-mutate)
  - [Namespace-scoped Policy](#namespace-scoped-policy)
- [🧩 Интеграция с Kustomize (GitOps)](#-интеграция-с-kustomize-gitops)
  - [1. Базовая политика (base/restrict-registry.yaml)](#1-базовая-политика-baserestrict-registryyaml)
  - [2. Базовый Kustomize (base/kustomization.yaml)](#2-базовый-kustomize-basekustomizationyaml)
  - [3. Оверлей для Dev (overlays/dev/kustomization.yaml)](#3-оверлей-для-dev-overlaysdevkustomizationyaml)
  - [4. Патч для Dev (overlays/dev/patch-registry.yaml)](#4-патч-для-dev-overlaysdevpatch-registryyaml)
  - [5. Применение](#5-применение)
- [📊 Мониторинг и отчеты](#-мониторинг-и-отчеты)
- [Посмотреть глобальный отчет](#посмотреть-глобальный-отчет)
- [Детали нарушения](#детали-нарушения)

---

## 🚀 Что такое Kyverno?

**Kyverno** (греч. "управлять") — это движок политик для Kubernetes.
*   **Kubernetes-native:** Использует стандартные ресурсы Kubernetes (CRD).
*   **YAML-based:** Политики пишутся на YAML (не нужно учить Rego, как в OPA).
*   **Функционал:** Валидация, Мутация, Генерация ресурсов, Проверка образов.

---

## 🛠 Установка

Рекомендуемый способ установки — через Helm.

```bash
# Добавление репозитория
helm repo add kyverno https://kyverno.github.io/kyverno/
helm repo update

# Установка в отдельный неймспейс
helm install kyverno kyverno/kyverno -n kyverno --create-namespace --wait

# Проверка статуса
kubectl get pods -n kyverno
````

## Режимы работы (validationFailureAction):
* **Enforce:** Блокировать запрос при нарушении.
* **Audit:** Разрешить запрос, но записать нарушение в отчет.

# 📝 Примеры политик

## Валидация (Validate)

Запрет создания подов без лейбла app.

````apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: require-app-label
spec:
  validationFailureAction: Enforce
  background: true
  rules:
    - name: check-app-label
      match:
        any:
          - resources:
              kinds:
                - Pod
      validate:
        message: "Лейбл 'app' обязателен для всех подов!"
        pattern:
          metadata:
            labels:
              app: "?*"
````

## Мутация (Mutate)
Автоматическое добавление securityContext для всех подов.

````apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: add-security-context
spec:
  rules:
    - name: add-run-as-non-root
      match:
        any:
          - resources:
              kinds:
                - Pod
      mutate:
        patchStrategicMerge:
          spec:
            securityContext:
              runAsNonRoot: true
````

## Namespace-scoped Policy

Правило, действующее только в неймспейсе development.

````apiVersion: kyverno.io/v1
kind: Policy
metadata:
  name: require-cpu-limits
  namespace: development # Важно: политика живет только здесь
spec:
  validationFailureAction: Audit
  rules:
    - name: cpu-limits-required
      match:
        any:
          - resources:
              kinds:
                - Pod
      validate:
        message: "CPU limits обязательны в dev окружении"
        pattern:
          spec:
            containers:
              - resources:
                  limits:
                    cpu: "?*"
````

# 🧩 Интеграция с Kustomize (GitOps)

Управление политиками как кодом. Позволяет иметь базовую политику и переопределять параметры (например, адрес реестра) для разных окружений (dev, prod).

````policies/
├── base/
│   ├── kustomization.yaml
│   └── restrict-registry.yaml
└── overlays/
    ├── dev/
    │   ├── kustomization.yaml
    │   └── patch-registry.yaml
    └── prod/
        ├── kustomization.yaml
        └── patch-registry.yaml
````

## 1. Базовая политика (base/restrict-registry.yaml)

````apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: restrict-registry
spec:
  validationFailureAction: Enforce
  rules:
    - name: validate-registry
      match:
        any:
          - resources:
              kinds:
                - Pod
      validate:
        message: "Недоверенный реестр образов"
        pattern:
          spec:
            containers:
              - image: "PLACEHOLDER_REGISTRY/*"
````

## 2. Базовый Kustomize (base/kustomization.yaml)

````
resources:
  - restrict-registry.yaml
````

## 3. Оверлей для Dev (overlays/dev/kustomization.yaml)

````
resources:
  - ../../base

patchesStrategicMerge:
  - patch-registry.yaml
```` 

## 4. Патч для Dev (overlays/dev/patch-registry.yaml)

````apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: restrict-registry
spec:
  rules:
    - name: validate-registry
      validate:
        pattern:
          spec:
            containers:
              - image: "dev-registry.internal.com/*"
````

## 5. Применение

````# Применить политику для DEV
kustomize build policies/overlays/dev | kubectl apply -f -

# Применить политику для PROD
kustomize build policies/overlays/prod | kubectl apply -f -
````

# 📊 Мониторинг и отчеты

Kyverno автоматически генерирует отчеты о соответствии.

````# Посмотреть отчеты по неймспейсам
kubectl get policyreport -A

# Посмотреть глобальный отчет
kubectl get clusterpolicyreport

# Детали нарушения
kubectl describe policyreport <name> -n <namespace>