# АНАЛИЗ ДАННЫХ И ИСКУССТВЕННЫЙ ИНТЕЛЛЕКТ [in GameDev]
Отчет по лабораторной работе #3 выполнил(а):
- Маврешко Тимофей Кириллович
- РИ230940
Отметка о выполнении заданий (заполняется студентом):

| Задание | Выполнение | Баллы |
| ------ | ------ | ------ |
| Задание 1 | * | 60 |
| Задание 2 | * | 20 |
| Задание 3 | * | 20 |

знак "*" - задание выполнено; знак "#" - задание не выполнено;

Работу проверили:
- к.т.н., доцент Денисов Д.В.
- к.э.н., доцент Панов М.А.
- ст. преп., Фадеев В.О.

[![N|Solid](https://cldup.com/dTxpPi9lDf.thumb.png)](https://nodesource.com/products/nsolid)

[![Build Status](https://travis-ci.org/joemccann/dillinger.svg?branch=master)](https://travis-ci.org/joemccann/dillinger)

Структура отчета

- Данные о работе: название работы, фио, группа, выполненные задания.
- Цель работы.
- Задание 1.
- Код реализации выполнения задания. Визуализация результатов выполнения (если применимо).
- Задание 2.
- Код реализации выполнения задания. Визуализация результатов выполнения (если применимо).
- Задание 3.
- Код реализации выполнения задания. Визуализация результатов выполнения (если применимо).
- Выводы.
- ✨Magic ✨

## Цель работы
Разработать оптимальный баланс изменения сложности для десяти уровней игры Dragon Picker.

## Задание 1
### Предложить вариант изменения найденных переменных для 10 уровней в игре. Визуализировать изменение уровня сложности в таблице.
Ход работы:
- В игре Dragon Picker три переменные играют наибольшую роль в определении сложности игры:
	- **Скорость:** Как понятно из названия, влияет на скорость передвижения дракона по экрану.
	- **Вероятность поворота:** Влияет на вероятность дракона сменить направление движения каждый кадр.
	- **Ожидание между падениями яиц:** Время (в секундах), которое проходит между падением каждого яйца.
- Сложность игры зависит от баланса скорости и вероятности поворота. Если вероятность поворота слишком низкая по сравнению со скоростью, то дракон будет двинаться слишком предсказуемо. Если же, наоборот, вероятность поворота слишком высокая, то дракон будет как бы "крутиться на месте", не побуждая игрока слишком активно управлять корзиной.
- Для упрощения задачи, сделаем так, чтобы отношение скорости к вероятности поворота было постоянным. Это значительно упростит формулу вычисления уровня сложности. Сложность уровня *прямо пропорционально* скорости.
- Также нужно обратить внимание на ожидание между падениями яиц. Чем чаще падают яйца, тем меньше у игрока времени, чтобы на них среагировать. Сложность уровня *обратно пропорционально* времени ожидания.
- Теперь у нас готов возможный вариант формулы для изменения уровня сложности:

*(сложность)=(скорость)/(время ожидания между яйцами)*

- Тем самым, если мы будет одновременно *линейно* изменять скорость и время ожидания, то сложность по нашей формуле будет изменяться *квадратично*. Таким образом, у нас готов алгоритм для определения значений переменных для каждого уровня:

```py

import gspread
import numpy as np
import matplotlib.pyplot as plt
import time

def lerp(a, b, dist):
    return a*(1-dist) + b*dist 

LEVEL_COUNT = 10 # количество уровней
START_SPEED = 4 # скорость дракона
END_SPEED = 40
START_TURNFREQ = 0.01 # вероятность поворота
END_TURNFREQ = START_TURNFREQ * (END_SPEED / START_SPEED) # для простоты мы делаем вероятность поворота прямо пропорциональным скорости
START_EGGDELAY = 2 # время ожидания между падением яиц
END_EGGDELAY = 0.5

gc = gspread.service_account(filename='unitydatascience-445819-53faf670fe6e.json')
sh = gc.open("UnityWorkshop3")

level = list(range(LEVEL_COUNT))
lerp_dists = [i*0.1 for i in level]
speeds = [lerp(START_SPEED, END_SPEED, i) for i in lerp_dists] # линейно рассчитываем переменные для каждого уровня
turnfreqs = [lerp(START_TURNFREQ, END_TURNFREQ, i) for i in lerp_dists]
eggdelays = [lerp(START_EGGDELAY, END_EGGDELAY, i) for i in lerp_dists] 
difficulties = [speeds[i] / eggdelays[i] for i in level] # вычисляем сложность как прямо пропорциальную скорости и обратно пропорциональную времени ожидания между яйцами

plt.plot(level, speeds, label="Скорость")
plt.plot(level, turnfreqs, label="Вероятность поворота")
plt.plot(level, eggdelays, label="Ожидание между яйцами")
plt.plot(level, difficulties, label="Сложность")

plt.legend(["Скорость", "Вероятность поворота", "Ожидание между яйцами", "Сложность"])
plt.show() # все переменные возрастают линейно, а сложность - квадратично

for i in level:
    sh.sheet1.update(('A' + str(i+1)), str(i+1))
    sh.sheet1.update(('B' + str(i+1)), str(speeds[i]))
    sh.sheet1.update(('C' + str(i+1)), str(turnfreqs[i]))
    sh.sheet1.update(('D' + str(i+1)), str(eggdelays[i]))
    sh.sheet1.update(('E' + str(i+1)), str(difficulties[i]))
    time.sleep(1) # ограничивает частоту обращений по API

```
![image](screenshots/Graph2.png)

- Результат заполнения таблицы:


![image](screenshots/Screenshot5.png)

## Задание 2
### Создайть 10 сцен на Unity с изменяющимся уровнем сложности.

- Создадим новый скрипт, читающий данные из Google таблицы и записывающий их в переменные в зависимости от уровня.

```cs

using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.Networking;
using SimpleJSON;

public class DifficultySetter : MonoBehaviour
{
    public int level = 0;
    private EnemyDragon dragon;

    // Start is called before the first frame update
    void Start()
    {
        dragon = GetComponent<EnemyDragon>();
        StartCoroutine(GoogleSheets());
    }

    IEnumerator GoogleSheets()
    {
        UnityWebRequest curentResp = UnityWebRequest.Get("https://sheets.googleapis.com/v4/spreadsheets/1HNPKYvkw2i7e8i0SZcKDJR0AalKUKrLWyiiisD8kBH4/values/Sheet1?key=AIzaSyBeSCHVzx3KT7KahUpVr6eTx8ubD4SzOaU");
        yield return curentResp.SendWebRequest();
        string rawResp = curentResp.downloadHandler.text;
        var rawJson = JSON.Parse(rawResp);
        var itemRawJson = rawJson["values"][level];

        var parseJson = JSON.Parse(itemRawJson.ToString());
        var selectRow = parseJson;
        Debug.Log(rawJson);
        Debug.Log(selectRow[1] + ", " + selectRow[2] + ", " + selectRow[3]);
        dragon.speed = float.Parse(selectRow[1], System.Globalization.CultureInfo.InvariantCulture);
        dragon.chanceDirection = float.Parse(selectRow[2], System.Globalization.CultureInfo.InvariantCulture);
        dragon.timeBetweenEggDrops = float.Parse(selectRow[3], System.Globalization.CultureInfo.InvariantCulture);
    }

}

```

- Прикрепим скрипт к дракону и скопируем сцену 10 раз.
- Для каждой сцены изменим параметр level от 0 до 9.
- Теперь у нас есть плавное увеличение сложности между уровнями.

## Задание 3
### Решение в 80+ баллов должно визуализировать данные из google-таблицы, и с помощью Python передавать в проект Unity. В Python данные также должны быть визуализированы.

- Это, собственно, и было сделано. Визуализация данных показана в ходе решения задания 1, а передача данных в Unity показана в ходе решения задачи 2.

## Выводы

Разработали оптимальный баланс изменения сложности для десяти уровней игры Dragon Picker.

Все созданные в ходе работы программы доступны в этой репозитории в папке workshop-projects.

| Plugin | README |
| ------ | ------ |
| Dropbox | [plugins/dropbox/README.md][PlDb] |
| GitHub | [plugins/github/README.md][PlGh] |
| Google Drive | [plugins/googledrive/README.md][PlGd] |
| OneDrive | [plugins/onedrive/README.md][PlOd] |
| Medium | [plugins/medium/README.md][PlMe] |
| Google Analytics | [plugins/googleanalytics/README.md][PlGa] |

## Powered by

**BigDigital Team: Denisov | Fadeev | Panov**
