+++
date = "2016-08-09T20:08:11+02:00"
draft = true
title = "Análisis de datos - Kruskal Wallis <Python>"
description = "Cuando no se cumplen las condiciones para ANOVA podemos utilizar un Test de Kruskal Wallis"
slug = "Kruskal-Wallis"
tags = ['Estadistica', 'python', 'Kruskal-Wallis']
+++

El test de [Kruskal-Wallis](https://es.wikipedia.org/wiki/Prueba_de_Kruskal-Wallis) es una herramienta [no paramétrica](https://es.wikipedia.org/wiki/Estad%C3%ADstica_no_param%C3%A9trica) que puede utilizarse como alternativa a ANOVA para probar si un grupo de datos proviene de la misma población.

Para realizar este análisis vamos a analizar un conjunto de datos sobre alpha acidos en 10 variedades distintas de cerveza.

``` python
import pandas as pd
from scipy import stats
import seaborn as sns
import matplotlib.pyplot as plt

%matplotlib inline

dataset = pd.read_csv('http://www.stat.ufl.edu/~winner/data/hop_alphaacid.dat',
                      delim_whitespace=True, header=None,usecols=[0,2])
dataset.columns=['beer','acid']

for beer,acids in dataset.groupby('beer'):
    print(stats.shapiro(acids.values))

"""
(0.6658862233161926, 1.5391195120173506e-05)
(0.6899054646492004, 2.94409146590624e-05)
(0.6546444296836853, 1.1463617738627363e-05)
(0.6694637537002563, 1.6923777366173454e-05)
(0.6543060541152954, 1.1363345038262196e-05)
(0.6866198778152466, 2.6898378564510494e-05)
(0.7335858345031738, 0.00010311066580470651)
(0.656919538974762, 1.216252894664649e-05)
(0.6805392503738403, 2.2788872229284607e-05)
(0.6560644507408142, 1.1894662748090923e-05)
"""
```

Como vemos, según el test de Shapiro no podemos contar con tener distribuciones normales.

```python
beer_1 = dataset[dataset['beer']==1]
beer_2 = dataset[dataset['beer']==2]
beer_3 = dataset[dataset['beer']==3]
beer_4 = dataset[dataset['beer']==4]
beer_5 = dataset[dataset['beer']==5]
beer_6 = dataset[dataset['beer']==6]
beer_7 = dataset[dataset['beer']==7]
beer_8 = dataset[dataset['beer']==8]
beer_9 = dataset[dataset['beer']==9]
beer_10 = dataset[dataset['beer']==10]

stats.kruskal(beer_1,beer_2,beer_3,beer_4,beer_5,beer_6,beer_7,beer_8,beer_9,beer_10)
# KruskalResult(statistic=778.76284061339015, pvalue=7.9136719119986609e-162)
```

Al ejecutar el test de Kruskal-Wallis nuestro p-value (95% / p-value < 0.05) nos hace rechazar la hipótesis nula y aceptar la hipótesis alternativa: Provienen de distribuciones diferentes.

Podríamos calcular las distintas combinaciones con el siguiente código.

```python
for i in range(1,11):
    for j in range(1,11):
        if (i!=j):
            result = stats.kruskal(dataset[dataset['beer']==i],dataset[dataset['beer']==j])
            print("Kruskal test result for beer_%s / beer_%s is %s" % (i, j,result))

"""
beer_1/beer_2 - KruskalResult(statistic=182.82099774559833, pvalue=1.1735670256700217e-41)
beer_1/beer_3 - KruskalResult(statistic=173.39513618946174, pvalue=1.3417656226854723e-39)
beer_1/beer_4 - KruskalResult(statistic=182.82099774559833, pvalue=1.1735670256700217e-41)
beer_1/beer_5 - KruskalResult(statistic=76.704856921459026, pvalue=1.9851455196975826e-18)
beer_1/beer_6 - KruskalResult(statistic=182.82099774559833, pvalue=1.1735670256700217e-41)
beer_1/beer_7 - KruskalResult(statistic=182.82099774559833, pvalue=1.1735670256700217e-41)
beer_1/beer_8 - KruskalResult(statistic=182.85640415554229, pvalue=1.1528635062101324e-41)
beer_1/beer_9 - KruskalResult(statistic=182.83869923647222, pvalue=1.1631702031256368e-41)
beer_1/beer_10 - KruskalResult(statistic=182.83869923647222, pvalue=1.1631702031256368e-41)
beer_2/beer_1 - KruskalResult(statistic=183.46971218345016, pvalue=8.4699407927293893e-42)
beer_2/beer_3 - KruskalResult(statistic=77.323926634945209, pvalue=1.4509758666303886e-18)
beer_2/beer_4 - KruskalResult(statistic=172.27745263449034, pvalue=2.353786118628795e-39)
beer_2/beer_5 - KruskalResult(statistic=77.346393635420256, pvalue=1.4345642124340119e-18)
beer_2/beer_6 - KruskalResult(statistic=86.160102337159444, pvalue=1.6594203595234067e-20)
beer_2/beer_7 - KruskalResult(statistic=178.9113898492601, pvalue=8.3774525767867606e-41)
beer_2/beer_8 - KruskalResult(statistic=150.29274301760984, pvalue=1.4961442844599786e-34)
beer_2/beer_9 - KruskalResult(statistic=123.90593198069237, pvalue=8.8333566703737788e-29)
beer_2/beer_10 - KruskalResult(statistic=119.60829113748925, pvalue=7.7070995196534613e-28)
beer_3/beer_1 - KruskalResult(statistic=193.69793191218577, pvalue=4.9566948331883891e-44)
beer_3/beer_2 - KruskalResult(statistic=319.41220814429568, pvalue=1.9450707121267461e-71)
beer_3/beer_4 - KruskalResult(statistic=183.75878333517758, pvalue=7.3243844870505036e-42)
beer_3/beer_5 - KruskalResult(statistic=77.632817148946174, pvalue=1.2409113917653372e-18)
beer_3/beer_6 - KruskalResult(statistic=183.75878333517758, pvalue=7.3243844870505036e-42)
beer_3/beer_7 - KruskalResult(statistic=183.75878333517758, pvalue=7.3243844870505036e-42)
beer_3/beer_8 - KruskalResult(statistic=183.79437480977282, pvalue=7.1945052698543894e-42)
beer_3/beer_9 - KruskalResult(statistic=183.77657734925091, pvalue=7.2591606845041683e-42)
beer_3/beer_10 - KruskalResult(statistic=183.77657734925091, pvalue=7.2591606845041683e-42)
beer_4/beer_1 - KruskalResult(statistic=183.35176410384076, pvalue=8.9873288947068507e-42)
beer_4/beer_2 - KruskalResult(statistic=196.1124519430231, pvalue=1.4730675010800318e-44)
beer_4/beer_3 - KruskalResult(statistic=183.36951698572531, pvalue=8.9074797775597577e-42)
beer_4/beer_5 - KruskalResult(statistic=77.228399861639573, pvalue=1.5228792172501558e-18)
beer_4/beer_6 - KruskalResult(statistic=77.191019222790757, pvalue=1.5519764650265139e-18)
beer_4/beer_7 - KruskalResult(statistic=158.91522887567419, pvalue=1.9528219723400345e-36)
beer_4/beer_8 - KruskalResult(statistic=77.205967135111209, pvalue=1.5402748061398791e-18)
beer_4/beer_9 - KruskalResult(statistic=77.19849245536146, pvalue=1.5461151282196017e-18)
beer_4/beer_10 - KruskalResult(statistic=77.19849245536146, pvalue=1.5461151282196017e-18)
beer_5/beer_1 - KruskalResult(statistic=318.65092433718934, pvalue=2.8494499854064845e-71)
beer_5/beer_2 - KruskalResult(statistic=318.62006226219296, pvalue=2.8939002355603347e-71)
beer_5/beer_3 - KruskalResult(statistic=318.6817923914698, pvalue=2.8056740890780262e-71)
beer_5/beer_4 - KruskalResult(statistic=318.62006226219296, pvalue=2.8939002355603347e-71)
beer_5/beer_6 - KruskalResult(statistic=182.92722241438946, pvalue=1.1125427685677647e-41)
beer_5/beer_7 - KruskalResult(statistic=182.92722241438946, pvalue=1.1125427685677647e-41)
beer_5/beer_8 - KruskalResult(statistic=182.96266312395343, pvalue=1.0928970261138296e-41)
beer_5/beer_9 - KruskalResult(statistic=182.94494105274811, pvalue=1.102677095022053e-41)
beer_5/beer_10 - KruskalResult(statistic=182.94494105274811, pvalue=1.102677095022053e-41)
beer_6/beer_1 - KruskalResult(statistic=182.89176659336405, pvalue=1.1325502840629965e-41)
beer_6/beer_2 - KruskalResult(statistic=305.39256257779004, pvalue=2.202661871660915e-68)
beer_6/beer_3 - KruskalResult(statistic=182.9094749363727, pvalue=1.1225129895223512e-41)
beer_6/beer_4 - KruskalResult(statistic=318.50122251417508, pvalue=3.0716362552529769e-71)
beer_6/beer_5 - KruskalResult(statistic=182.96262054652371, pvalue=1.0929204185484175e-41)
beer_6/beer_7 - KruskalResult(statistic=182.87406167888261, pvalue=1.1426753659523052e-41)
beer_6/beer_8 - KruskalResult(statistic=182.9094749363727, pvalue=1.1225129895223512e-41)
beer_6/beer_9 - KruskalResult(statistic=171.83823863007632, pvalue=2.9355478056599267e-39)
beer_6/beer_10 - KruskalResult(statistic=171.35313632716307, pvalue=3.7465773760223305e-39)
beer_7/beer_1 - KruskalResult(statistic=183.35176410384076, pvalue=8.9873288947068507e-42)
beer_7/beer_2 - KruskalResult(statistic=188.44067210620938, pvalue=6.9615482496704649e-43)
beer_7/beer_3 - KruskalResult(statistic=183.36951698572531, pvalue=8.9074797775597577e-42)
beer_7/beer_4 - KruskalResult(statistic=211.19655096113951, pvalue=7.5303602019768899e-48)
beer_7/beer_5 - KruskalResult(statistic=183.42279626426838, pvalue=8.6720741412281443e-42)
beer_7/beer_6 - KruskalResult(statistic=183.3340146591066, pvalue=9.0678781751384901e-42)
beer_7/beer_8 - KruskalResult(statistic=78.179133008741843, pvalue=9.4107412852460189e-19)
beer_7/beer_9 - KruskalResult(statistic=77.19849245536146, pvalue=1.5461151282196017e-18)
beer_7/beer_10 - KruskalResult(statistic=77.19849245536146, pvalue=1.5461151282196017e-18)
beer_8/beer_1 - KruskalResult(statistic=182.87852370346798, pvalue=1.1401151239515657e-41)
beer_8/beer_2 - KruskalResult(statistic=222.43786053203112, pvalue=2.6585023617581759e-50)
beer_8/beer_3 - KruskalResult(statistic=182.8962341938518, pvalue=1.1300095618467693e-41)
beer_8/beer_4 - KruskalResult(statistic=318.51424145180914, pvalue=3.0516444886766595e-71)
beer_8/beer_5 - KruskalResult(statistic=182.94938625261202, pvalue=1.1002157823871668e-41)
beer_8/beer_6 - KruskalResult(statistic=182.86081664269111, pvalue=1.1503090811367809e-41)
beer_8/beer_7 - KruskalResult(statistic=317.00141086643799, pvalue=6.5173095416915248e-71)
beer_8/beer_9 - KruskalResult(statistic=93.186144544976585, pvalue=4.7600178124704071e-22)
beer_8/beer_10 - KruskalResult(statistic=76.70469366016961, pvalue=1.9853096360155938e-18)
beer_9/beer_1 - KruskalResult(statistic=183.75878333517758, pvalue=7.3243844870505036e-42)
beer_9/beer_2 - KruskalResult(statistic=257.16956972739723, pvalue=7.1038717452054866e-58)
beer_9/beer_3 - KruskalResult(statistic=183.77657734925091, pvalue=7.2591606845041683e-42)
beer_9/beer_4 - KruskalResult(statistic=319.38128431738653, pvalue=1.9754739751130509e-71)
beer_9/beer_5 - KruskalResult(statistic=183.82998007416836, pvalue=7.0668802701188614e-42)
beer_9/beer_6 - KruskalResult(statistic=196.21162028327984, pvalue=1.4014571744715431e-44)
beer_9/beer_7 - KruskalResult(statistic=319.38128431738653, pvalue=1.9754739751130509e-71)
beer_9/beer_8 - KruskalResult(statistic=295.34299687335709, pvalue=3.4072310637003454e-66)
beer_9/beer_10 - KruskalResult(statistic=120.60916509903728, pvalue=4.6534321262908033e-28)
beer_10/beer_1 - KruskalResult(statistic=182.87408708642246, pvalue=1.1426607712272738e-41)
beer_10/beer_2 - KruskalResult(statistic=260.1813079328187, pvalue=1.5667344529260427e-58)
beer_10/beer_3 - KruskalResult(statistic=182.8917954322234, pvalue=1.1325338651118414e-41)
beer_10/beer_4 - KruskalResult(statistic=318.49667372031587, pvalue=3.0786521745575927e-71)
beer_10/beer_5 - KruskalResult(statistic=182.94494105274811, pvalue=1.102677095022053e-41)
beer_10/beer_6 - KruskalResult(statistic=195.15017570072769, pvalue=2.3891077662400293e-44)
beer_10/beer_7 - KruskalResult(statistic=318.49667372031587, pvalue=3.0786521745575927e-71)
beer_10/beer_8 - KruskalResult(statistic=318.55835604310471, pvalue=2.9848649669054756e-71)
beer_10/beer_9 - KruskalResult(statistic=262.17236084983955, pvalue=5.7676750019395979e-59)
"""
```

Aunque vemos que por pares los resultados son similares.
