# Задание 1 — найди ошибки

В этом репозитории находятся материалы тестового задания "Найди ошибки" для [14-й Школы разработки интерфейсов](https://academy.yandex.ru/events/frontend/shri_msk-2018-2) (осень 2018, Москва, Санкт-Петербург, Симферополь).

Для работы тестового приложения нужен Node.JS v9. В проекте используются [Yandex Maps API](https://tech.yandex.ru/maps/doc/jsapi/2.1/quick-start/index-docpage/) и [ChartJS](http://www.chartjs.org).

## Задание

Код содержит ошибки разной степени критичности. Некоторые из них — стилистические, а другие — даже не позволят вам запустить приложение. Вам нужно найти все ошибки и исправить их.

Пункты для самопроверки:

1. Приложение должно успешно запускаться.
1. По адресу http://localhost:9000 должна открываться карта с метками.
1. Должна правильно работать вся функциональность, перечисленная в условиях задания.
1. Не должно быть лишнего кода.
1. Все должно быть в едином codestyle.

## Запуск

```
npm i
npm start
```

При каждом запуске тестовые данные генерируются заново случайным образом.

## Ход мыслей

Попыталась запустить проект. Выдал ошибку. 
По тексту ошибки оказалось, что у меня на `9000` порту уже был запущен php.

Разобравшись с процессами на порту, попыталась запустить вновь. Снова ошибка.
`"export 'default' (imported as 'initMap') was not found in './map'`
В этот раз оказалось, что неправильно настроен экспорт функции `initMap`.

"Ура, оно работает!" – подумала я, устранив ошибку и увидев `Compiled successfully.` в консоли.
Не тут то было! Белый экран браузера вернул меня на землю. "Вечер будет долгий..." – поняла я и продолжила поиски багов.
Пошла смотреть в консоль. Ошибок нет.
Залезла в DevTools. Оказалось контейнер карты схлопнулся до `0px` по высоте. 
Вернув контейнеру его высоту, наконец-то увидела Карту.

При этом места размещения базовых станций на ней отсутствовали.
Карта на месте, ошибок в консоли нет. Пошла разбираться, а как же они вообще должны отрисовываться?
Вычитывая API Карт, пробиралась всё глубже и глубже. В какой-то момент я и вовсе потеряла счёт времени.
И тут стало ясно – в функции `initMap` при инициализации карты, мы никак не связываем `objectManager` с самой картой.
После добавления соответствующего кода, пины базовых станций отобразились на карте.

Но все они были на севере Ирана.
Проверив получаемые данные о пинах, сравнив их с координатами Москвы, было ясно – где-то перепутана широта и долгота.
Поразбиравшись немного в процессах в коде, нашла где эти данные заполняются и поняла, что нужно поменять местами.

Ура! Карта на месте, пины на карте, кластеры сворачиваются, но внутри себя не учитывают станции `defective`.
Снова забурилась в API смотреть, что же такое кластеры и с чем их едят.
Разобравшись, как задавать кластеризацию и её параметры, выяснилось, что в коде перетирается то, что уже работало по-умолчанию, заданием в опциях `'preset', 'islands#greenClusterIcons'`. 
Исправила.

Из багов осталось только открытие `ballon`, при клике на станцию.
В тот момент он открывался, но пустой и в левом верхнем углу.
Сначала искала ошибки в обработчике клика по станции. Прошерстила кучу API. В обработчике всё верно.
Потом искала проблемы в заполнении самого `ballon` данными станции.
Оказалось, что при задании методов `build` и `clear` для шаблона использовались стрелочные функции.
Из-за этого `this` ссылалось не туда при вызове родительских методов.

Ура! `ballon` открывается, но не отрисовывается график внутри. 
Поизучав настройки ChartJS, нашла лишнее добавление параметра `max: 0`.
После его устранения, обновив страницу браузера, поняла, что баги закончились!

Весь описанный в ТЗ функционал заработал и я смогла со спокойной душой переходить к следующему испытанию!
