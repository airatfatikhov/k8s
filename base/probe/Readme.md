**Настройка Probes (Пробов) в Kubernetes —** это один из самых важных аспектов стабильности приложения. Неправильные числа приводят либо к постоянным перезагрузкам (restart loops), либо к тому, что кластер отправляет трафик на "мертвые" поды (502/503 ошибки).

В Kubernetes есть три типа пробов:
* **Liveness (Живучесть):** "Приложение зависло? Перезагрузи его".
* **Readiness (Готовность):** "Приложение готово принимать трафик? Если нет — убери из балансировщика".
* **Startup (Запуск):** "Приложение еще загружается? Не трогай его, пока не закончит" (для медленных приложений).

# ⚠️ Формула времени до действия
## 1: Быстрое веб-приложение (Go, Node.js, Python)
Такие приложения стартуют быстро (1-5 секунд). Им не нужен startupProbe.
Важно: Используйте отдельный эндпоинт (например, /healthz), а не главный /. Главный эндпоинт может быть тяжелым или вызывать побочные эффекты.

````yaml
apiVersion: v1
kind: Pod
metadata:
  name: fast-web-app
spec:
  containers:
    - name: app
      image: my-app:latest
      ports:
        - containerPort: 8080
      
      # 1. LIVENESS (Перезагрузка если завис)
      livenessProbe:
        httpGet:
          path: /healthz      # Легкий эндпоинт, просто возвращает 200 OK
          port: 8080
        initialDelaySeconds: 10  # Даем 10 сек на старт
        periodSeconds: 10        # Проверяем каждые 10 сек
        timeoutSeconds: 2        # Если нет ответа 2 сек — ошибка
        failureThreshold: 3      # 3 ошибки подряд = рестарт
        # Итого: под завис -> через 40 сек рестарт

      # 2. READINESS (Трафик только если готов)
      readinessProbe:
        httpGet:
          path: /ready        # Проверяет подключение к БД, кэшу и т.д.
          port: 8080
        initialDelaySeconds: 5   # Начинаем проверять раньше, чем liveness
        periodSeconds: 5         # Проверяем чаще, чтобы быстрее убрать трафик
        timeoutSeconds: 2
        failureThreshold: 3      # 3 ошибки = убрать из Service (но не рестарт!)
        # Итого: БД упала -> через 20 сек трафик уйдет на другие поды
````

## 2: Медленное Java-приложение (Spring Boot)
Java-приложения могут стартовать 1-5 минут. Если поставить обычный livenessProbe, Kubernetes убьет под еще до того, как он запустится. Здесь нужен startupProbe.
Логика: Пока startupProbe не вернет успех, liveness и `readiness** отключены.

````yaml
apiVersion: v1
kind: Pod
metadata:
  name: java-slow-app
spec:
  containers:
    - name: app
      image: my-java-app:latest
      ports:
        - containerPort: 8080

      # 1. STARTUP (Самый важный для Java!)
      startupProbe:
        httpGet:
          path: /healthz
          port: 8080
        initialDelaySeconds: 10
        periodSeconds: 10
        failureThreshold: 30    # 30 попыток * 10 сек = 300 сек (5 минут) на старт
        # Если за 5 минут не стартовал — под убивается и создается заново.

      # 2. LIVENESS (Включается ТОЛЬКО после успеха startupProbe)
      livenessProbe:
        httpGet:
          path: /healthz
          port: 8080
        initialDelaySeconds: 0   # Не ждем, startup уже проверил старт
        periodSeconds: 10
        failureThreshold: 3
        timeoutSeconds: 5        # Java может делать GC паузы, даем больше времени

      # 3. READINESS
      readinessProbe:
        httpGet:
          path: /ready
          port: 8080
        initialDelaySeconds: 0
        periodSeconds: 5
        failureThreshold: 3
````

## 3: База данных или TCP сервис (Redis, Postgres)

Здесь нет HTTP, поэтому используем tcpSocket или exec.

````yaml
apiVersion: v1
kind: Pod
metadata:
  name: redis-db
spec:
  containers:
    - name: redis
      image: redis:7
      ports:
        - containerPort: 6379
      
      # Проверка порта
      livenessProbe:
        tcpSocket:
          port: 6379
        initialDelaySeconds: 15
        periodSeconds: 20       # Реже, чтобы не нагружать БД
        failureThreshold: 3

      # Проверка командой (более глубокая)
      readinessProbe:
        exec:
          command:
            - redis-cli
            - ping
        initialDelaySeconds: 5
        periodSeconds: 10
        failureThreshold: 3