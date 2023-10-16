University: [ITMO University](https://itmo.ru/ru/)
Faculty: [FICT](https://fict.itmo.ru)
Course: [Introduction to distributed technologies](https://github.com/itmo-ict-faculty/introduction-to-distributed-technologies)
Year: 2023/2024
Group: K4112c
Author: Satyshev Vladislav Igorevich
Lab: Lab4
Date of create: 17.10.2023
Date of finished: 


# 1. Запуск minikube с плагином calico с 2 `Node`

Недостатком стандартной реализации сетевого взаимодействия в Kubernetes является низкая безопасность: сетевой трафик в/из `Pods` разрешен по умолчанию. Если не заблокировать сетевое взаимодействие с помощью сетевой политики (network policy), все `Pods` смогут свободно взаимодействовать с другими `Pods`.

Calico состоит из Container Network Interface (CNI) в виде подключаемого модуля к minikube, который обеспечивает сетевое взаимодействие между рабочим нагрузками (workloads) и из пакета сетевых политик (Calico network policy suite) для обеспечения безопасности облачных микросервисов/приложений любого масштаба. [source](https://docs.tigera.io/calico/latest/about/)

Запустим кластер minikube с CNI плагином Calico с двумя `Node`:

```bash
minikube start --network-plugin=cni --cni=calico --nodes 2
```
Покажем, что было создано 2 `Node` с работающим плагином Calico:

![Рисунок 1](images/1.PNG)





