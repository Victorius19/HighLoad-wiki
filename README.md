# Тема и целевая аудитория
## Тема
[Wikipedia](https://wikipedia.org/) — общедоступная многоязычная универсальная интернет-энциклопедия со свободным контентом, реализованная на принципах вики.

## Продуктовые метрики
Википедия - это один из самых посещяемых сайтов в мире. Он имеет огромные месячные и дневные показатели статистики [[1]](https://www.semrush.com/website/wikipedia.org/overview/)[[2]](https://www.similarweb.com/ru/website/wikipedia.org/):
| Метрика                               | Показатель |
| ------------------------------------- | ---------- |
| Посещений в месяц                     | 6.2 млрд   |
| Посещений в день                      | 200 млн    |
| Средняя продолжительность посещения   | 03:52      |
| Среднее количество посещенных страниц | 3.13       |

Процент пользователей Википедии из стран [[3]](https://www.similarweb.com/ru/website/wikipedia.org/):
| Страна            | Процент пользователей |
| ----------------- | --------------------- |
| Соединенные Штаты | 24.57%                |
| Япония            | 6.45%                 |
| Великобритания    | 5.56%                 |
| Германия          | 5.23%                 |
| Россия            | 4.67%                 |
| Другие            | 53.52%                |

## Расчет нагрузки
В Википедии есть несколько основных направлений использования сайта, способных генерировать нагрузку. В основном это чтение статей, т.к. большая часть пользовтелей википедии никогда не задумываются о их редактировании, однако часто пользуются, читая информацию. Так же часть пользователей создают аккаунты и редактируют статьи, однако это нагрузка уже гораздо менее существенна, чем просмотр статей.

Информация, используемая для расчета следующих показателей:
| Тип                                                                                                                                                                   | Показатель |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ---------- |
| Дневное количество пользователей                                                                                                                                               | 200 млн    |
| Месячное количество пользователей                                                                                                                                              | 6.2 млрд   |
| Срденее время просмотра страницы                                                                                                                                               | 3:52 мин       |
| Среднее количество посещенных страниц                                                                                                                                          | 3.13      |
| Средний размер страницы                                                                                                                                                        | 500 Кб     |
| Количество редактирований статей в месяц [[4]](https://stats.wikimedia.org/#/en.wikipedia.org/contributing/edits/normal\|bar\|all\|~total\|monthly)                            | 5 млн      |
| Создание и редактирование аккаунтов [[5]]( https://stats.wikimedia.org/#/en.wikipedia.org/contributing/user-edits/normal\|bar\|all\|(page_type)~content*non-content\|monthly ) | 3 млн      |
| Общее количество страниц [[6]]( https://stats.wikimedia.org/#/en.wikipedia.org/contributing/new-pages/normal\|bar\|all\|~total\|monthly )                                      | 44 млн     |

### Показатели RPS

#### Расчет нагрузки генерируемой просмотром статей

Большая часть пользователей википедии не редактируют страницы, а только просматривают их
200 млн пользователей в день в среднем находятся на странице 3:52 минуты и посещают 3.13 страницы, т.е, один пользователь генерирует
`200 000 000 * 3.13 / (60 * 60 * 24) = 7245 RPS`

#### Расчет нагрузки генерируемой редактированием

В среднем в месяц статьи редактируются 5 млн раз. Следовательно генерируемая нагрузка
`5 000 000 / (60 * 60 * 24 * 30) = 1.92 RPS`

#### Расчет нагрузки генерируемой действиями с аккаунтами

В среднем в месяц создается и редактируется 3 млн пользователей. Следовательно генерируемая нагрузка
`3 000 000 / (60 * 60 * 24 * 30) = 1.15 RPS`

| Тип нагрузки                            | RPS      |
| --------------------------------------- | -------- |
| Просмотр статей                         | 7245 RPS |
| Редактирование и создание статей        | 1.92 RPS |
| Редактирование и создание пользователей | 1.15 RPS |

### Сетевая нагрузка и хранилище

Средний размер страницы 500 Кб, а количество просмотров страниц в секунду 7245, следовательно исходящий трафик:
`500 Кб * 7245 = 3.625 Гб/сек`

В википедии 44 млн страниц, а средний вес страницы включая медиа файлы 500 Кб. Количество пользователей и их данные несущественное количество информации. Следовательно можно посчитать размер хранилища
`44 000 000 * 500 Кб = 22 Тб` 

## Логическая схема базы данных
![wiki-rpz](https://user-images.githubusercontent.com/47574124/204141242-50664ac1-f727-4dd2-8620-49211efdc23b.png)
## Физическая схема базы данных
![image](https://user-images.githubusercontent.com/47574124/204141656-0f529918-e60d-4c53-b64c-34f0a0103355.png)
Медиа файлы - наибольший по объему тип информации, который придется хранить, текст статей и юзеры будут занимать относительно малое количество информации. Поэтому было решено вынести файлы на amazon S3 с хранением кеша и со связью многие ко многих к статьям, чтобы не дублировать уже загруженные файлы, а также чтобы не хранить файлы, которые уже не используются в статьях.

Статьи содержат два поля id и theme_id, чтобы была возможность логически связать статьи на разных языках относящиеся к разным темам.

## Технологии
| Технологии | Область применения | Мотивационная часть |
|-|-|-|
| TypeScript + SvelteJS + Routify | Frontend | Популярный стэк технологий, который позволит быстро разработать удобное веб-приложения, которое будет мало весить для пользователя. |
| Go | Backend | Высокая производительность и простая разработка. |
| Redis | Хранение сессий + кеш запросов на статьи | Быстрое key value хранилище, возможность простой работы с go и с nginx |
| Amazon S3 | Backend | Высокая производительность, масштабируемость, высокая надежность, низкие затраты |
| HTTP Redis2 module for nginx | кеширование статей | Модуль для nginx, который позволит закешировать в redis популярные статьи на небольшой промежуток времмени |
| Prometheus + Grafana | Backend | Мониторинг сервисов |

Backend обязательно разбить на микросервисы. Выделяю три микросервиса: работа с получением статей из БД, работа с пользователями, работа с вложениями (attaches) и работа с изменением статей. Это необходимо из-за огромного различия в нагрузке на просмотр статей и на работа с юзерами и изменением статей. Просмотр статей гораздо в несколько тысяч раз имеет больше RPS, поэтому микросервис по выгрузке статей из бд будет иметь гораздо больше нод.
