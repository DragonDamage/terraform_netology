# Домашнее задание:

Чек-лист готовности к домашнему заданию
Скачайте и установите Terraform версии >=1.12.0 . Приложите скриншот вывода команды terraform --version.
```bash
bogatyrevam@vm:~$ sudo snap install terraform --classic
bogatyrevam@vm:~$ terraform --version
Terraform v1.14.3
on linux_amd64
```

Скачайте на свой ПК этот git-репозиторий. Исходный код для выполнения задания расположен в директории 01/src.
```bash
bogatyrevam@vm:~$ git clone https://github.com/netology-code/ter-homeworks.git
bogatyrevam@vm:~$ cd ter-homeworks/01/src
```

Убедитесь, что в вашей ОС установлен docker.
```bash
bogatyrevam@vm:~$ docker --version
Docker version 29.1.5, build 0e6fee6
```

# Задания:
1. Перейдите в каталог src. Скачайте все необходимые зависимости, использованные в проекте.
`$ cd ter-homeworks/01/src`


2. Изучите файл .gitignore. В каком terraform-файле, согласно этому .gitignore, допустимо сохранить личную, секретную информацию?(логины,пароли,ключи,токены итд)

В файле .gitignore указана строка: `personal.auto.tfvars`
Согласно этому файлу, личную, секретную информацию (логины, пароли, ключи, токены) допустимо сохранять в файле personal.auto.tfvars.
Terraform автоматически загрузит переменные из этого файла, а Git проигнорирует его.


3. Выполните код проекта. Найдите в state-файле секретное содержимое созданного ресурса random_password, пришлите в качестве ответа конкретный ключ и его значение.

`$ terraform init`

`$ nano main.tf`

Заменяем `  required_version = "~>1.12.0"` на `  required_version = ">= 1.12.0"`
```bash
$ terraform apply -auto-approve
$ cat terraform.tfstate
            "result": "ccDy4CrR4gNOEUeY",
```

4. Раскомментируйте блок кода, примерно расположенный на строчках 29–42 файла main.tf.
Выполните команду terraform validate. Объясните, в чём заключаются намеренно допущенные ошибки. Исправьте их.

Ошибки:
`bogatyrevam@vm:~/ter-homeworks/01/src$ terraform validate`

╷
│ Error: Missing name for resource
│ 
│   on main.tf line 23, in resource "docker_image":
│   23: resource "docker_image" {
│ 
│ All resource blocks must have 2 labels (type, name).
╵
╷
│ Error: Invalid resource name
│ 
│   on main.tf line 28, in resource "docker_container" "1nginx":
│   28: resource "docker_container" "1nginx" {
│ 
│ A name must start with a letter or underscore and may contain only letters, digits, underscores, and dashes.

Решение:
1. Missing name for resource
Меняем `resource "docker_image" {` на `resource "docker_image" "nginx" {`
У ресурса обязательно должно быть второе имя (label)

2. Invalid resource name "1nginx"
Меняем `resource "docker_container" "1nginx" {` на `resource "docker_container" "nginx" {`

3. Ссылка на пароль — ещё одна ошибка
`name = "example_${random_password.random_string_FAKE.resulT}"`

Имя ресурса random_password.random_string_FAKE — должно быть random_string (как объявлено выше)
Атрибут .resulT — с опечаткой, должно быть .result

Меняем `name = "example_${random_password.random_string_FAKE.resulT}"` на `name = "example_${random_password.random_string.result}"`

```bash
bogatyrevam@vm:~/ter-homeworks/01/src$ terraform validate
Success! The configuration is valid.
```

5. Выполните код. В качестве ответа приложите: исправленный фрагмент кода и вывод команды docker ps.
`$ terraform apply -auto-approve`
```bash
docker_image.nginx: Creating...
docker_image.nginx: Creation complete after 8s [id=sha256:5cdef4ac3335f68428701c14c5f12992f5e3669ce8ab7309257d263eb7a856b1nginx:latest]
docker_container.nginx: Creating...
docker_container.nginx: Creation complete after 1s [id=3a5defe797af41c9f333ff26ffb305fb1e9bbe00d29ea2c55c0e2eb84feb15b6]

Apply complete! Resources: 2 added, 0 changed, 0 destroyed.
```

```bash
bogatyrevam@vm:~/ter-homeworks/01/src$ docker ps
CONTAINER ID   IMAGE          COMMAND                  CREATED          STATUS          PORTS                  NAMES
3a5defe797af   5cdef4ac3335   "/docker-entrypoint.…"   35 seconds ago   Up 35 seconds   0.0.0.0:9090->80/tcp   example_ccDy4CrR4gNOEUeY
```


6. Замените имя docker-контейнера в блоке кода на hello_world. Не перепутайте имя контейнера и имя образа. Мы всё ещё продолжаем использовать name = "nginx:latest". Выполните команду terraform apply -auto-approve. Объясните своими словами, в чём может быть опасность применения ключа -auto-approve. Догадайтесь или нагуглите зачем может пригодиться данный ключ? В качестве ответа дополнительно приложите вывод команды docker ps.

`$ nano main.tf`

`name = "hello_world"`

Опасность ключа -auto-approve: Пропускает интерактивное подтверждение (yes). Можно случайно применить изменения, которые удаляют критически важные ресурсы или нарушают инфраструктуру. В production-среде это может привести к необратимым последствиям (например, удаление базы данных).

Зачем нужен -auto-approve: Для автоматизации в CI/CD-пайплайнах (GitLab CI, Jenkins, GitHub Actions), где нет возможности интерактивного ввода. При тестировании в изолированной среде, где каждое применение заранее проверено.

`bogatyrevam@vm:~/ter-homeworks/01/src$ terraform apply -auto-approve`

`bogatyrevam@vm:~/ter-homeworks/01/src$ docker ps -a`

```bash
CONTAINER ID   IMAGE                        COMMAND                  CREATED          STATUS         PORTS                 NAMES
c261ca13ee30   c881927c4077                 "/docker-entrypoint.…"   28 seconds ago   Created                              hello_world
```

При выполнении:
`bogatyrevam@vm:~/ter-homeworks/01/src$ sudo terraform apply -auto-approve`

Получаю ошибку: 
```bash
Plan: 1 to add, 0 to change, 1 to destroy.
docker_container.nginx: Destroying... [id=3a5defe797af41c9f333ff26ffb305fb1e9bbe00d29ea2c55c0e2eb84feb15b6]
docker_container.nginx: Still destroying... [id=3a5defe797af41c9f333ff26ffb305fb1e9bbe00d29ea2c55c0e2eb84feb15b6, 00m10s elapsed]
╷
│ Error: Error stopping container 3a5defe797af41c9f333ff26ffb305fb1e9bbe00d29ea2c55c0e2eb84feb15b6: Error response from daemon: cannot stop container: 3a5defe797af41c9f333ff26ffb305fb1e9bbe00d29ea2c55c0e2eb84feb15b6: permission denied
```

Сделал `systemctl restart docker` и `docker.socket`, контейнер создался


7. Уничтожьте созданные ресурсы с помощью terraform. Убедитесь, что все ресурсы удалены. Приложите содержимое файла terraform.tfstate.

`$ terraform destroy -auto-approve`

`bogatyrevam@vm:~/ter-homeworks/01/src$ cat terraform.tfstate`

```bash
{
  "version": 4,
  "terraform_version": "1.14.3",
  "serial": 11,
  "lineage": "619d18f0-00a3-7258-39ec-27e6fdd0e501",
  "outputs": {},
  "resources": [],
  "check_results": null
}
```


8. Объясните, почему при этом не был удалён docker-образ nginx:latest. Ответ ОБЯЗАТЕЛЬНО НАЙДИТЕ В ПРЕДОСТАВЛЕННОМ КОДЕ, а затем ОБЯЗАТЕЛЬНО ПОДКРЕПИТЕ строчкой из документации terraform провайдера docker. (ищите в классификаторе resource docker_image )

docker-образ не был удален из-за параметра: `keep_locally = true`
Этот параметр указывает провайдеру не удалять образ с диска при уничтожении ресурса

Подтверждение из документации Terraform провайдера Docker:
Согласно официальной документации resource docker_image, аргумент keep_locally:

`https://registry.terraform.io/providers/kreuzwerker/docker/latest/docs/resources/image#keep_locally`

keep_locally (Boolean) If true, the image will be kept when the resource is destroyed. Defaults to false.
Перевод: «Если true, образ будет сохранён при уничтожении ресурса. По умолчанию false».

