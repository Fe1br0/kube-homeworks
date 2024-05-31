# Домашнее задание к занятию «Организация сети»

### Подготовка к выполнению задания

1. Домашнее задание состоит из обязательной части, которую нужно выполнить на провайдере Yandex Cloud, и дополнительной части в AWS (выполняется по желанию). 
2. Все домашние задания в блоке 15 связаны друг с другом и в конце представляют пример законченной инфраструктуры.  
3. Все задания нужно выполнить с помощью Terraform. Результатом выполненного домашнего задания будет код в репозитории. 
4. Перед началом работы настройте доступ к облачным ресурсам из Terraform, используя материалы прошлых лекций и домашнее задание по теме «Облачные провайдеры и синтаксис Terraform». Заранее выберите регион (в случае AWS) и зону.

---
### Задание 1. Yandex Cloud 

**Что нужно сделать**

1. Создать пустую VPC. Выбрать зону.
2. Публичная подсеть.

 - Создать в VPC subnet с названием public, сетью 192.168.10.0/24.
 - Создать в этой подсети NAT-инстанс, присвоив ему адрес 192.168.10.254. В качестве image_id использовать fd80mrhj8fl2oe87o4e1.
 - Создать в этой публичной подсети виртуалку с публичным IP, подключиться к ней и убедиться, что есть доступ к интернету.
3. Приватная подсеть.
 - Создать в VPC subnet с названием private, сетью 192.168.20.0/24.
 - Создать route table. Добавить статический маршрут, направляющий весь исходящий трафик private сети в NAT-инстанс.
 - Создать в этой приватной подсети виртуалку с внутренним IP, подключиться к ней через виртуалку, созданную ранее, и убедиться, что есть доступ к интернету.

Resource Terraform для Yandex Cloud:

- [VPC subnet](https://registry.terraform.io/providers/yandex-cloud/yandex/latest/docs/resources/vpc_subnet).
- [Route table](https://registry.terraform.io/providers/yandex-cloud/yandex/latest/docs/resources/vpc_route_table).
- [Compute Instance](https://registry.terraform.io/providers/yandex-cloud/yandex/latest/docs/resources/compute_instance).

---
    ### Правила приёма работы

Домашняя работа оформляется в своём Git репозитории в файле README.md. Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.
Файл README.md должен содержать скриншоты вывода необходимых команд, а также скриншоты результатов.
Репозиторий должен содержать тексты манифестов или ссылки на них в файле README.md.

### Решение 

* Проверяем работоспособность `yc`:

```bash
PS D:\Git\kube-homeworks\15.1> yc config list
token: xxxx
cloud-id: b1gn61l7bppdgs9dpppc
folder-id: b1gkj8ffp01tf8okka77
compute-default-zone: ru-central1-b
```



* Проверим конфигурацию Terraform для созданного файла netology.tf

![1](https://github.com/Fe1br0/kube-homeworks/assets/106814458/b993e710-c33a-4250-b464-ffacf8fa45f3)


* Запустим создание ресурсов с помощью Terraform:
```bash
> terraform apply --auto-approve 
```
![2](https://github.com/Fe1br0/kube-homeworks/assets/106814458/fdaa8e02-ec7f-48e7-bffc-f2f95738eadb)

![3](https://github.com/Fe1br0/kube-homeworks/assets/106814458/69a5e66d-91f6-49f3-b024-a26925f89941)

![5](https://github.com/Fe1br0/kube-homeworks/assets/106814458/3f35c12e-3344-4fa1-b8b9-d878897004fb)
![4](https://github.com/Fe1br0/kube-homeworks/assets/106814458/a7686e55-892a-478f-b1c7-572d15f13491)

* Убедимся, что в сети с именем network-netology созданы подсети public и private (при этом private с route table):


![6](https://github.com/Fe1br0/kube-homeworks/assets/106814458/76e5f645-fd6e-419f-a8c4-c92afc5e892f)

* Убедимся, что созданы виртуальные машины nat-instance, public-instance и private-instance:
  
![7](https://github.com/Fe1br0/kube-homeworks/assets/106814458/778291dc-41ef-48a7-a72e-db5c1003c7b6)


* Подключаемся к публичной машине и проверяем наличие соединения с интернетом

![10](https://github.com/Fe1br0/kube-homeworks/assets/106814458/623806de-1922-49e8-8e57-f3743b8ffa81)


* Виртуальная машина private-instance не имеет внешнего IP-адреса. Подключение к данной виртуальной машине выполним через виртуальную машину public-instance, скопировав на нее приватный ключ ssh:

```bash
C:\Users\Fel>scp C:\Users\Fel\.ssh\id_rsa ubuntu@158.160.90.172:/home/ubuntu/.ssh/id_rsa
```
* Изменим права доступа ключа

```
chmod 0600 /home/ubuntu/.ssh/id_rsa
```





* Находясь на виртуальной машине private-instance, проверим доступ в Интернет:

![9](https://github.com/Fe1br0/kube-homeworks/assets/106814458/b25530d0-e975-4e18-9ea7-c9268fef3ed2)


[Файл netology.tf](https://github.com/Fe1br0/kube-homeworks/blob/main/15.1/netology.tf)
