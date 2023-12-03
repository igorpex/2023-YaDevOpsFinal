# Сборка Docker образа
Для сборки Docker образа в качестве дистрибутива использую облегченный distroless/base-debian12.

Но, так как там нет bash и утилит, то копирую их из busybox (mkdir, adduser)

В качестве начинки бинарный файл, дефолтный конфиг (см src/app и src/config).

  

билд

    docker build . -t registry.launc.ru/igorpex/bingo:1.2 

отправка в Registry:

    docker push registry.launc.ru/igorpex/bingo:1.2

 
затем уже Kuberentes будет брать образ оттуда

Делательнее - в комментариях Dockerfile