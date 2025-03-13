# АНАЛИЗ ДАННЫХ И ИСКУССТВЕННЫЙ ИНТЕЛЛЕКТ [in GameDev]
Отчет по лабораторной работе #2 выполнил(а):
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
Научиться передавать в Unity данные из Google Sheets с помощью Python.

## Задание 1
### Выбрать одну из игровых переменных в игре СПАСТИ РТФ: Выживание, описать её роль в игре. Построить схему экономической модели.
- В качестве игровой переменной возьмём количество патронов от револьера. Её роль - показывать, сколько выстрелов может сделать игрок из револьера до того, как придётся переключится на другое оружие.
	- Количество патронов уменьшается на единицу после каждого выстрела. Можно добавить 12 патронов для револьера, купив их в магазине за 20 монет.
	- Количество патронов может достигать любого неотрицательного значения.

![image](screenshots/Graph1.png)

- Патроны являются одним из ресурсов игровой экономики. Их нельзя добыть напрямую, но можно преобразовать из монет. Структура игровой экономики такова:
	- **Кран:** Краном (источником дохода) являются сами зомби и монеты, которые падают от них. Так как игроку всегда даётся нож в начале игры, у него всегда есть возможность атаковать зомби и поэтому источник дохода является неисчерпаемым.
	- **Инвентарь:** Инвентарём является запас монет и патронов у игрока.
	- **Труба:** Трубой (удалителем ресурсов) является оружие, тратящее патроны.
	- **Преобразователь:** Магазин практически является преобразователем монет в патроны и новое оружие.
	- **Торговец:** Торговец (отдельный субъект со своим инвентарём и ресурсами) в данной игре не присутствует.


## Задание 2
### Заполнить google-таблицу данными, описывающими выбранную игровую переменную. Описать характер изменения этой величины и недостатки.

- Первым делом создадим новый проект в Google Cloud для автоматического добавления данных в таблицу.
- Подключим Google Drive API и Google Sheets API
- Создадим сервисный аккаунт. Создадим на Диске новую Google таблицу и дадим этой учётной записи доступ.
- Теперь, c помощью python-скрипта, смоделируем то, как будет изменяться количество патронов на револьере с такими условиями:
	- Начальное количество патронов: 150;
	- Идеальный геймплей (каждый зомби убивается одним патроном через выстрел в голову);
	- От каждого убитого зомби падает ровно одна монета;
	- Игрок не использует и не покупает никакое другое оружие;
	- Игрок не тратит очки опыта на апгрейды;


```py

import gspread
import numpy as np
import time
gc = gspread.service_account(filename='unitydatascience-445819-53faf670fe6e.json')
sh = gc.open("UnityWorkshop2")
ammo = 150 # начальное количество патронов в револьере
money = 0 # начальное количество денег

i = 0
while ammo > 0:
    i += 1
    if i == 0:
        continue
    else:
        ammo -= 1
        money += 1
        if money >= 20: # если денег достаточно, покупаем патроны
            ammo += 12
            money -= 20
            
        sh.sheet1.update(('B' + str(i)), str(ammo))
        sh.sheet1.update(('C' + str(i)), str(money))
        
        time.sleep(2) # ограничивает частоту обращений по API
        print(ammo, money)

```

- Организуем полученные данные в таблицу

![image](screenshots/Screenshot3.png)

- Величина убывает почти линейно, скачнообразным образом. Здесь явно виден недостаток: даже при идеальной игре у игрока рано или поздно (где-то после 350 выстрелов) кончатся патроны, вынудив его использовать ближний бой, что на более сложных уровнях игры становится просто нереалистично. Тут есть несколько способов решить проблему:
	- **Поменять преобразвователь:** Банально уменьшить стоимость патронов, либо увеличись количество, получаемое при покупке. Чтобы рассчитать, насколько надо поменять стоимость, мы можем задать среднее количество выстрелов, которые мы хотим, чтобы игрок тратил на убийство одного зомби, а затем рассчитать отношение количества патронов к стоимости по формуле

	*(количество патронов при одной покупке) / (стоимость покупки) =  (среднее количество выстрелов для убийства)*
	- **Поменять кран:** Добавить дополнительное денежное вознаграждение за выстрелы в голову. Кроме решения проблемы с нехваткой патронов, это также даём игроку дополнительный стимул целиться в голову.
	- **Поменять трубу:** Поменять немного функционал оружия, добавив возможность, например, пробивать несколько врагов насквозь одним выстрелом, опять же делая геймплей более разнообразным, ппощряя игрока искать наиболее выгодные углы для выстрела.


## Задание 3
### Настроить на сцене Unity воспроизведение звуковых файлов, описывающих динамику изменения выбранной переменной.

- Создадим новый 3D Unity проект.
- Импортируем пакет со скриптами для парсинга JSON
- Импортируем звуки для выстрела и покупки
- Напишем скрипт, берущий данные из нашей таблицы с помощью API-ключа и воспроизводящий звуки при выстреле и покупке

```cs

using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.Networking;
using SimpleJSON;

public class NewBehaviourScript : MonoBehaviour
{
    public AudioClip shootSound;
    public AudioClip buySound;
    public AudioClip noammoSound;
    private AudioSource selectAudio;
    private Dictionary<string,float> dataSet = new Dictionary<string, float>();
    private bool statusStart = false;
    private int i = 1;

    // Start is called before the first frame update
    void Start()
    {
        StartCoroutine(GoogleSheets());
    }

    // Update is called once per frame
    void Update()
    {
        if (dataSet["Mon_" + i.ToString()] == 0 & statusStart == false & i < dataSet.Count & i != 1)
        {
            StartCoroutine(PlaySelectAudioBuy());
            Debug.Log(dataSet["Amm_" + i.ToString()] + ", " + dataSet["Mon_" + i.ToString()]);
        }

        if (statusStart == false & i < dataSet.Count)
        {
            StartCoroutine(PlaySelectAudioShoot());
            Debug.Log(dataSet["Amm_" + i.ToString()] + ", " + dataSet["Mon_" + i.ToString()]);
        }

        if (dataSet["Amm_" + i.ToString()] == 0 & statusStart == false & i < dataSet.Count)
        {
            StartCoroutine(PlaySelectAudioNoAmmo());
            Debug.Log(dataSet["Amm_" + i.ToString()] + ", " + dataSet["Mon_" + i.ToString()]);
        }
    }

    IEnumerator GoogleSheets()
    {
        UnityWebRequest curentResp = UnityWebRequest.Get("https://sheets.googleapis.com/v4/spreadsheets/1HgFcyuWOR8Or5w-nx4rLDTeGmjs4DMqWM-lF52owjuU/values/Sheet1?key=AIzaSyBeSCHVzx3KT7KahUpVr6eTx8ubD4SzOaU");
        yield return curentResp.SendWebRequest();
        string rawResp = curentResp.downloadHandler.text;
        var rawJson = JSON.Parse(rawResp);
        foreach (var itemRawJson in rawJson["values"])
        {
            var parseJson = JSON.Parse(itemRawJson.ToString());
            var selectRow = parseJson[0].AsStringList;
            dataSet.Add("Amm_" + selectRow[0].ToString(), float.Parse(selectRow[1], System.Globalization.CultureInfo.InvariantCulture));
            dataSet.Add("Mon_" + selectRow[0].ToString(), float.Parse(selectRow[2], System.Globalization.CultureInfo.InvariantCulture));
        }
    }

    IEnumerator PlaySelectAudioShoot()
    {
        statusStart = true;
        selectAudio = GetComponent<AudioSource>();
        selectAudio.clip = shootSound;
        selectAudio.Play();
        yield return new WaitForSeconds(0.1f);
        statusStart = false;
        i++;
    }
    IEnumerator PlaySelectAudioBuy()
    {
        statusStart = true;
        selectAudio = GetComponent<AudioSource>();
        selectAudio.clip = buySound;
        selectAudio.Play();
        yield return new WaitForSeconds(1.2f);
        statusStart = false;
        i++;
    }
    IEnumerator PlaySelectAudioNoAmmo()
    {
        statusStart = true;
        selectAudio = GetComponent<AudioSource>();
        selectAudio.clip = noammoSound;
        selectAudio.Play();
        yield return new WaitForSeconds(4);
        statusStart = false;
        i++;
    }
}

```

- Добавим скрипт на пустой объект с компонентом AudioSource.
- Результат:

![image](screenshots/Screenshot4.png)

## Выводы

Научились передавать в Unity данные из Google Sheets с помощью Python.

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
