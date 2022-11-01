# Credit-Scoring
Прогнозирование вероятности дефолта заемщика

Соревнование kaggle (Rus_Salih_b) Top-4%: https://www.kaggle.com/competitions/sf-dst-scoring

Задача: Классификация

## Требования
Python 3.7.6

Зависимости: requirements.txt

###
## Сборка
    git clone https://github.com/salih-ds/credit-scoring.git

## Данные и описание полей
https://www.kaggle.com/competitions/sf-dst-scoring/data

## Обзор
### Данные
- Столбцы заполнены корректно, пропуски отсутствуют
- Значительная доля заемщиков не дефолтные

### Разведочный анализ (EDA)
- Чаще заемщиками являются клиенты 25-35
- Большинство не имеет отказных заявок, но есть небольшое число клиентов с высоким количеством отказов, отказы смещены влево
- Оценка bki распределена нормально
- Доход смещен влево, есть выбросы с крайне высоким уровнем дохода
- Половина клиентов имеет 0 или 1 запрос в БКИ, данные смещены влево
- Крайне мало клиентов из регионов с рейтиногом 30 и ниже
- Молодые более склонны к совершению дефолта
- Дефолт совершают люди, которые имеют более высокое значение скоринговой оценки
- Платежеспособные люди живут, как правило, в регионах с более высоким рейтингом
- У совершающих дефолт доход чуть ниже
- Количество запросов в БКИ, обычно, выше у совершающих дефолт
- Люди с более высоким образованием имеют, как правило, более высокий доход
- Признаки "домашний" и "рабочий" адресы имеют высокую зависимость - большинство выбирает работу рядом с домом
- Нет признаков, сильно коррелирующих с целевой переменной

### Feature engineering
Сгенерируем признаки

int:
- app_date - cколько дней прошло с 1-го запроса
- bki/decline - количество отказных заявок/количество запросов
- normalized_income_minus_mean - нормализованный доход по категории возраста
- income_per_score_bki - доход/скоринг оценку

binary:
- home_work - дом далеко от работы
- inc_large_mean - доход больше среднего по категории возраста
- request_large_mean - больше среднего обращений в БКИ по категории возраста
- inc_large_mean_region - доход больше среднего по региону
- decline_app_cnt - без отказов от банков, но имеет обращения в БКИ
- decline_app_cnt - не имеет запросов в БКИ и не имеет отказов
- good_work - имеет доход выше среднего, но не имеет пометку "хорошая" работа (бизнес?)

category:
- age_category - категории возрастов

### F-test
оценим зависимость признаков и целевой переменной

Числовые признаки:
![image](https://user-images.githubusercontent.com/73405095/196608892-0424712f-9587-4b3c-93f5-650867680f80.png)

Бинарные и категориальные признаки:
![image](https://user-images.githubusercontent.com/73405095/196608930-dcd32f7c-eff3-4b95-8631-49791851d27a.png)

### Подготовка данных на вход модели
- логарифмируем числовые признаки для борьбы с выбросами
- перекодируем текстовые столбцы с 2-мя уникальными значениями в бинарные признаки (LabelEncoder)
- перекодируем категориальные признаки (get_dummies)

### Обучим наивную модель
![image](https://user-images.githubusercontent.com/73405095/196611369-f61ed44f-d2d8-414b-870f-d375306a12c8.png)
![image](https://user-images.githubusercontent.com/73405095/196611989-ed60a17e-1679-4b96-8dbc-237edb4cbc02.png)

precision_score: 0.4057971014492754

recall_score: 0.030026809651474532

f1_score: 0.05591612581128308

**Результат:**
- Модель имеет низкие показатели качества
- Модель предсказывает, что практически все не дефолтные

### Протестируем классические алгоритмы
Разделим размеченные данные на train и test 80/20

**SVC**

Всех обозначил не дефолтными

**KNeighborsClassifier**
- precision_score: 0.27176220806794055
- recall_score: 0.06863270777479893
- f1_score: 0.10958904109589042

**MLPClassifier**
- precision_score: 0.32840236686390534
- recall_score: 0.05951742627345844
- f1_score: 0.10077167498865182

**XGBClassifier**
- precision_score: 0.3561643835616438
- recall_score: 0.0418230563002681
- f1_score: 0.07485604606525913


## Тюнинг моделей и оценка предиктов
Разделяю выборку на 3 фолда и рандомно выбираю параметры из списка параметров, отправляю предикт лучшей модели для теста 

**XGBClassifier**
- Sub_result: 0.74268

**MLPClassifier**
- Sub_result: 0.73829

**KNeighborsClassifier**
- Sub_result: 0.62991

