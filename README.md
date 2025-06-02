
# Домашнее задание «Terraform. Знакомство и практика»  
Автор: **Асадбеков Асадбек**

> Репозиторий с кодом: <https://github.com/asad-bekov/yc-terraform-homework>

---

## Содержание
1. [Обзор задания](#overview)
2. [Задание 1 — локальный Docker + Terraform](#task1)
3. [Задание 2 — удалённый Docker через SSH (Yandex Cloud)](#task2)
4. [Задание 3 — OpenTofu](#task3)
5. [Ответы на вопросы](#answers)
6. [Команды для запуска](#commands)
7. [Скриншоты](#screenshots)
8. [Секреты и безопасность](#secrets)
9. [Полезные ссылки](#links)

---

## <a name="overview"></a>Обзор задания

Практика разбита на три логических части:

| № | Что делаем | Цель |
|---|-----------|------|
| **1** | Управляем локальным Docker‑демоном и поднимаем контейнер **nginx** | Знакомство с базовыми командами `terraform init / plan / apply / destroy` и провайдером **docker** |
| **2** | Подключаемся из Terraform к удалённому Docker‑демону на ВМ Yandex Cloud и запускаем **MySQL 8** | Работа с `host = "ssh://..."`, генератор паролей `random_password`, изоляция порта `127.0.0.1:3306` |
| **3** | Проверяем тот же конфиг с помощью **OpenTofu** | Доказательство совместимости конфигурации Terraform ↔ OpenTofu |

---

## <a name="task1"></a>Задание 1 — локальный Docker

1. **Установка Terraform ≥ 1.8.4**  
   ![Terraform version](https://github.com/asad-bekov/hw-22/raw/main/img/1.png)

2. **Установка Docker (локально)**  
   ![Docker local version](https://github.com/asad-bekov/hw-22/raw/main/img/2.png)

3. **Клонирование репозитория и `terraform init`**  
   ![terraform init](https://github.com/asad-bekov/hw-22/raw/main/img/3.png)

4. **Поиск random_password и `terraform apply`**  
   ![terraform apply nginx](https://github.com/asad-bekov/hw-22/raw/main/img/4.1.png)
   ![результат random_password](https://github.com/asad-bekov/hw-22/raw/main/img/4.2.png)

5. **Контейнер `hello_world` после переименования**  
   ![docker ps hello_world](https://github.com/asad-bekov/hw-22/raw/main/img/5.png)

6. **`terraform apply -auto-approve` + `docker ps`**  
   ![auto‑approve result](https://github.com/asad-bekov/hw-22/raw/main/img/6.png)

7. **`terraform destroy`**  
   ![destroy](https://github.com/asad-bekov/hw-22/raw/main/img/7.png)

### Исправления намеренных ошибок (файл `main.tf`)

```hcl
resource "docker_image" "nginx_image" {
  name         = "nginx:latest"
  keep_locally = true          # образ остаётся после destroy
}

resource "docker_container" "nginx" {
  name  = "hello_world"        # исправил имя (не с цифры)
  image = docker_image.nginx_image.name

  ports {
    internal = 80
    external = 9090
  }
}
```

*Ошибки до фикса:* отсутствовало имя ресурса у `docker_image`, имя контейнера начиналось с цифры, использовался атрибут `image_id` вместо `name`, опечатка в ссылке на `random_password`, дублирующий блок `ports`.

---

## <a name="task2"></a>Задание 2 — удалённый Docker (Yandex Cloud)

1. **ВМ `deb-docker`**  
   ![](./8.png)
   ![ВМ в панели YC](https://github.com/asad-bekov/hw-22/raw/main/img/8.png)

2. **Docker на ВМ**  
   ![docker version vm](https://github.com/asad-bekov/hw-22/raw/main/img/9.png)

3. **`docker ps` на ВМ** (после `terraform apply`)  
   ![docker ps mysql](https://github.com/asad-bekov/hw-22/raw/main/img/10.png)

4. **`terraform apply` (4 added)**  
   ![terraform apply mysql](https://github.com/asad-bekov/hw-22/raw/main/img/10.1.png)

5. **ENV‑переменные внутри MySQL‑контейнера**  
   ![env mysql](https://github.com/asad-bekov/hw-22/raw/main/img/11.png)

---

## <a name="task3"></a>Задание 3 — OpenTofu

1. **Версия OpenTofu**  
   ![](./12.png)
   ![tofu version](https://github.com/asad-bekov/hw-22/raw/main/img/12.png)

2. **`tofu init`**  
   ![](./13.png)
   ![tofu init](https://github.com/asad-bekov/hw-22/raw/main/img/13.png)

3. **`tofu apply` (0 added, 0 changed)** / `apply`  

   ![tofu apply](https://github.com/asad-bekov/hw-22/raw/main/img/14.png)
   
4. **`tofu destroy` (0 added, 0 changed)** / `destroy`  

   ![tofu destroy](https://github.com/asad-bekov/hw-22/raw/main/img/15.png)

---

## <a name="answers"></a>Ответы на вопросы

| № | Вопрос | Ответ |
|---|--------|-------|
| 1 | Где хранить секреты? | **`personal.auto.tfvars`**, игнорируется `.gitignore`. |
| 2 | Значение `random_password`? | `4OWVMqC18dzhrzWa` |
| 3 | Какие ошибки были в закомментированном блоке? | Отсутствие имени ресурса `docker_image`, имя контейнера начиналось с цифры, неправильный атрибут `image_id`, опечатка в ссылке на пароль, дублирующий блок `ports`. |
| 4 | Опасность `-auto-approve`? | План выполняется без финального подтверждения → можно случайно удалить прод‑ресурсы. Полезен в CI/CD. |
| 5 | Почему образ `nginx:latest` не удалился? | `keep_locally = true` у ресурса `docker_image`. Док‑ция провайдера: «image won't be deleted on destroy». |

---

## <a name="commands"></a>Команды для запуска

```bash
# Задание 1
cd ter-homeworks/01/src
terraform init && terraform apply

# Задание 2
cd ~/yc-remote-docker
terraform init && terraform apply

# Задание 3
rm -rf .terraform tofu.lock.hcl
tofu init && tofu apply
```

---

## <a name="screenshots"></a>Скриншоты

См. папку .img репозитория:

| Файл | Описание |
|------|----------|
| `1.png` | terraform --version (локально) |
| `2.png` | docker --version (локально) |
| `3.png` | terraform init |
| `4.1.png` | terraform apply (nginx) |
| `4.2.png` | вывод random_password |
| `5.png` | docker ps (`hello_world`) |
| `6.png` | apply с `-auto-approve` + docker ps |
| `7.png` | terraform destroy |
| `8.png` | ВМ в панели YC |
| `9.png` | docker --version на ВМ |
| `10.1.png` | terraform apply (MySQL) + docker ps |
| `11.png` | ENV в MySQL‑контейнере |
| `12.png` | tofu --version |
| `13.png` | tofu init |
| `14.png` | tofu apply |
| `15.png` | tofu destroy |

---

## <a name="secrets"></a>Секреты и безопасность

* Токен Yandex Cloud и прочие чувствительные переменные лежат в **`personal.auto.tfvars`**.  
* Файл добавлен в `.gitignore`; при пуше в GitHub не публикуется.

---

## <a name="links"></a>Полезные ссылки
* Docker‑provider: <https://registry.terraform.io/providers/kreuzwerker/docker/latest>  
* OpenTofu docs: <https://opentofu.org/docs/>  
* HashiCorp mirror (RU): <https://hashicorp-releases.yandexcloud.net/>

