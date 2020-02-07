---
layout: single
title: "[Python] - Jupyter에서 Interacive Plot 그리기"
categories: [data]
tags: [data, plot, python]
---

# Jupyter Notebook으로 Interaction Plot 그리기

최근 프로젝트는 데이터 처리 backend는 R로 되어있는 EDA(Explore Data Analysis) 툴을 개발하는 것이었다.

이 프로젝트에서는 Front는 React, Back은 Spring 그리고 workflow 툴인 airflow 등 여러개를 사용하였다.



그런데 아무래도 주변 분석가들이 EDA 할 때 노가다하고.. 할 때마다 설정 변경해서 다시 돌리고 하는 작업을 보니까 그냥 개인한테 쉽고 예쁘게 볼 수 있는 뭔가가 없을까 했다.

그러다가 예전에 봤던 jupyter에서 interactive한 widget을 만들고 plot을 바로 변형해서 보여주는 Jupyter Widgets(ipywidgets)이 떠올랐다.



사용하게 될 라이브러리는

- pandas : 데이터 프레임 처리
- plotly : plot 전용 라이브러리
- ipywidgets : interactive widgets 지원
- cufflinks : plotly와 pandas 를 유연하게 연결하여 쉬운 ploting을 제공

가 되겠다.



## 1. Installation

OSX 기준으로 설명한다.



```
pip install ipywidgets chart_studio cufflinks
```





## 2. 함수 만들기

간단한 예제로 피벗 테이블을 만들어서 피벗 테이블 Bar plot까지 그려보는 예제로 진행하겠다.



```python
#Import Library
import pandas as pd
import ipywidgets as widgets
from ipywidgets import interact, interact_manual, fixed
import chart_studio.plotly as py 
import cufflinks as cf 

cf.go_offline(connected=True)
```



간단하게 pivot테이블을 wrapping한 함수를 만들어 보겠다.

pivot 함수의 parameter는  다음과 같이 들어 올 것이다.

```python
pivotDict = {
    "index" : ["Age", "Drug"],
    "columns" : "Sex",#None,
    "values" : ["SBP_1", "DBP_1"],
    "aggfunc" : "mean",
    "dropna" : True,
    "margins" : True
}
pivotDict
```



추가적으로 theme과 colorscale은 plotly, cufflinks를 이용하여 plot을 그릴 때 색상과 테마를 지정하기 위해 추가하였다.

참고로 pivot 테이블 만들 때 컬럼이 굉장히 많이 생기는 컬럼을 선택하면 상당히 느리기에 1000건으로 잡았다.



저기서 중요한 부분이 

`df.iplot(kind='bar')` 이다. pythond에서 plot library는 maplotlib, seaborn 등 많은데 이번 프로젝트에서 R Plotly를 많이 사용했고 interactive를 제공해 줘서 좋았다.



df.iplot 역시 plotly와 cufflinks를 이용해 pandas의 데이터프레임의 plot을 쉽게 그려준다.

pivot 테이블을 만들면

<img src="/Users/kim-youngjae/Library/Application Support/typora-user-images/image-20200206220509817.png" alt="image-20200206220509817" style="zoom:50%;" />

이런 모양이다.

Sex별(column) Drug별(row) 나이(Age)의 평균 값을 보고 싶을 때 만든 테이블이다.

저 pivot table을 iplot으로 그리면 알아서 grouped bar plot으로 그려준다.



<img src="/Users/kim-youngjae/Library/Application Support/typora-user-images/image-20200206220734432.png" alt="image-20200206220734432" style="zoom:50%;" />

실은 최종 결과는 위와 같다.

왜냐면 나중에 저렇게 일반적인 dataframe 형태로 만들어야 DB던 parquet던 arrow던 유연하게 데이터 포맷을 변경할 수 있다.



```python
def interactPivot(df, index, columns, values, aggfunc, dropna, margins, theme, colorscale):
    funcMap = {"sum" : np.sum, "mean" : np.mean, "min" : np.min, "max" : np.max, 
               "median" : np.median, "std" : np.std, "var" : np.var}
    
    # pivot table에서 index, value, columns의 선택한 항목이 같으면 에러가 발생하여 같은 경우 예외처리.
    if index in values or index == columns or columns in values:
        return("Index is same with values")
    if len(df[columns].unique())>1000:
        return("Too many columns")
    
    df = df.pivot_table(
        values=values, columns=columns, index=index, dropna=dropna, margins=margins, aggfunc=funcMap[aggfunc])
    
    df.iplot(kind='bar')
    
    if columns != None:    
        if len(values) >1:
                df.columns = list(map(lambda x: '__'.join(x), df.columns))
        else:
            df.columns = list(map(lambda x: x[1], df.columns))
    
    df.reset_index(col_level=1, inplace=True)
    return(df)

```



## 3. Do interact!

함수는 만들었으니까 함수를 이제 보여준다.



- Dropdown : list 중 한개 선택
- SelectMultiple : list 중 여러개 선택
- Options : 보여줄 목록
- value : 초기 값



interact함수에 parameter로 위에서 만든 함수를 넘겨준다.

그리고 dataframe은 ipython fixed 객체로 변환해서 넘겨줘야한다.

df.select_dtypes로 선택 가능한 컬럼만 출력하도록 하였다.

theme과 colorscale은 cufflinks에서 제공한다. 많다.



```python
print(cf.themes.THEMES.keys())
print(cf.colors._scales_names.keys())

dict_keys(['ggplot', 'pearl', 'solar', 'space', 'white', 'polar', 'henanigans'])
dict_keys(['brbg', 'prgn', 'piyg', 'puor', 'rdbu', 'rdgy', 'rdylbu', 'rdylgn', 'spectral', 'paired', 'set3', 'dflt', 'original', 'plotly', 'accent', 'dark2', 'pastel1', 'pastel2', 'set1', 'set2', 'blues', 'bugn', 'bupu', 'gnbu', 'greens', 'greys', 'orrd', 'oranges', 'pubu', 'pubugn', 'purd', 'purples', 'rdpu', 'reds', 'ylgn', 'ylgnbu', 'ylorbr', 'ylorrd', 'ggplot', 'polar'])
```



```python
indexWidget = widgets.Dropdown(options=df.columns, 
                               value=df.columns[0])

valuesWidget = widgets.SelectMultiple(
        options=df.select_dtypes("number").columns,
        value=[df.select_dtypes("number").columns[0]],
        #rows=10,
        description='values',
        disabled=False
    )

interact(
    interactPivot, 
    df=fixed(df), 
    index=indexWidget, 
    columns=df.select_dtypes("object").columns, 
    values=valuesWidget,
    #df.select_dtypes("number").columns, 
    aggfunc=["mean", "sum", "min", "max", "median", "std", "var"],
    dropna = [True,False],
    margins = [True,False],
    theme=list(cf.themes.THEMES.keys()), 
    colorscale=list(cf.colors._scales_names.keys())
)
```



음 원래는 index랑 values를 선택할 때 상호 선택한 것은 배제하려고 했다.

그래서 index를 선택할 때 values의 options에서 제외되고 values를 선택하면 index의 option에서 제외되도록 했다.

event를 추가하기 위해 각 widget에 observe를 걸어 놨는데 아직 동작 원리는 제대로 안보고 구현 했기에 일단은 빼 버렸다.

그래서 예외처리가 일단 들어갔죠..



실행을 시키면

![](/Users/kim-youngjae/Kim/4.study/0.Blog/dudwo7783.github.io/assets/images/ipywidgets.jpg)



위와 같이 나온다.

<img src="/Users/kim-youngjae/Kim/4.study/0.Blog/dudwo7783.github.io/assets/images/ipyindex.jpg" alt="ipyindex" style="zoom:50%;" />

dropdown 위젯은 위처럼 한개만 선택가능하며

![ipyvalues](/Users/kim-youngjae/Kim/4.study/0.Blog/dudwo7783.github.io/assets/images/ipyvalues.png)



위의 values는 selectMultiple이기에 여러개를 선택할 수 있다.

여러개를 선택하면 바로 위 grouped bar plot이 그려진다.











## 마치며

