# Course_Work
=
# Использование методов машинного обучения для отображения пространства сигналов ЭМГ мышц лица в пространство эмоций человека.
=
### Постановка задачи:
Необходимо с помощью методов машинного обучения изучить возможность отображения пространства сигналов ЭМГ мышц лица в пространство эмоций человека.
### Описание данных:
Набор данных, используемый в задаче, находится в файле "data_vad.csv". Он содержит в себе 165165 строк и 8 признаков, один из которых в 
задаче не используется. Признаки "msec", "label", "batch_id" являются вспомогательными и напрямую в отображении пространств не участвуют.
Признаки "Mas", "Zyg", "Corr" составляют пространство сигналов ЭМГ мышц лица, а "Valence" и "Arousal" - пространство эмоций человека. 
### Ход работы:
-Предобработка:
   Поскольку "Arousal" и "Valence" содержат 3632 не пустых строк, то в первом приближении я удалил все пропуски. В дальнейшей работе был
   использован дата фрейм с 7 признаками и 3632 строками.
   Были рассмотрены:
   1. Распределения по отдельным признакам.
          + Распределения для "Arousal", "Zyg", "Mas", "Corr" смещены влево относительно нормального распределения, что может
          свидетельствовать о выбросах.        
          + Распределения "Valence" и "msec" близки к нормальному.
   2. Корреляции между признаками.
          Наблюдаются небольшие корреляции между следующими парами признаков:
              - Corr и Arousal
              - Zyg и Mas
              - Zyg и Valence
   3. Графики boxplot по отдельным признакам в зависимости от эксперимента.(Идентификатор эксперимента - 'label')
          + Графики подтвердили выбросы в данных.
          + Меньше всего выбросов наблюдается в 3 эксперименте.
          + Значения медианы для отдельных признаков в различных экспериментах отличаются друг от друга. 
   4. Автокорреляции на отдельных сигналах.(Идентификатор сигнала - "batch_id")
          В среднем автокорреляции в сигнале уменьшаются в exp раз:
              - Для "Corr" на 6 шаге.
              - Для "Mas" на 5 шаге.
              - Для "Zyg" на 6 шаге.
   С помощью межквартильного размаха были детектированы отдельные сигналы в которых наблюдались выбросы. Визуализация выбросов 
   показала, что программа не учитывает расширения диапазона сигнала и детектирует его, как объект с выбросами. Попытка удаления
   выбросов не улучшило распределения переменных, поэтому в дальнейшей работе продолжим использовать дата фрейм описанный выше.
-Разделение данных:
     Созданный дата фрейм был разделен на 2 выборки, тренировочную, которая содержит 70% данных, и отложенную - 30%. Разделение учитывает, 
     как особенность каждого эксперимента, так и то что данные содержатся в различных сигналах.
-Применение моделей машинного обучения для отображения пространств:
     Сначала были применены 12 моделей "из коробки":
         -DummyRegressor
         -LinearRegression
         -BayesianRidge
         -ElasticNet
         -SVR
         -KNeighborsRegressor
         -DecisionTreeRegressor
         -RandomForestRegressor
         -GradientBoostingRegressor
         -CatBoostRegressor
         -LGBMRegressor
         -XGBRegressor
     После чего с помощью групповой кросс-валидации(GroupKFold) и сетки(GridSearchCV) были подобраны параметры моделей. 
-Сравнение моеделей до и после подбора параметров:
     Сравнение качества моделей проводилось с помощью метрики $R^2$, средней квадратической ошибкой(MSE) и средней абсолютной ошибкой(MAE).
     Лучшая модель для предсказания "Valence": *Случайный лес с глубиной деревьев равной 4 и количеством деревьев равным 100*   
     Лучшая модель для предсказания "Arousal": *Метод k ближайших соседей с количеством ближайших соседей равным 102"
-Новое приближение:
     -В новом приближении я создаю дата фрейм погружение, в котором помимо моментальных значений "Corr", "Mas", "Zyg" учитываются также
     значения этих признаков за прошлые моменты времени. Погружение производится на среднее количество шагов при котором корреляция падает в
     exp раз для каждого признака в отдельности.
     Новый дата фрейм состоит из 25 признаков и 2654 строк. 
     -По аналогии с первым приближением были поделены новые данные, применены 12 моделей "из коробки" и подобраны их параметры. После чего
     было проведено общее сравнение всех моделей из обоих случаев.
-Общее сравнение(интерпретация результатов):
     -Ни одна из моделей в полной мере не отображает пространство сигналов ЭМГ мышц лица в пространство эмоций человека.
     -Поскольку есть случаи в которых после подбора параметров оценка качества модели $R^2$ снизилась, то непонятно как проводить сравнение
     моделей? И корректно ли их вообще сравнивать? Хотя есть заметные улучшения и большинство моделей с погружением показывают себя лучше,
     чем аналогичные им без него.           