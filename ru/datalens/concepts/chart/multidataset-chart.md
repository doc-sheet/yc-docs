---
title: Мультидатасетные чарты
description: Из статьи вы узнаете, что такое мультидатасетные чарты, и ознакомитесь с их особенностями.
---

# Мультидатасетные чарты

Мультидатасетные чарты — чарты, которые отображают данные из разных датасетов.

Запросы для каждого датасета отрабатываются независимо друг от друга. Вы не можете создавать вычисляемые поля над полями из нескольких датасетов.
При добавлении второго датасета {{ datalens-short-name }} автоматически устанавливается связь по первому совпадению имени полей и типа данных полей.

При этом вы можете:

* изменять связи;
* добавлять новые связи;
* удалять связи.

{% note info %}

Датасеты в чарте могут быть не связаны.

{% endnote %}

Особенности работы со связанными датасетами в чарте, кроме случаев использования слоев в геочарте:

* в одном чарте могут быть использованы любые показатели из датасетов вне зависимости от связей;
* в одном чарте можно использовать только связанные измерения;
* фильтры по связанным измерениям применяются ко всем датасетам;
* фильтры по несвязанным измерениям применяются только к своему датасету.

Особенности работы со связанными датасетами в геовизуализациях на разных слоях:

* в одном геослое могут быть использованы любые показатели из датасетов вне зависимости от связей;
* в одном геослое можно использовать только связанные измерения;
* фильтры в секции **Общие фильтры** по связанным измерениям применяются ко всем датасетам во всех слоях;
* фильтры в секции **Общие фильтры** по несвязанным измерениям применяются только к своему датасету во всех слоях;
* фильтры в секции **Фильтры слоя** по связанным измерениям применяются ко всем датасетам в рамках текущего слоя;
* фильтры в секции **Фильтры слоя** по несвязанным измерениям применяются только к своему датасету в рамках текущего слоя;
* ограничения на использование несвязанных измерений в разных слоях не предусмотрены.

