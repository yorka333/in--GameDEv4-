# -in-GameDev4-
# АНАЛИЗ ДАННЫХ И ИСКУССТВЕННЫЙ ИНТЕЛЛЕКТ [in GameDev]
Отчет по лабораторной работе #4 выполнил:
- Котенева Жанна Евгеньевна
- РИ-231002

| Задание | Выполнение | Баллы |
| ------ | ------ | ------ |
| Задание 1 | * | * |
| Задание 2 | * | * |
| Задание 3 | * | * |

Работу проверили:
- к.т.н., доцент Денисов Д.В.
- к.э.н., доцент Панов М.А.
- ст. преп., Фадеев В.О.

[![N|Solid](https://cldup.com/dTxpPi9lDf.thumb.png)](https://nodesource.com/products/nsolid)

[![Build Status](https://travis-ci.org/joemccann/dillinger.svg?branch=master)](https://travis-ci.org/joemccann/dillinger)

## Цель работы
Познакомиться с одной из первых моделей нейросетей - перцептроном и реализовать эту модель в Unity

## Задание 1. В проекте Unity реализовать перцептрон, который умеет производить вычисления
Ход работы:
- Возьмем скрипт, в котором описана логика обучения и вычисления перцептрона
```C#  
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

[System.Serializable]
public class TrainingSet
{
	public double[] input;
	public double output;
}

public class Perceptron : MonoBehaviour {

	public TrainingSet[] ts;
	double[] weights = {0,0};
	double bias = 0;
	double totalError = 0;

	double DotProductBias(double[] v1, double[] v2) 
	{
		if (v1 == null || v2 == null)
			return -1;
	 
		if (v1.Length != v2.Length)
			return -1;
	 
		double d = 0;
		for (int x = 0; x < v1.Length; x++)
		{
			d += v1[x] * v2[x];
		}

		d += bias;
	 
		return d;
	}

	double CalcOutput(int i)
	{
		double dp = DotProductBias(weights,ts[i].input);
		if(dp > 0) return(1);
		return (0);
	}

	void InitialiseWeights()
	{
		for(int i = 0; i < weights.Length; i++)
		{
			weights[i] = Random.Range(-1.0f,1.0f);
		}
		bias = Random.Range(-1.0f,1.0f);
	}

	void UpdateWeights(int j)
	{
		double error = ts[j].output - CalcOutput(j);
		totalError += Mathf.Abs((float)error);
		for(int i = 0; i < weights.Length; i++)
		{			
			weights[i] = weights[i] + error*ts[j].input[i]; 
		}
		bias += error;
	}

	double CalcOutput(double i1, double i2)
	{
		double[] inp = new double[] {i1, i2};
		double dp = DotProductBias(weights,inp);
		if(dp > 0) return(1);
		return (0);
	}

	void Train(int epochs)
	{
		InitialiseWeights();
		
		for(int e = 0; e < epochs; e++)
		{
			totalError = 0;
			for(int t = 0; t < ts.Length; t++)
			{
				UpdateWeights(t);
				Debug.Log("W1: " + (weights[0]) + " W2: " + (weights[1]) + " B: " + bias);
			}
			Debug.Log("TOTAL ERROR: " + totalError);
		}
	}

	void Start () {
		Train(8);
		Debug.Log("Test 0 0: " + CalcOutput(0,0));
		Debug.Log("Test 0 1: " + CalcOutput(0,1));
		Debug.Log("Test 1 0: " + CalcOutput(1,0));
		Debug.Log("Test 1 1: " + CalcOutput(1,1));		
	}
	
	void Update () {
		
	}
}
```
- Для удобства я решила добавить возможность указать количество эпох через инспектор, так выглядит обновленный скрипт (так же он содержит изменения и для 3 задания)
```C#
using System;
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

[System.Serializable]
public class TrainingSet
{
	public double[] input;
	public double output;
}

public class Perceptron : MonoBehaviour 
{
	public event Action EpochIsCompleted;
	[SerializeField] int epochs;
	public TrainingSet[] ts;
	double[] weights = {0,0};
	double bias = 0;
	public double totalError = 0;

	double DotProductBias(double[] v1, double[] v2) 
	{
		if (v1 == null || v2 == null)
			return -1;
	 
		if (v1.Length != v2.Length)
			return -1;
	 
		double d = 0;
		for (int x = 0; x < v1.Length; x++)
		{
			d += v1[x] * v2[x];
		}

		d += bias;
	 
		return d;
	}

	double CalcOutput(int i)
	{
		double dp = DotProductBias(weights,ts[i].input);
		if(dp > 0) return(1);
		return (0);
	}

	void InitialiseWeights()
	{
		for(int i = 0; i < weights.Length; i++)
		{
			weights[i] = UnityEngine.Random.Range(-1.0f,1.0f);
		}
		bias = UnityEngine.Random.Range(-1.0f,1.0f);
	}

	void UpdateWeights(int j)
	{
		double error = ts[j].output - CalcOutput(j);
		totalError += Mathf.Abs((float)error);
		for(int i = 0; i < weights.Length; i++)
		{			
			weights[i] = weights[i] + error*ts[j].input[i]; 
		}
		bias += error;
	}

	public double CalcOutput(double i1, double i2)
	{
		double[] inp = new double[] {i1, i2};
		double dp = DotProductBias(weights,inp);
		if(dp > 0) return(1);
		return (0);
	}

	void Train()
	{
		totalError = 0;
		for(int t = 0; t < ts.Length; t++)
		{
			UpdateWeights(t);
			Debug.Log("W1: " + (weights[0]) + " W2: " + (weights[1]) + " B: " + bias);
		}
		Debug.Log("TOTAL ERROR: " + totalError);
	}

	IEnumerator TrainCoroutine(int epochs) 
	{
		for (int i = 0; i < epochs; i++)
		{
            yield return new WaitForSeconds(1.0f);
            Train();
            EpochIsCompleted?.Invoke();
        }
		PrintResult();
	}
	void PrintResult()
	{
        Debug.Log("Test 0 0: " + CalcOutput(0, 0));
        Debug.Log("Test 0 1: " + CalcOutput(0, 1));
        Debug.Log("Test 1 0: " + CalcOutput(1, 0));
        Debug.Log("Test 1 1: " + CalcOutput(1, 1));
    }

	void Start () {
        InitialiseWeights();
		StartCoroutine(TrainCoroutine(epochs));
	}
}
```
- Запускаем код и видим результат (количество эпох: 5)

Как выглядит проект в незапущенном виде
![image](https://github.com/user-attachments/assets/f2543f28-2c3b-4341-b3e0-3ea8cd9463fd)
Как выглядит проект в запущенном виде(результат)
![image](https://github.com/user-attachments/assets/9c5bc0e8-62f1-4b8c-96b4-816a15e0952f)
![image](https://github.com/user-attachments/assets/3d5dc240-e47d-4e81-8b51-a0fefa2a8015)
- Как мы видим за 5 эпох, перцептрон смог обучиться вычислять выражение OR и выводит правильный результат


##Задание 2. Построение графиков зависимости количества эпох от ошибки обучения

Ход работы:
- Поскольку веса и пороговые значения устанавливаются случайным образом, графики будут различаться при каждом запуске. По этой причине я решил запустить перцептрон с разным количеством эпох лишь один раз.
- Затем я собрал все полученные значения в Google таблицу и на их основе создал график.  
image  
Ссылка на таблицу: https://docs.google.com/spreadsheets/d/1OfrKf16pv_pNA-ZwetEWpByjoqtfVh3hGdsvrTYTwD4/edit?gid=0#gid=0  
- Получившийся график не совсем очевиден, но даже из него видно, что увеличение числа эпох в общем приводит к более точным результатам работы перцептрона. Тем не менее, иногда можно наблюдать ситуацию, когда при небольшом количестве эпох обучение может проходить успешно.
Задание 3. Визуальная модель работы перцептрона в Unity

##Ход работы:
- Концепция визуализации: каждый раз, когда завершится эпоха, на платформу будет падать кубик, цвет которого будет отражать уровень обученности модели. Если кубик красного цвета, это будет означать, что модель не обучена, а если зеленого — что она обучена.
- Для реализации данной идеи я разработал скрипт, который получает результаты завершенной эпохи и в зависимости от этих данных создает куб.
```C#
using System.Collections;
using System.Collections.Generic;
using TMPro;
using UnityEngine;

public class CubeSpawner : MonoBehaviour
{
    [SerializeField] private Perceptron perceptron;
    [SerializeField] private GameObject _prefab;
    [SerializeField] private GameObject _platform;
    [SerializeField] private TextMeshProUGUI _text;
    private Vector3 lastPos;
    private int epochs = 0;

    private void OnEnable()
    {
        perceptron.EpochIsCompleted += CreateCube;
    }
    private void Start()
    {
        Vector3 platformPos = _platform.transform.position;
        var x = platformPos.x - (_platform.transform.localScale.x)/2.0f + 2;
        lastPos = new Vector3(x, platformPos.y + 10, platformPos.z);
    }
    private void CreateCube()
    {
        var obj = Instantiate(_prefab);
        obj.transform.position = CalculateThePosition(lastPos);
        lastPos = obj.transform.position;
        var color = obj.GetComponent<MeshRenderer>();
        if (perceptron.totalError > 0) 
            color.material.color = Color.red;
        else 
            color.material.color = Color.green;
        epochs++;
        _text.text = epochs.ToString();
    }

    private Vector3 CalculateThePosition(Vector3 lastPosition)
    {
        var newPos = new Vector3(lastPosition.x + _prefab.transform.localScale.x + 1, lastPosition.y, lastPosition.z);
        return newPos;
    }

}
```
- Так же я немного изменила скрипт перцептрона, для взаимодействия с скриптом выше (измененный скрипт перцептрона находиться в отчете по 1 заданию)
- Результат:

Сцена (запуск идет с 4 эпохами):
![image](https://github.com/user-attachments/assets/d3366925-e427-47df-81d6-513ebff5f681)

1 эпоха:
![image](https://github.com/user-attachments/assets/c333941a-beda-4732-9c92-a435fcf80c82)

2 эпоха:
![image](https://github.com/user-attachments/assets/bc165d55-0252-466e-837e-5abbd51e47bc)

3 эпоха:
![image](https://github.com/user-attachments/assets/0258d671-8b87-4ec9-aef2-ba0809e4d4f1)

4 эпоха:
![image](https://github.com/user-attachments/assets/9b01830e-3893-4942-b9b8-e36b048df7c5)

Финальное состояние:
![image](https://github.com/user-attachments/assets/a447d0d4-e18e-4f4a-a962-7ff07b7c3091)

- Как видно из скриншотов, визуализация успешно показывает состояние перцептрона на каждой стадии обучения(эпохе)
  
## Выводы
Я познакомилась с первой моделью нейросети - перцептроном, а также построил модель и визуализировал работу в Unity
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