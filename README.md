## JavaStack Облачный проект

### Автоматизированное развёртывание многоуровневого веб-приложения в AWS

---

## Требования

Для выполнения проекта необходимо установить и настроить следующие инструменты:

* Git, Bash, любой текстовый редактор
* AWS CLI (сконфигурирован через `aws configure`)
* Terraform
* Ansible
* Docker
* Jenkins (можно локально или в EC2)
* Созданный SSH-ключ (`~/.ssh/id_rsa.pub`)
* Учетная запись в AWS с доступом к EC2, ECR, VPC и IAM

---

## Архитектура проекта

![Cloud Stack](aws-architecture.png)

            [Пользователь]
                  │
        https://multitier.store
                  │
              [nginx (ALB)]
                  │
           перенаправление
                  │
               [backend]
         ┌────────┼─────────┐
         ▼        ▼         ▼
     [Database] [Redis] [RabbitMQ]


В этом проекте развёртывание производится в **облаке AWS**, используя:

* **Terraform** — для создания инфраструктуры (EC2, VPC, Security Groups и т.д.)
* **Ansible** — для настройки всех компонентов на серверах
* **Docker** — для упаковки приложения
* **Jenkins** — для CI/CD пайплайна
* **ECR** — для хранения Docker-образов

---

## 🔑 Основные команды Terraform

Перейдите в каталог `terraform/` и выполните:

```bash
terraform init
terraform apply
```

Создастся инфраструктура:

* 5 EC2-инстансов: `frontend`, `backend`, `db`, `redis`, `rabbitmq`
* VPC, подсети, интернет-шлюз
* Security Groups
* SSH доступ по ключу

Остановить инфраструктуру:

```bash
terraform destroy
```
---
## 📂 Настройка через Ansible

После развёртывания Terraform получаем IP-адреса машин и заполняем файл `inventory.ini`:

```ini
[frontend]
3.XX.XX.10

[backend]
3.XX.XX.11

[db]
3.XX.XX.12

[redis]
3.XX.XX.13

[rabbitmq]
3.XX.XX.14
```
Затем запускаем Playbook:

```bash
ansible-playbook -i inventory.ini playbook.yml
```
Ansible выполнит:

* Установку всех необходимых сервисов
* Настройку Nginx, Tomcat, MySQL, Redis, RabbitMQ
* Копирование `application.properties`
---
## 🐳 Docker-образ приложения

Создание Dockerfile multistage
далее разделение на Dockerfile.build и Dockerfile.run 
для pipeline
---
## 🔁 Jenkins CI/CD

Создайте `Jenkinsfile`
---
## 🌐 Проверка

* Nginx: `http://<frontend-IP>/`
* Приложение (Tomcat): `http://<backend-IP>:8080/`
* RabbitMQ Web UI: `http://<rabbitmq-IP>:15672`
* Redis, MySQL: только через внутренние порты
---

## 🧩 Компоненты и роли
### 1. nginx (на отдельной VM или через AWS Load Balancer)
Отвечает на входящие HTTPS-запросы с домена multitier.store.

Является точкой входа и маршрутизирует запросы на backend-сервер.

Хостится на EC2 или через Application Load Balancer (ALB).

Также может выполнять SSL termination.

📤 Отправляет запросы на:

backend (приложение на Java)

### 2. Backend (Tomcat/Java App)
Приложение развёрнуто в Docker-контейнере.

Поднимается через docker-compose или Dockerfile, запускаемый через Ansible.

Получает артефакт (JAR/WAR) из Jenkins.

Использует application.properties для подключения к базам.

📥 Получает запросы от:

nginx

📤 Обращается к:

database (MySQL)

redis (кеш)

rabbitmq (асинхронные очереди)

### 3. Database (MySQL)
Содержит данные пользователей и транзакций.

Развёрнут на отдельной EC2 машине.

Может быть управляемым сервисом (например, Amazon RDS) или установлен вручную.

📥 Получает SQL-запросы от backend.

### 4. Redis
Используется как кеш для ускорения ответа backend.

Установлен на отдельной машине или через Amazon ElastiCache.

📥 Получает запросы от backend.

### 5. RabbitMQ
Брокер сообщений, обрабатывает события (например, регистрация, подтверждение email).

Установлен на отдельной машине.

Используется как асинхронный механизм взаимодействия между сервисами.

📥 Получает сообщения от backend.

### 6. Jenkins VM
Развёрнута на отдельной машине.

Выполняет CI/CD пайплайн:

Собирает проект.

Создаёт Docker-образ.

Пушит его в AWS Elastic Container Registry (ECR).

Деплоит образ через Ansible на backend-машину.

## 🔁 Сценарий взаимодействия
CI/CD

Jenkins (на VM) → GitHub → Сборка → Docker-образ → AWS ECR → Ansible → backend (EC2)

Запрос пользователя

Пользователь → nginx (multitier.store) → backend

backend → БД (MySQL)

backend → Redis

backend → RabbitMQ

