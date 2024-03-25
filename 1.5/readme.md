

# Домашнее задание к занятию «Сетевое взаимодействие в K8S. Часть 2»

### Цель задания

В тестовой среде Kubernetes необходимо обеспечить доступ к двум приложениям снаружи кластера по разным путям.

------

### Чеклист готовности к домашнему заданию

1. Установленное k8s-решение (например, MicroK8S).
2. Установленный локальный kubectl.
3. Редактор YAML-файлов с подключённым Git-репозиторием.

------

### Инструменты и дополнительные материалы, которые пригодятся для выполнения задания

1. [Инструкция](https://microk8s.io/docs/getting-started) по установке MicroK8S.
2. [Описание](https://kubernetes.io/docs/concepts/services-networking/service/) Service.
3. [Описание](https://kubernetes.io/docs/concepts/services-networking/ingress/) Ingress.
4. [Описание](https://github.com/wbitt/Network-MultiTool) Multitool.

------

### Задание 1. Создать Deployment приложений backend и frontend

1. Создать Deployment приложения _frontend_ из образа nginx с количеством реплик 3 шт.

[frontend](https://github.com/Fe1br0/kube-homeworks/blob/main/1.5/frontend.yaml)

![1](https://github.com/Fe1br0/kube-homeworks/assets/106814458/d1082877-2d41-4e6e-80f1-2c2bbd947856)


2. Создать Deployment приложения _backend_ из образа multitool. 

[backend](https://github.com/Fe1br0/kube-homeworks/blob/main/1.5/backend.yaml)


![2](https://github.com/Fe1br0/kube-homeworks/assets/106814458/5251e25b-fbae-465e-9165-dd4026218e55)


3. Добавить Service, которые обеспечат доступ к обоим приложениям внутри кластера. 

[service-backend](https://github.com/Fe1br0/kube-homeworks/blob/main/1.5/service-backend.yaml)

[service-frontend](https://github.com/Fe1br0/kube-homeworks/blob/main/1.5/service-frontend.yaml)

![3](https://github.com/Fe1br0/kube-homeworks/assets/106814458/a60c464b-60a9-4c6a-95f9-b2dafc91f255)


4. Продемонстрировать, что приложения видят друг друга с помощью Service.

![4](https://github.com/Fe1br0/kube-homeworks/assets/106814458/39acbbca-705b-4f4c-a115-b283949768d9)


------

### Задание 2. Создать Ingress и обеспечить доступ к приложениям снаружи кластера

1. Включить Ingress-controller в MicroK8S.

![5](https://github.com/Fe1br0/kube-homeworks/assets/106814458/1aa483da-111d-4c9c-bb5d-80a4695c1107)


2. Создать Ingress, обеспечивающий доступ снаружи по IP-адресу кластера MicroK8S так, чтобы при запросе только по адресу 
открывался _frontend_ а при добавлении /api - _backend_.

![6](https://github.com/Fe1br0/kube-homeworks/assets/106814458/9077ef7a-c098-49eb-a62f-c4b37a9d8471)




3. Продемонстрировать доступ с помощью браузера или `curl` с локального компьютера.

front


![8](https://github.com/Fe1br0/kube-homeworks/assets/106814458/fabe03fb-2d45-414a-9fbe-fb8c91c54b37)



back

![9](https://github.com/Fe1br0/kube-homeworks/assets/106814458/62defcaf-34d3-493b-b20e-bd591e2e7d5a)


------

### Правила приема работы

1. Домашняя работа оформляется в своем Git-репозитории в файле README.md. Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.
2. Файл README.md должен содержать скриншоты вывода необходимых команд `kubectl` и скриншоты результатов.
3. Репозиторий должен содержать тексты манифестов или ссылки на них в файле README.md.

------
