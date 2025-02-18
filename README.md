# Оценка риска ДТП  

### Описание проекта  
От каршеринговой компании поступил заказ: нужно создать систему, которая могла бы оценить риск ДТП по выбранному маршруту движения. Под риском понимается вероятность ДТП с любым повреждением транспортного средства. Как только водитель забронировал автомобиль, сел за руль и выбрал маршрут, система должна оценить уровень риска. Если уровень риска высок, водитель увидит предупреждение и рекомендации по маршруту.  
Идея создания такой системы находится в стадии предварительного обсуждения и проработки. Чёткого алгоритма работы и подобных решений на рынке ещё не существует. Текущая задача — понять, возможно ли предсказывать ДТП, опираясь на исторические данные одного из регионов.  
Идея решения задачи от заказчика:  
1. Создать модель предсказания ДТП (целевое значение — at_fault (виновник) в таблице parties)  
    o Для модели выбрать тип виновника — только машина (car).  
    o Выбрать случаи, когда ДТП привело к любым повреждениям транспортного средства, кроме типа SCRATCH (царапина).  
    o Для моделирования ограничиться данными за 2012 год — они самые свежие.  
    o Обязательное условие — учесть фактор aа автомобиля.  
2.	На основе модели исследовать основные факторы ДТП.  
3.	Понять, помогут ли результаты моделирования и анализ важности факторов ответить на вопросы:   
o	Возможно ли создать адекватную системы оценки водительского риска при выдаче авто?  
o	Какие ещё факторы нужно учесть?  
o	Нужно ли оборудовать автомобиль какими-либо датчиками или камерой?  

Заказчик предлагает поработать с базой данных по происшествиям и сформировать свои идеи создания такой системы.  

### Краткое описание таблиц  
• **collisions** — общая информация о ДТП   
Имеет уникальный `case_id`. Эта таблица описывает общую информацию о ДТП. Например, где оно произошло и когда.   
• **parties** — информация об участниках ДТП   
Имеет неуникальный `case_id`, который сопоставляется с соответствующим ДТП в таблице collisions. Каждая строка здесь описывает одну из сторон, участвующих в ДТП. Если столкнулись две машины, в этой таблице должно быть две строки с совпадением case_id. Если нужен уникальный идентификатор, это case_id and party_number.   
• **vehicles** — информация о пострадавших машинах  
Имеет неуникальные `case_id и неуникальные `party_number`, которые сопоставляются с таблицей collisions и таблицей parties. Если нужен уникальный идентификатор, это `case_id` and `party_number`.  


### Первичное исследование таблиц:  
- все загруженные таблицы имеют набор данных;  
- количество таблиц и типы данных соответствуют условию задачи;  
- таблица `case_ids` содержит данные только за 2021-й год.  

### Статистический анализ факторов ДТП:  
- осень и весна – сезоны с наибольшим числом аварий, декабрь также находится в ледерах по количеству ДТП;  
- если автомобилю от трёх до восьми лет, то вероятность оказаться виновником ДТП снижается по мере увеличения возраста транспортного средства,  
от восьми лет – возрастает;  
- неблагоприятные погодные условия негативно влияют на вероятность стать виновником ДТП.    

### Предобработка данных:  
- обработка пропусков:  
    - `direction`, `location_type`, `party_sobriety`, - заполнены значением `unknown`;  
    - `vehicle_transmission` - значением `auto`;  
    - `party_drug_physical` - заполнены значением `not applicable`;    
    - `cellphone_in_use` - заполнены значением `1`;    
    - `vehicle_age` - заполнены модой в зависимости от географического района, типа кузова и типа КПП;    
    - `insurance_premium` - заполнены модой в зависимости от географического района, типа кузова, типа КПП и возраста автомобиля;    
    - пропуски в остальных столбцах были удалены;    
- в переменных `distance`, ` vehicle_age ` и ` insurance_premium ` удалены выбросы и аномалии;  
- после предобработки сохранилось примерно 97% от изначального объема данных.  

### Создание модели для оценки водительского риска:  
- подобраны гиперпараметры для моделей случайного леса, градиентного бустинга и логистической регрессии;  
- на кросс-валидации лучший результат продемонстрировал `CatBoostClassifier`;  
- исходя из поставленной задачи, было принято решение понизить порог классификации, для того чтобы увеличить охват потенциальных виновников ДТП;  
- значение метрик, полученные на тестовой выборке:  
    - `roc_auc` = 73;  
    - `f1` = 68;  
    - `precision` = 60;  
    - `recall` = 78.  
 
Для создания адекватной системы оценки риска при выдаче авто необходимы данные о случаях, которые не привели к ДТП. Также в полученных датасетах в большей степени содержится информация о транспортных средствах и обстоятельствах аварии и не достаёт информации о клиенте.  
