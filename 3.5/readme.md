# Домашнее задание к занятию Troubleshooting

### Цель задания

Устранить неисправности при деплое приложения.

### Чеклист готовности к домашнему заданию

1. Кластер K8s.

### Задание. При деплое приложение web-consumer не может подключиться к auth-db. Необходимо это исправить

1. Установить приложение по команде:
```shell
kubectl apply -f https://raw.githubusercontent.com/netology-code/kuber-homeworks/main/3.5/files/task.yaml
```
2. Выявить проблему и описать.
3. Исправить проблему, описать, что сделано.
4. Продемонстрировать, что проблема решена.


### Правила приёма работы

1. Домашняя работа оформляется в своём Git-репозитории в файле README.md. Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.
2. Файл README.md должен содержать скриншоты вывода необходимых команд, а также скриншоты результатов.
3. Репозиторий должен содержать тексты манифестов или ссылки на них в файле README.md.

### Решение

```Запустил предложенный в задании task.yaml```

![1](https://github.com/Fe1br0/kube-homeworks/assets/106814458/fd1c046f-27ee-4dd8-9823-9bf19b9aa013)

 ```По ошибкам стало понятно, что в кластере нет namespace'ов web и data. Создал эти два namespace'a```

![2](https://github.com/Fe1br0/kube-homeworks/assets/106814458/066c0651-2222-4f9b-9cde-e7c7f2268b49)


 ```Повторяю попытку запуска task.yaml , в этот раз успешно ```


![3](https://github.com/Fe1br0/kube-homeworks/assets/106814458/e00e13f8-63dd-428e-bb7c-458abcd965f0)

 ``` Проверяю , что было создано ```

![4](https://github.com/Fe1br0/kube-homeworks/assets/106814458/b5684aad-eac1-4c7f-bcb9-9faec89306a6)



```Видим, что все ресурсы созданы корректно в пространствах имен data и web. Для дальнейшего поиска проблемы оценим логи deployment'ов `auth-db` и `web-consumer` ```


![7](https://github.com/Fe1br0/kube-homeworks/assets/106814458/5fab6e77-6a68-401b-9e2f-cbe701c9dcbb)

```Из логов видно, что под web-consumer не может выполнить разрешение имени `auth-db`. Для уточнения ситуации попробуем выполнить разрешение имени `auth-db` изнутри пода ```

![11](https://github.com/Fe1br0/kube-homeworks/assets/106814458/508fb4b9-0926-45de-aa53-a26c40f01bd3)

```Разрешение имени не работает также изнутри пода. Попробуем выполнить разрешение изнутри пода по полному имени```


![10](https://github.com/Fe1br0/kube-homeworks/assets/106814458/af5d3537-722a-4f33-b069-0df020e8086c)

```Видим, что по полному имени разрешение изнутри пода работает. Соотвественно, необходимо выполнить редактирование манифеста для deployment'a `web-consumer`, заменив в нем сокращенное имя `auth-db` на полное имя `auth-db.data.svc.cluster.local` в следующем фрагменте конфигурации в блоке `- command:` ```

```bash
kubectl -n web edit deployments web-consumer
```
```bash
      labels:
        app: web-consumer
    spec:
      containers:
      - command:
        - sh
        - -c
        - while true; do curl auth-db.data.svc.cluster.local; sleep 5; done

```
* После завершения редактирования манифеста, поды автоматически пересоздались
```bash
kubectl -n web edit deployments web-consumer
deployment.apps/web-consumer edited
```
``` Вновь оценим логи  web-consumer ```

![9](https://github.com/Fe1br0/kube-homeworks/assets/106814458/7f0ecfa0-68ac-42b9-8fe6-5fd04397c7c0)






``` Приложение web-consumer получило доступ к приложению auth-db. Проблема решена. ```
