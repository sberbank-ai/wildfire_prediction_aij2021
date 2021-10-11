NoFireWithAI: прогнозирование степени пожароопасности
=================================
[link to github](https://github.com/sberbank-ai/no_fire_with_ai_aij2021)

Соревнование алгоритмов, прогнозирующих пожароопасность региона на 8 дней вперёд. 

Участникам предлагается спрогнозировать пожар в определённом регионе России, используя различные, имеющиеся в открытом доступе, данные.

## Постановка задачи

Разобьём пространство на территории России на ячейки размером 0,2x0,2 градуса и для каждой ячейки поставим метку пересечения с полигоном пожара в соответствующий день. Таким образом, получим набор прямоугольников с меткой 1, если этот участок горел, и 0 в обратном случае.

Требуется разработать алгоритм, выдающий прогноз возникновения пожара (0 или 1) на 1, 2, 3, 4, 5, 6, 7 и 8 дней вперёд для заданной ячейки на определённую дату. Решение должно быть реализовано в виде программы, которая принимает на вход CSV таблицу со следующими столбцами:  
```
id - id строки для оценки предсказаний,
dt - дата,  
lon_min - минимальное значение долготы ячейки,  
lat_min - минимальное значение широты ячейки,  
lon_max - максимальное значение долготы ячейки,  
lat_max - максимальное значение широты ячейки, 
lon - долгота пожара, зафиксированного в ячейке на данную дату (пусто если пожара не было),  
lat - широта пожара, зафиксированного в ячейке на данную дату (пусто если пожара не было),  
grid_index - индекс ячейки,
type_id - тип пожара (пусто если пожара не было),  
type_name - расшифровка типа пожара (пусто если пожара не было).  
```

На выход необходимо сформировать таблицу, где для каждой ячейки (строки - `id` из входного файла) будут указаны 8 чисел (0 или 1), означающих наличие пожара через 1, 2, 3, 4, 5, 6, 7 и 8 дней соответственно. В проверяющую систему необходимо отправить код алгоритма, запакованный в ZIP-архив. Решения запускаются в изолированном окружении при помощи Docker.

В качестве дополнительных данных, доступных участникам как на момент обучения моделей, так и в момент инференса в проверяющей системе, могут быть использованы:

- Исходные данные по произошедшим пожарам - [train_raw.csv](https://dsworks.s3pd01.sbercloud.ru/aij2021/NoFireWithAI/train_raw.csv);
- Пример таблицы для которой требуется сформировать прогнозы пожаров - [sample_test.csv](https://dsworks.s3pd01.sbercloud.ru/aij2021/NoFireWithAI/sample_test.csv);
- Предобработанные исходные данные, описываемые в базовом решении - [train.csv](https://dsworks.s3pd01.sbercloud.ru/aij2021/NoFireWithAI/train.csv)
- Данные [openstreetmap](https://www.openstreetmap.org)  - [russia-latest.osm.pbf](https://dsworks.s3pd01.sbercloud.ru/aij2021/NoFireWithAI/russia-latest.osm.pbf)
- Данные по населённым пунктам РФ (https://wiki.openstreetmap.org/wiki/RU:Key:place) - [city_town_village.geojson](https://dsworks.s3pd01.sbercloud.ru/aij2021/NoFireWithAI/city_town_village.geojson)
 - Данные реанализа, полученные от Copernicus.eu ([ERA5](https://cds.climate.copernicus.eu/cdsapp#!/dataset/reanalysis-era5-land)  — данные реанализа "(2019): ERA5-Land hourly data from 1981 to present. Copernicus Climate Change Service (C3S) Climate Data Store (CDS)". ) Подробное содержание датасета приведено в [input/README.md](https://github.com/sberbank-ai/no_fire_with_ai_aij2021/blob/main/input/README.md)  

Также участники могут использовать для обучения моделей любые открытые источники данных, например спутниковые снимки и индексы пожароопасности. Однако стоит учитывать, что после загрузки решения в проверяющую систему у моделей не будет доступа в интернет и все необходимые для прогноза дополнительные данные нужно упаковать в docker-контейнер.


## Формат решения

В проверяющую систему необходимо отправить код алгоритма, запакованный в ZIP-архив. Решения запускаются в изолированном окружении при помощи Docker. Время и ресурсы во время тестирования ограничены. Участнику нет необходимости разбираться с технологией Docker.

### Содержимое контейнера

В корне архива обязательно должен быть файл metadata.json следующего содержания:
```json
{
    "image": "cr.msk.sbercloud.ru/aicloud-base-images-test/custom/aij2021/infire:f66e1b5f-1269",
    "entrypoint": "python3 /home/jovyan/solution.py"
}
```

Здесь `image` — поле с названием docker-образа, в котором будет запускаться решение, `entrypoint` — команда, при помощи которой запускается решение. Для решения текущей директорией будет являться корень архива. 

Для запуска решений можно использовать существующее окружение:

- `cr.msk.sbercloud.ru/aicloud-base-images-test/custom/aij2021/infire:f66e1b5f-1269` — [Dockerfile](https://github.com/sberbank-ai/no_fire_with_ai_aij2021/blob/main/Dockerfile) с описанием данного image и [requirements](https://github.com/sberbank-ai/no_fire_with_ai_aij2021/blob/main/requirements.txt) с библиотеками

Подойдет любой другой образ, доступный для загрузки из `sbercloud`. При необходимости, Вы можете подготовить свой образ, добавить в него необходимое ПО и библиотеки (см. [инструкцию по созданию Docker-образов для `sbercloud`](https://github.com/sberbank-ai/no_fire_with_ai_aij2021/blob/main/sbercloud_instruction.md)); для использования его необходимо будет опубликовать на `sbercloud`.

### Ограничения

В течение одного дня Участник или Команда Участников может загрузить для оценки не более пяти решений. Учитываются только валидные попытки, получившие численную оценку.  

Контейнер с решением запускается в следующих условиях:

- 94 Гб оперативной памяти;
- 3 vCPU;
- 1 GPU Tesla V100 32 Gb;
- время на выполнение решения: 30 минут;
- решение не имеет доступа к ресурсам интернета;
- максимальный размер упакованного и распакованного архива с решением: 10 Гб;
- максимальный размер используемого Docker-образа: 25 Гб.


## Проверка качества


Качество решения оценивается на отложенной выборке.  

Целью данного соревнования является прогнозирование начала пожара. Оценка как быстро этот пожар потушат зависит от многих факторов и не является приоритетной для данной задачи. Поэтому для расчёта метрики используются только метки начала пожара. После первой метки начала пожара (т. е. первой "1" в прогнозе) будем считать, что ячейка "горит" все оставшиеся дни:

![Corrected forecast](https://raw.githubusercontent.com/sberbank-ai/no_fire_with_ai_aij2021/main/input/burned_cells.png)


В общем виде формула расчёта метрики:
```
total_error = 1 - sum((C**(penalty_i / 16) - 1) / (C-1)) / N

penalty_i = (-2) * (sum(gt_corr_i) - sum(pred_corr_i)), штраф для i-ой строки, если пожар случился раньше, чем прогнозировался
penalty_i = (sum(gt_corr_i) - sum(pred_corr_i)), в ином случае

где N - количество строк в прогнозе,
С=5 - нормировочный коэффициент, 
pred_corr_i - скорректированный прогноз пожара для i-ой строки,
gt_corr_i - скорректированное действительное состояние i-ой строки.
```

Для финальной оценки можно выбрать 3 решения. По умолчанию это решения с наилучшей метрикой на public-лидерборде. Большее значение метрики соответствует лучшему результату (лучшее значение метрики → 1, худшее → 0). В случае одинакового значения метрики у двух или более участников приоритет имеет решение, загруженное в систему раньше.

## Призовой фонд

1 место - 500 000 рублей;  
2 место - 250 000 рублей;  
3 место - 150 000 рублей;

Дополнительно, есть возможность получить специальный приз - 100 000 рублей, который выдаётся на основании заключения экспертной комиссии, оценивающей применимость решения для прогнозирования опасных пожаров. Комиссия учитывает, но не ограничивается только ими, следующие факторы:
- интерпретируемость модели;
- возможность определения степени важности используемых признаков;
- общее количество признаков и скорость их подготовки;
- скорость работы модели и её размер.

Участники, претендующие на спец. приз должны:
- входить в топ-10 на лидерборде;
- предоставить воспроизводимое решение для обучения моделей, использовавшихся в их лучшей попытке.

##
[Пользовательское соглашение](https://api.dsworks.ru/dsworks-transfer/api/v1/public/file/terms_of_use.pdf/download)  
[Правила соревнования](https://api.dsworks.ru/dsworks-transfer/api/v1/public/file/rules.pdf/download)
