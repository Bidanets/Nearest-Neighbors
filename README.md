# Nearest-Neighbors
Исходный код + Описание команд программы + Описание идеи алгоритма

sklearn.neighbors - это библиотека, которая предоставляет возможности работы с алгоритмами, основанными на соседях (как для случая обучения с учетелем, так и для случая обучения без учителя).

Метод ближайших соседей без учителя является основой для многих других алгоритмов машинного обучения, в частности, manifold learning и спектральной кластеризации.

Manifold learning - это подход, заключайщийся в нелинейном уменьшении размерности.

Обучение с учителем, основанное на соседях представляется в двух вариантах: классификация (для данных с дискретными метками) и регрессия (для данных с непрерывными метками).

Принцип метода ближайших соседей заключается в поиске предопределенного количества тренировочных (учебных) образцов, ближайших по расстоянию до новой точки, и предсказать метку по ним. Число образцов может быть определено пользователем константой (k-ближайших соседей), либо варьироваться в зависимости от локальной плотности точек (радиус-ориентированное обучение). Расстояние, вообще говоря, может быть любой метрической мерой: стандартное евклидово расстояние является наиболее распространенным выбором. Методы, основанные на соседях, известны как не обобщающие методы машинного обучения, так как они просто «запоминают» все свои учебные данные (возможно, превращаются в быструю структуру индексирования, такую как дерево шаров или KD дерево).

Несмотря на свою простоту, ближайшие соседи успешно справились с большим количеством проблем классификации и регрессии, включая рукописные цифры или сцены спутникового изображения. Будучи непараметрическим методом, он часто бывает успешным в ситуациях классификации, где граница решения очень нерегулярна.

Классы в sklearn.neighbors могут обрабатывать либо массивы Numpy, либо scipy.sparse матрицы в качестве входных данных. Для плотных матриц поддерживается большое количество возможных метрик расстояния. Для разреженных матриц для поиска поддерживаются произвольные метрики Минковского.

Существует множество учебных процедур, которые опираются на ближайших соседей по своей сути. Одним из примеров является оценка плотности ядра, обсуждаемая в разделе оценки плотности.

## 1.6.1. Ближайшие соседи (без учителя)

NearestNeighbors реализует метод обучения ближайших соседей без учителя. Он действует как единый интерфейс для трех разных алгоритмов ближайших соседей: BallTree, KDTree и алгоритм полного перебора, основанный на подпрограммах в sklearn.metrics.pairwise. Выбор алгоритма поиска соседей контролируется с помощью ключевого слова «algorithm», который должен быть одним из ['auto', 'ball_tree', 'kd_tree', 'brute']. Когда передается значение по умолчанию «auto», алгоритм пытается определить наилучший подход из обучающих (тренировочных) данных. Для обсуждения сильных и слабых сторон каждого варианта см. «Ближайшие соседние алгоритмы».

![Описание изображения](https://github.com/Bidanets/Nearest-Neighbors/blob/master/pictures/waights_uniform.png)
![Описание изображения](https://github.com/Bidanets/Nearest-Neighbors/blob/master/pictures/weights_distance.png)

## 1.6.4. Алгоитмы ближайших соседей

### 1.6.4.1. Полный перебор

Быстрое вычисление ближайших соседей - это активная область исследований машинного обучения. Наиболее наивная реализация поиска соседа включает в себя вычисление полного перебора расстояний между всеми парами точек в наборе данных: для N образцов в D измерениях вычислительная сложность такого подхода вычисляется как O(D * N^2). Поиск соседей при помощи полного перебора может быть конкурентноспособным для небольших выборок данных. Однако по мере роста количества образцов N полный перебор быстро становится неосуществимым. В классах внутри sklearn.neighbors поиск соседей при помощи полного перебора задается с использованием ключевого слова algorithm = 'brute' и вычисляется с использованием подпрограмм, доступных в sklearn.metrics.pairwise.

### 1.6.4.2. K-D Tree

Для устранения вычислительной неэффективности полного перебора были изобретены различные древовидные структуры данных. В целом, эти структуры пытаются уменьшить необходимое количество вычислений функции расстояния между объектами, эффективно кодируя совокупную информацию о расстоянии для образца. Основная идея заключается в том, что если точка A очень далека от точки B, а точка B очень близка к точке C, то мы знаем, что точки A и C очень далеки, без явного расчета их расстояния. Таким образом, вычислительная стоимость поиска ближайших соседей может быть сведена к O(D * N * log(N)) Это значительное улучшение по сравнению с полным перебором для больших N. Ранним подходом к использованию этой совокупной информации была структура данных KD дерева (сокращение для K-мерного дерева), которая обобщает двумерные деревья квадратнтов и трехмерные деревья октантов на произвольное число измерений. Дерево KD представляет собой двоичную древовидную структуру, которая рекурсивно разделяет пространство параметров вдоль осей данных, деля его на вложенные ортотопические области, в которых находятся точки данных. Конструкция KD дерева очень быстрая: поскольку разбиение выполняется только по осям данных, D-мерные расстояния не нужно вычислять. Однажды построенное, KD-дерево позволяет находить ближайшего соседа для точки-запроса за O[log(N)] вычислений расстояния. Хотя подход KD tree очень быстрый для поисков соседей с низким размером (D < 20), он становится неэффективным по мере того, как D растет очень велико: это одно из проявлений так называемого «проклятия размерности». В scikit-learn поиск соседей деревьев KD задается с использованием ключевого слова algorithm = 'kd_tree' и вычисляется с использованием класса KDTree.



### 1.6.4.3. Ball Tree

Для устранения неэффективности KD деревьев в более высоких измерениях была разработана структура данных шарового дерева. В тех случаях, когда деревья KD делят данные по декартовым осям, разбиение данных в серию гнездовых гиперсфер. Это делает конструкцию дерева более дорогостоящей, чем KD дерево, но приводит к структуре данных, которая может быть очень эффективной для высокоструктурированных данных даже при очень больших размерностях. Дерево шаров рекурсивно делит данные на узлы, определяемые центроидом C и радиусом r, так что каждая точка в узле лежит внутри гиперсферы, определяемой r и C. Число точек-кандидатов для поиска соседей уменьшается за счет использования неравенства треугольника:
|x+y| <= |x| + |y|.

В такой конструкции достаточно одного вычисления расстояния между контрольной точкой и центроидом, чтобы определить нижнюю и верхнюю границы расстояния до всех точек в узле. Из-за сферической геометрии узлов шарового дерева он может вывести KD-дерево в больших размерах, хотя фактическая производительность сильно зависит от структуры данных обучения. В scikit-learn поиски соседей на основе шариковых деревьев задаются с использованием ключевого слова algorithm = 'ball_tree' и вычисляются с использованием класса sklearn.neighbors.BallTree. Кроме того, пользователь может напрямую работать с классом BallTree.

### 1.6.4.4. Выбор алгоритма ближайших соседей

Оптимальный алгоритм для конкретного набора данных является сложным выбором и зависит от ряда факторов:

* количество образцов N (например, n_samples) и размерность D (например, n_features).

        - Время на выполнение запроса при помощи полного перебора растёт как O[DN].
        
        - Время на выполнение запроса при помощи дерева шаров растёт приближённо как O[Dlog(N)].
        
        - Время выполнения запроса KD деревом изменяется с D так, что его трудно точно охарактеризовать. Для малых D (менее 20 или около того) стоимость приблизительно равна O [D * log (N)], и запрос KD дерева может быть очень эффективным. Для большего D стоимость увеличивается почти до O[DN], а издержки из-за древовидной структуры могут приводить к времени выполнения запроса, которое медленнее, чем при полном переборе.
       
       Для небольших наборов данных (N менее 30 или около того), log(N) сравним с N, и полный перебор может быть более эффективным, чем методы, основанные на древовидной структуре. Оба метода KDTree и BallTree предоставляют параметр размера листа: он контролирует количество образцов, при которых запрос переключается на полный перебор. Это позволяет обеим алгоритмам приблизиться к эффективности вычисления полного перебора для малых N.
        
* Структура данных: внутренняя размерность данных и / или разреженность данных. Внутренняя размерность относится к размерности d <= D многообразия, на котором лежат данные, которые могут быть линейно или нелинейно вложены в пространство параметров. Разрешающая способность относится к степени, в которой данные заполняют пространство признаков (это следует отличать от понятия, используемого в «разреженных» матрицах. В матрице данных могут отсутствовать нулевые записи, но структура все еще может быть «разреженной» в этом смысле).
        
        - Вреия выполнения запроса при помощи полного перебора не изменяется за счет введения структуры данных.
        
        - Время обработи запроса при помощи дерева шаров и KD дерева может сильно зависеть от структуры данных. В общем, более редкие данные с меньшей внутренней размерностью приводят к более быстрому времени выполнения запросов. Поскольку внутреннее представление KD дерева выровнено по осями признаков, оно, как правило, не будет демонстрировать такого улучшения, как шаровое дерево для произвольно структурированных данных. Наборы данных, используемые в машинном обучении, как правило, очень структурированы и очень хорошо подходят для запросов, основанных на деревьях.
        
* количество соседей k, запрошенных для точки запроса.

        - Время выполнения запроса при помощи полного перебора в значительной степени не зависит от значения k.
        
        - Время шара и время запроса дерева KD будут замедляться с ростом k. Это связано с двумя эффектами: во-первых, больший k приводит к необходимости поиска в большей части пространства призаков. Во-вторых, использование k > 1 требует внутренней очередности результатов при прохождении дерева.
        
Поскольку k становится большим относительно N, способность обрезать ветви в древовидном запросе уменьшается. В этой ситуации запросы Brute force могут быть более эффективными.

* Количество точек запроса. И шаровое дерево, и KD дерево требуют фазы построения. Стоимость этого построения становится незначительной при амортизации по многим запросам. Однако, если будет выполнено лишь небольшое количество запросов, конструкция может составлять значительную часть общей стоимости. Если требуется очень мало точек запроса, полный перебор лучше, чем метод на основе дерева.

В настоящее время algorithm = 'auto' выбирает 'kd_tree' если k < N/2 и 'effective_metric_' находится в списке 'VALID_METRICS' списка 'kd_tree'. Алгоритм выбирает 'ball_tree', если k < N/2 и 'effective_metric_' находится в списке 'VALID_METRICS' списка 'ball_tree'. Алгоритм выбирает 'brute', если k < N/2 и 'effective_metric_' находится в списке 'VALID_METRICS' списка 'kd_tree' или 'ball_tree'. Алгоритм выбирает 'brute', если k >= N/2. Этот выбор основывается на предположении, что число точек запросов как минимум того же порядка, что и число тренировочных (обучающих) точек выборки, и что leaf_size близок к своему значению по умолчанию 30.

### 1.6.4.5. Эффект leaf_size

Как отмечалось выше, для небольших выборок полный перебор может быть более эффективным, чем запрос на основе дерева. Этот факт учитывается в дереве шаров и KD дереве, внутренне переключаясь на поиск при помощи полного перебора в листовых узлах. Уровень этого переключения можно задать параметром leaf_size. Этот параметр имеет множество эффектов:

время построения

Более крупный leaf_size приводит к более быстрому времени построения дерева, поскольку необходимо создать меньше узлов.

время выполнения запроса

Как большой, так и маленький leaf_size может привести к субоптимальной стоимости запроса. Для параметра leaf_size, приближающегося к 1, служебные данные, участвующие в продвижении по узлам, могут значительно замедлить время запроса. Для leaf_size, приближающегося к размеру тренировочного набора, запросы становятся по существу полным перебором. Хорошим компромиссом между ними является значение leaf_size = 30, значение по умолчанию для параметра.

память
По мере увеличения leaf_size память, необходимая для хранения древовидной структуры, уменьшается. Это особенно важно в случае шарового дерева, в котором хранится D-мерный центроид для каждого узла. Требуемое пространство для хранения для BallTree составляет приблизительно 1 / leaf_size, умноженное на размер обучающего набора.

## 1.6.5. Классификатор по ближайшему центроиду

Классификатор NearestCentroid - это простой алгоритм, который представляет каждый класс центроидом его членов. По сути, это делает его похожим на фазу обновления метки алгоритма sklearn.KMeans. Он также не имеет параметров для выбора, что делает его хорошим базовым классификатором. Тем не менее, он страдает от невыпуклых классов, а также когда классы имеют резко разные отклонения, поскольку предполагается равная дисперсия во всех измерениях. См. Линейный дискриминантный анализ (sklearn.discriminant_analysis.LinearDiscriminantAnalysis) и Квадратичный дискриминационный анализ (sklearn.discriminant_analysis.QuadraticDiscriminantAnalysis) для более сложных методов, которые не делают этого предположения. 

Использование по умолчанию NearestCentroid просто:

```python
    >>> from sklearn.neighbors.nearest_centroid import NearestCentroid
    >>> import numpy as np
    >>> X = np.array([[-1, -1], [-2, -1], [-3, -2], [1, 1], [2, 1], [3, 2]])
    >>> y = np.array([1, 1, 1, 2, 2, 2])
    >>> clf = NearestCentroid()
    >>> clf.fit(X, y)
    NearestCentroid(metric = 'euclidean', shrink_threshold = None)
    >>> print(clf.predict([[-0.8, -1]]))
```

![Описание изображения](https://github.com/Bidanets/Nearest-Neighbors/blob/master/pictures/sphx_glr_plot_nearest_centroid_1.png)
![Описание изображения](https://github.com/Bidanets/Nearest-Neighbors/blob/master/pictures/sphx_glr_plot_nearest_centroid_002.png)



### 1.6.5.1. Ближайший деформированных центроид

У классификатора NearestCentroid есть параметр shrink_threshold, который реализует классификатор ближайших деформированных центроидов. Фактически, значение каждой функции для каждого центроида делится на дисперсию внутри класса этой функции. Значения функций затем уменьшаются с помощью параметра shrink_threshold. Прежде всего, если конкретное значение функции пересекает ноль, оно устанавливается равным нулю. По сути, это устраняет эту функцию, влияя на классификацию. Это полезно, например, для удаления шумных функций.
В приведенном ниже примере использование небольшого порога усадки увеличивает точность модели от 0,81 до 0,82.

```python
print(__doc__)

import numpy as np
import matplotlib.pyplot as plt
from matplotlib.colors import ListedColormap
from sklearn import datasets
from sklearn.neighbors import NearestCentroid

n_neighbors = 15

# import some data to play with
iris = datasets.load_iris()
# we only take the first two features. We could avoid this ugly
# slicing by using a two-dim dataset
X = iris.data[:, :2]
y = iris.target

h = .02  # step size in the mesh

# Create color maps
cmap_light = ListedColormap(['#FFAAAA', '#AAFFAA', '#AAAAFF'])
cmap_bold = ListedColormap(['#FF0000', '#00FF00', '#0000FF'])

for shrinkage in [None, .2]:
    # we create an instance of Neighbours Classifier and fit the data.
    clf = NearestCentroid(shrink_threshold = shrinkage)
    clf.fit(X, y)
    y_pred = clf.predict(X)
    print(shrinkage, np.mean(y == y_pred))
    # Plot the decision boundary. For that, we will assign a color to each
    # point in the mesh [x_min, x_max]x[y_min, y_max].
    x_min, x_max = X[:, 0].min() - 1, X[:, 0].max() + 1
    y_min, y_max = X[:, 1].min() - 1, X[:, 1].max() + 1
    xx, yy = np.meshgrid(np.arange(x_min, x_max, h), np.arange(y_min, y_max, h))
    Z = clf.predict(np.c_[xx.ravel(), yy.ravel()])

    # Put the result into a color plot
    Z = Z.reshape(xx.shape)
    plt.figure()
    plt.pcolormesh(xx, yy, Z, cmap = cmap_light)

    # Plot also the training points
    plt.scatter(X[:, 0], X[:, 1], c=y, cmap=cmap_bold, edgecolor='b', s=20)
    plt.title("3-Class classification (shrink_threshold=%r)"
              % shrinkage)
    plt.axis('tight')

plt.show()
```


### Описание использумемых методов класса NearestNeighbors:

* fit(X, y = None)

Обучает модель используя X в качестве тренировочных данных

Параметры:	

X : {массив, разреженная матрица, дерево шаров, KD дерево}

Тренировочные данные. Если массив или матрица, форма [n_samples, n_features], или [n_samples, n_samples] если аргумент metric = 'precomputed'.

y - ответы для объектов выборки

### Описание использумемых методов класса NearestCentroid:

* Конструктор

* predict(X)[source]

Производит классификацию объектов тестовой выборки X и записывает ответы в массив.

The predicted class C for each sample in X is returned.

Аргументы: X : массиво-подобные типы, форма = [n_samples, n_features]

Возвращаемый результат:	C : массв, форма = [n_samples]

###### Обратите внимание

Если параметр metric, который был передан в конструктор равен “precomputed”, то предполагается, что X является матрицей расстояний между данными, подлежащими прогнозированию.

### Ссылки:

1. [Статья: KNN (классификация + регрессия) библиотеки scikitlearn с примерами кода] http://scikit-learn.org/stable/modules/neighbors.html#classification

2. [Документация по классу NearestCentroid] http://scikit-learn.org/stable/modules/generated/sklearn.neighbors.NearestCentroid.html#sklearn.neighbors.NearestCentroid

3. [Документация по классу NearestNeighbors] http://scikit-learn.org/stable/modules/generated/sklearn.neighbors.NearestNeighbors.html#sklearn.neighbors.NearestNeighbors

