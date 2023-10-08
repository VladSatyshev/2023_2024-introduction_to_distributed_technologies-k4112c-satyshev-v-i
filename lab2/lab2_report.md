University: [ITMO University](https://itmo.ru/ru/)
Faculty: [FICT](https://fict.itmo.ru)
Course: [Introduction to distributed technologies](https://github.com/itmo-ict-faculty/introduction-to-distributed-technologies)
Year: 2023/2024
Group: K4112c
Author: Satyshev Vladislav Igorevich
Lab: Lab1
Date of create: 08.10.2023
Date of finished: 


# 1. Создание deployment с двумя репликами контейнера
Перед тем как приступить к созданию `deployment` обратим внимание на то, что потребуется задать переменные окружения `REACT_APP_USERNAME` и `REACT_APP_COMPANY_NAME`. В связи с этим создадим `ConfigMap`.

`ConfigMap` позволет отделить детали конфигурации от образа контейнера. Используя ConfigMaps, мы передаем данные конфигурации в виде пар ключ-значение, которые используются `Pod` или другими системными компонентами и контроллерами. 
В данной работе `ConfigMap` используется для передачи в контейнер переменных среды, однако, следует отметить, что `ConfigMap` может также передавать наборы команд и аргументов или `volumes`.
Манифест для создания `ConfigMap` представлен далее:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: config-map-react
data:
  username: TestUser
  companyName: TestCompany
```
- `apiVersion` - используемая версия API;
- `kind` - тип описываемого объекта;
- `.metadata.name` - название `ConfigMap`;
- `data` - перечисление пар ключ-значение;

Создадим `ConfigMap`:
```bash
minikube kubectl -- apply -f config_map_manifest.yaml
```
Покажем, что `ConfigMap` был успешно создан:
![Рисунок 1](images/1.PNG)

Заметим также, что для скачивания требуемого образа необходима авторизация в Docker Hub. В связи с этим создадим `Secret`, содержащий конфигурационные данные.
`Secret` по своей сути похожи на `ConfigMap` и отличаются тем, что хранящаяся в них информация кодируется, что повышает безопасность (информация не передается как plain text). Отметим, что для повышения безопасности рекомендуется зашифровать данные, прежде чем помещать их в `Secret` (и, соответственно, расшифровывать при использовании).
Для создания `Secret` можно воспользоваться командой:
```bash
minikube kubectl -- create secret docker-registry regcred \
  --docker-server=https://index.docker.io/v1/
  --docker-username=vladsatyshev \
  --docker-password=mypassword \
  --docker-email=sateshev5@yandex.ru \
```
Данная команда создает `Secret` с именем `regcred`, содержащий аутентификационные данные для Docker Hub.
Покажем, что `Secret` был успешно создан:
![Рисунок 2](images/2.PNG)

Создадим `Deployment` с двумя репликами контейнера [ifilyaninitmo/itdt-contained-frontend:master](https://hub.docker.com/layers/ifilyaninitmo/itdt-contained-frontend/master/images/sha256-08756f1022aea55538e740562aa980b56be6241d2166e6d8d6521386e0876dbe?context=explore).

`Deployment` предоставляет декларативные обновления для Pod и ReplicaSets. Принцип работы `Deployment` состоит в том, что создается описание желаемого состояния, после чего фактическое состояние стремится к желаемому за счет работы  deployment контроллера.
`Deployment` может быть использован в [ряде случаев](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#use-case). В настоящей работе `Deployment` задействован для развертывания набора реплик (`ReplicaSet`). 
Цель `ReplicaSet` - поддерживать стабильный набор `Pod`, работающих в любой момент времени. Таким образом, `ReplicaSet` используется для гарантии доступности определенного количества идентичных `Pod`, что позволяет масштабировать проект горизонтально.

Манифест для создания `Deployment` представлен далее:

```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: react-deployment
  labels:
    app: react
spec:
  replicas: 2
  selector:
    matchLabels:
      app: react
  template:
    metadata:
      labels:
        app: react
    spec:
      containers:
      - name: react-container
        image: ifilyaninitmo/itdt-contained-frontend:master
        env:
        - name: REACT_APP_USERNAME
          valueFrom: 
            configMapKeyRef:
              name: config-map-react
              key: username
        - name: REACT_APP_COMPANY_NAME
          valueFrom:
            configMapKeyRef:
              name: config-map-react
              key: companyName  
        ports:
        - containerPort: 80
      imagePullSecrets:
      - name: regcred
```

- `.spec.replicas` - указывает количество реплик;
- `.spec.selector` - указывает каким образом созданный `ReplicaSet` должен определить, каким из `Pod` управлять. В данном случае используется селектор по label (`matchLabels`), который указывает, что `ReplicaSet` управляет `Pod` у которых имеется label `app` со значением `react`;
`template` - описывает шаблон по которому будут создаваться реплики: указывается соответсвующий label (`app: react`) и спецификация;
- В спецификации указывается название контейнра (`name: react-container`), образ, на основе которого контейнер создается (`image: ifilyaninitmo/itdt-contained-frontend:master`), переменные окружения, с требуемыми в описании контейнера названиями (`REACT_APP_USERNAME`, `REACT_APP_COMPANY_NAME`), взятые из созданного ранее `ConfigMap`. `ConfigMap` находится по его имени (`name: config-map-react`), а значения - по указанным в `ConfigMap` ключам (`key: username`, `key: companyName`). Поле `ContainerPort` указывает порт контейнера, который будет доступен для кластера Kubernetes.
- `imagePullSecrets` - указывает, какой `Secret` использовать.

Покажем, что `Deployment` был успешно создан:
![Рисунок 3](images/3.PNG)

Покажем, что `ReplicaSet` был успешно создан:
![Рисунок 4](images/4.PNG)

Покажем, что было успешно создано 2 `Pod`:
![Рисунок 5](images/5.PNG)

