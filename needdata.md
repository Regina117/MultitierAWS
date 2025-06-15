## Для работы с Terraform, Ansible и Jenkins в AWS вам понадобятся следующие данные и файлы:

### 1. AWS Credentials (IAM)
Эти данные нужны для аутентификации в AWS:

AWS_ACCESS_KEY_ID – Access Key ID для IAM-пользователя/роли

AWS_SECRET_ACCESS_KEY – Secret Access Key для IAM-пользователя/роли

AWS_REGION (например, us-east-1, eu-west-1)

AWS_SESSION_TOKEN (если используется временный доступ через STS)

Где взять?
→ AWS IAM Console → Users → Security credentials → Create access key

Формат хранения:

В Jenkins: через Credentials (тип Secret text или AWS credentials)

В Terraform: через ~/.aws/credentials или переменные окружения

В Ansible: через aws_access_key и aws_secret_key в playbook или ~/.aws/credentials

### 2. SSH-ключи (для доступа к EC2)
Нужны для подключения к серверам через Ansible/Jenkins:

id_rsa (приватный ключ)

id_rsa.pub (публичный ключ)

→ Сгенерировать локально:
ssh-keygen -t rsa -b 4096 -f ~/.ssh/aws_key
→ Или взять из AWS EC2 Console → Key Pairs → Create key pair

Формат хранения:

В Jenkins: через SSH Username with private key в Credentials
В Ansible: в ~/.ssh/ или указать путь в ansible.cfg
В Terraform: через aws_key_pair (публичный ключ)

### 3. ECR (Elastic Container Registry) Credentials
Нужны для работы с Docker-образами в AWS:

ECR_REPO_URL (например, 931130763859.dkr.ecr.us-east-1.amazonaws.com)

ECR_ACCESS_TOKEN (генерируется через aws ecr get-login-password)

Получить: aws ecr get-login-password --region us-east-1

Формат хранения:

В Jenkins: через Username and Password (логин — AWS, пароль — токен)
В Terraform: через aws_ecr_repository
В Ansible: через модуль ecr_login

### 4. Terraform State Storage (S3 + DynamoDB)
Если используется remote state:

S3_BUCKET_NAME (где хранится terraform.tfstate)

DYNAMODB_TABLE_NAME (для state-locking)

AWS_PROFILE (если используется named profile)
В Terraform (backend.tf)

### 5. Ansible Vault Password (если используется шифрование)
ANSIBLE_VAULT_PASSWORD (пароль для расшифровки vault.yml)

Формат хранения:

В Jenkins: через Secret file или Secret text

В Ansible: через --vault-password-file или переменную окружения ANSIBLE_VAULT_PASSWORD_FILE

### 6. Дополнительные AWS-ресурсы (если используются)
RDS Database Credentials (DB_HOST, DB_USER, DB_PASSWORD)

S3 Bucket Names (для хранения артефактов)

Route53 Zone ID (если управляете DNS через Terraform)

VPC ID, Subnet IDs, Security Group IDs (для инфраструктуры)

🔒 Где и как хранить эти данные?

| Данные               | Jenkins                          | Terraform                     | Ansible                      |
|----------------------|----------------------------------|-------------------------------|------------------------------|
| **AWS Keys**         | `AWS Credentials`               | `~/.aws/credentials`          | `vars.yml` / `env variables` |
| **SSH Private Key**  | `SSH Credentials`               | Указывается в `main.tf`       | `ansible.cfg` / `inventory`  |
| **ECR Password**     | `Secret Text`                   | Через `aws_ecr_repository`    | `docker_login` модуль        |
| **DB Passwords**     | `Vault` / `Credentials`         | `TF_VAR_db_password`          | `vault.yml`                  |
| **Ansible Vault**    | `Secret File`                   | Не требуется                  | `vault_password_file`        |

🚨 Важно!
Никогда не коммитьте .env, *.pem, aws-credentials.json в Git!

Используйте .gitignore:

gitignore
# AWS / sensitive keys
*.pem
*.key
*.crt
*.pub
aws-credentials.json
.env
.env.*
В Jenkins используйте Credentials Binding Plugin или HashiCorp Vault.
В Terraform используйте S3 backend + DynamoDB locking.
В Ansible используйте ansible-vault для шифрования секретов.