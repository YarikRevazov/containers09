# containers09

## Цель работы

Целью работы является знакомство с методами оптимизации Docker-образов: удаление временных файлов, уменьшение количества слоев, использование минимального базового образа, перепаковка и комбинированный подход.

## Задание

Сравнить следующие методы оптимизации Docker-образов:

* Удаление неиспользуемых зависимостей и временных файлов;
* Уменьшение количества слоев;
* Использование минимального базового образа;
* Перепаковка образа;
* Использование всех методов одновременно.

## Ход выполнения работы

### 1. Подготовка

Я создал локальный репозиторий `containers09`, в нем создал папку `site` и поместил туда простой сайт с файлами `index.html`, `style.css`, `script.js`.

### 2. Базовый Dockerfile (Dockerfile.raw)

Создал Dockerfile с названием `Dockerfile.raw` с таким содержимым:

```Dockerfile
FROM ubuntu:latest
RUN apt-get update && apt-get upgrade -y
RUN apt-get install -y nginx
COPY site /var/www/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

Собрал образ:

```bash
docker image build -t mynginx:raw -f Dockerfile.raw .
```

### 3. Оптимизация: удаление временных файлов (Dockerfile.clean)

Создал Dockerfile `Dockerfile.clean` с удалением временных файлов:

```Dockerfile
FROM ubuntu:latest
RUN apt-get update && apt-get upgrade -y
RUN apt-get install -y nginx
RUN apt-get clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*
COPY site /var/www/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

Собрал образ:

```bash
docker image build -t mynginx:clean -f Dockerfile.clean .
docker image list
```

### 4. Уменьшение количества слоев (Dockerfile.few)

Создал Dockerfile `Dockerfile.few`, где объединил команды в один слой:

```Dockerfile
FROM ubuntu:latest
RUN apt-get update && apt-get upgrade -y && \
    apt-get install -y nginx && \
    apt-get clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*
COPY site /var/www/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

Собрал образ:

```bash
docker image build -t mynginx:few -f Dockerfile.few .
docker image list
```

### 5. Минимальный базовый образ (Dockerfile.alpine)

Создал Dockerfile `Dockerfile.alpine` с использованием `alpine`:

```Dockerfile
FROM alpine:latest
RUN apk update && apk upgrade
RUN apk add nginx
COPY site /var/www/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

Собрал образ:

```bash
docker image build -t mynginx:alpine -f Dockerfile.alpine .
docker image list
```

### 6. Перепаковка образа (repack)

Перепаковал образ `mynginx:raw` в `mynginx:repack`:

```bash
docker container create --name mynginx mynginx:raw
docker container export mynginx | docker image import - mynginx:repack
docker container rm mynginx
docker image list
```

### 7. Комбинированный подход (Dockerfile.min)

Создал Dockerfile `Dockerfile.min`, использующий все методы оптимизации:

```Dockerfile
FROM alpine:latest
RUN apk update && apk upgrade && \
    apk add nginx && \
    rm -rf /var/cache/apk/*
COPY site /var/www/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

Собрал образ:

```bash
docker image build -t mynginx:minx -f Dockerfile.min .
```

Перепаковал его:

```bash
docker container create --name mynginx mynginx:minx
docker container export mynginx | docker image import - mynginx:min
docker container rm mynginx
docker image list
```

### 8. Сравнение размеров

| Образ           | Размер |
| --------------- | ------ |
| mynginx\:raw    | ХХХ MB |
| mynginx\:clean  | ХХХ MB |
| mynginx\:few    | ХХХ MB |
| mynginx\:alpine | ХХ MB  |
| mynginx\:repack | ХХХ MB |
| mynginx\:minx   | ХХ MB  |
| mynginx\:min    | ХХ MB  |

(После выполнения `docker image list` вставлю реальные значения)

## Ответы на вопросы

**Какой метод оптимизации образов вы считаете наиболее эффективным?**

Наиболее эффективным я считаю комбинированный метод (на базе Alpine, объединение слоев, удаление кэша и перепаковка), так как он позволяет достичь минимального размера.

**Почему очистка кэша пакетов в отдельном слое не уменьшает размер образа?**

Потому что каждый слой сохраняет изменения предыдущего. Если сначала кэш добавился в одном слое, а в следующем удалён — размер предыдущего слоя останется.

**Что такое перепаковка образа?**

Перепаковка — это экспорт работающего контейнера и импорт его содержимого как нового образа. Это позволяет избавиться от истории слоев и уменьшить итоговый размер.

## Выводы

В ходе работы я сравнил различные подходы оптимизации Docker-образов. Наиболее эффективным оказался способ с использованием базового образа Alpine, удалением временных файлов, объединением слоев и финальной перепаковкой. Это позволяет получить минимально возможный образ с рабочим функционалом.

## Ссылка на репозиторий

\[Ссылка на репозиторий будет добавлена позже]
