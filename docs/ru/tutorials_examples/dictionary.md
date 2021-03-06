# Как работает словарь SubQuery?

Вся идея проекта универсального словаря состоит в том, чтобы индексировать все данные из цепочки блоков и записывать события, внешние данные и их типы (модуль и метод) в базу данных в порядке высоты блока. Затем другой проект может запросить эту конечную точку ` network.dictionary ` вместо значения по умолчанию ` network.endpoint `, определенного в файле манифеста.

Конечная точка ` network.dictionary ` - это необязательный параметр, который, если он присутствует, SDK автоматически обнаружит и использует. ` network.endpoint ` является обязательным и не будет компилироваться, если он отсутствует.

Взяв в качестве примера проект [ SubQuery dictionary ](https://github.com/subquery/subql-dictionary), файл [ schema ](https://github.com/subquery/subql-dictionary/blob/main/schema.graphql) определяет 3 объекта; внешние, события, specVersion. Эти 3 объекта содержат 6, 4 и 2 поля соответственно. При запуске этого проекта эти поля отражаются в таблицах базы данных.

![extrinsics table](/assets/img/extrinsics_table.png) ![events table](/assets/img/events_table.png) ![specversion table](/assets/img/specversion_table.png)

Затем данные из цепочки блоков сохраняются в этих таблицах и индексируются для повышения производительности. Затем проект размещается в проектах SubQuery, и конечная точка API доступна для добавления в файл манифеста.

## Как включить словарь в свой проект?

Добавьте ` dictionary: https://api.subquery.network/sq/subquery/dictionary-polkadot ` в сетевой раздел манифеста. Например:

```shell
network:
  endpoint: wss://polkadot.api.onfinality.io/public-ws
  dictionary: https://api.subquery.network/sq/subquery/dictionary-polkadot
```

## Что происходит, если словарь НЕ используется?

Когда словарь НЕ используется, индексатор будет извлекать данные каждого блока через api polkadot в соответствии с флагом ` batch-size `, который по умолчанию равен 100, и помещает их в буфер для обработки. Позже индексатор берет все эти блоки из буфера и при обработке данных блока проверяет, соответствуют ли событие и внешний вид в этих блоках заданному пользователем фильтру.

## Что происходит, когда словарь используется?

Когда используется словарь, индексатор сначала принимает фильтры вызовов и событий в качестве параметров и объединяет их в запрос GraphQL. Затем он использует API словаря для получения списка только соответствующих высот блоков, который содержит конкретные события и внешние элементы. Часто это существенно меньше 100, если используется значение по умолчанию.

Например, представьте себе ситуацию, когда вы индексируете события передачи. Не все блоки имеют это событие (на изображении ниже нет событий передачи в блоках 3 и 4).

![dictionary block](/assets/img/dictionary_blocks.png)

Словарь позволяет вашему проекту пропустить это, поэтому вместо того, чтобы искать в каждом блоке событие передачи, он пропускает только блоки 1, 2 и 5. Это потому, что словарь является предварительно вычисленной ссылкой на все вызовы и события в каждом блоке.

Это означает, что использование словаря может уменьшить объем данных, которые индексатор получает из цепочки, и уменьшить количество «нежелательных» блоков, хранящихся в локальном буфере. Но по сравнению с традиционным методом он добавляет дополнительный шаг для получения данных из API словаря.

## Когда словарь НЕ полезен?

Когда [ обработчики блоков ](https://doc.subquery.network/create/mapping.html#block-handler) используются для захвата данных из цепочки, каждый блок должен быть обработан. Следовательно, использование словаря в этом случае не дает никаких преимуществ, и индексатор автоматически переключится на не словарный подход по умолчанию.

Кроме того, при работе с событиями или внешними событиями, которые происходят или существуют в каждом блоке, таком как ` timestamp.set `, использование словаря не даст никаких дополнительных преимуществ.
