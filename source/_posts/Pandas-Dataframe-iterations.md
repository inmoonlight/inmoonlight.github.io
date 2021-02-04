---
title: "Pandas Dataframe의 다양한 iteration 방법 비교"
layout: post
date: 2021-02-04 14:22
tags:
- pandas
categories: 
- ML
toc: true
widgets:
   - type: toc
     position: left
   - type: categories
     position: left
   - type: recent_posts
     position: left
---

`pandas`는 데이터를 다루는 사람들이라면 누구나 쓸 수 밖에 없는 오픈소스 라이브러리이다. table 형식의 데이터를 다루기에 편리하지만 오픈소스라는 특징과 다양한 기능 지원 때문에 속도 면에서는 최적화되어 있지 않은 편이다. 이번 글에서는 `pandas`의 여러 기능 중에서 `iteration`하는 여러 방법을 속도와 사용성 측면에서 비교해본 내용을 아주 간단하게 정리해 보았다.

<!--more-->

## TL;DR

rank | method | time | `iterrows` 대비 속도
-- | -- | -- | --
1 | **`itertuples`** | **7.7ms** | **x8.1**
2 | `at` / `iat` | 15.8ms | x4
3 | `loc` / `iloc` | 24.6ms | x2.5
4 | `iterrows` | 62.7ms | x1
||
번외 | `values` | 7.1ms | x8.8
번외 | `apply` + `to_dict` | 9.91 ms | x6.3

## Introduction
실험에 사용한 데이터는 아래와 같이 `id`, `text`, `title` 정보로 이루어진 위키피디아를 처리한 table 형식의 데이터이다. `text`는 위키피디아 문서를 일정 길이 단위로 잘라서 가공한 문장들이고, `title`은 해당 문장이 속한 위키피디아 문서의 제목을 의미한다. `id`는 각 문장들의 고유 번호이다. 

<img src="/assets/images/pandas-data-example.png?style=centerme" width=70%>
<br>

데이터의 row 별로 iteration을 하면서 처리할 내용은 1) 아래의 `cut_text`를 통해 `text`의 길이를 줄이고, 2) table 의 내용을 `list_of_dict` 형식으로 변환하는 것이다.
```python
def cut_text(text, max_len: int = 100):
    return ' '.join(text.split()[:max_len])
```

실험할 함수는 크게 `iterrows`, `loc`/`iloc`, `at`/`iat`, `itertuples`,  그리고 속도 면에서는 장점이 있으나 약간의 단점이 있는 `values`, 그리고 이번 task 에 overfitting 된 `apply` + `to_dict` 가 있다. 하나하나 살펴보도록 하자!


## `iterrows`
많이 사용되는 함수이지만 가장 성능이 좋지 않다.
```python
%%timeit

result = []
for i, row in data.iterrows():
    short_text = cut_text(row['text'])
    instance = {
        'id': row['id'],
        'text': short_text,
        'title': row['title']
    }
    result.append(instance)
```

**62.7 ms** ± 729 µs per loop (mean ± std. dev. of 7 runs, 10 loops each)

## `loc` / `iloc`

`iterrows` 다음으로 많이 사용되는 방식이다. `iterrows`에 비해 2.5배 정도 빠른 속도를 보인다.

```python
%%timeit

result = []
for idx in data.index:
    short_text = cut_text(data.loc[idx, 'text'])
    instance = {
        'id': data.loc[idx, 'id'],
        'text': short_text,
        'title': data.loc[idx, 'title']
    }
    result.append(instance)
```

**24.6 ms** ± 235 µs per loop (mean ± std. dev. of 7 runs, 10 loops each)


:warning: 다만, `loc`을 잘못 쓰게 되면 `iterrows`를 썼을 때보다도 더 오랜 시간이 소요된다.

```python
%%timeit

result = []
for idx in data.index:
    short_text = cut_text(data.loc[idx]['text'])  # diff
    instance = {
        'id': data.loc[idx]['id'],
        'text': short_text,
        'title': data.loc[idx]['title']
    }
    result.append(instance)
```

**261 ms** ± 1.12 ms per loop (mean ± std. dev. of 7 runs, 1 loop each)

:warning: 미리 row를 받으면 조금 더 빨라지지만, 그럼에도 `iterrows`대비 느리다.
```python
%%timeit

result = []
for idx in data.index:
    row = data.loc[idx]
    short_text = cut_text(row['text'])  # diff
    instance = {
        'id': row['id'],
        'text': short_text,
        'title': row['title']
    }
    result.append(instance)
```
**99.4 ms** ± 904 µs per loop (mean ± std. dev. of 7 runs, 10 loops each)


## `at` / `iat`

`loc` / `iloc` 과 유사하지만, 특정 column과 row에 해당하는 값을 받고 싶을 때 사용한다. `at` 함수에 대한 상세한 설명은 [pandas 공식 문서](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DataFrame.at.html)에서 확인할 수 있다. `iterrows`에 비해 4배 정도 빠른 속도를 보인다.

```python
%%timeit

result = []
for idx in data.index:
    short_text = cut_text(data.at[idx, 'text'])
    instance = {
        'id': data.at[idx, 'id'],
        'text': short_text,
        'title': data.at[idx, 'title']
    }
    result.append(instance)
```
**15.8 ms** ± 49.6 µs per loop (mean ± std. dev. of 7 runs, 100 loops each)



## `itertuples`

`iterrows`와 유사하지만, Series가 return되는 `iterrows`와는 다르게 NamedTuple이 return 된다. column에 대응되는 값에 접근하기도 쉽고, 속도도 8배 이상 빠르다.


```python
%%timeit

result = []
for row in data.itertuples():
    short_text = cut_text(row.text)
    instance = {
        'id': row.id,
        'text': short_text,
        'title': row.title
    }
    result.append(instance)
```
**7.7 ms** ± 21.9 µs per loop (mean ± std. dev. of 7 runs, 100 loops each)



## `values`

여기서부터는 번외 느낌인데, values는 속도가 가장 빠르다는 장점이 있지만 column에 대응되는 값을 불러올 때 불편한 점이 있다. 이 점을 감안해서 써도 무관하다면 가장 좋은 선택이 될 것 같다.
```python
%%timeit

result = []
for value in data.values:
    short_text = cut_text(value[1])
    instance = {
        'id': value[0],
        'text': short_text,
        'title': value[2]
    }
    result.append(instance)
```

**7.1 ms** ± 43.8 µs per loop (mean ± std. dev. of 7 runs, 100 loops each)



## `apply` + `to_dict`

`for` 문 안에서 처리할 내용이 복잡하지 않은 이번 태스크같은 경우에 쓰기 적합한 방식이다. 새로운 dataframe 혹은 새로운 column을 생성해야 해서 메모리 측면에서 오는 단점은 있지만, 코드가 짧고 깔끔하다는 장점이 있다. 

```python
%%timeit

result = data.copy()
result['text'] = result['text'].apply(lambda x: cut_text(x))
result = result.to_dict(orient='records')
```
**9.91 ms** ± 19.6 µs per loop (mean ± std. dev. of 7 runs, 100 loops each)



## References
- [https://inmoonlight.github.io/notebooks/html/2021-02-04-pandas-dataframe-iterations.html](https://inmoonlight.github.io/notebooks/html/2021-02-04-pandas-dataframe-iterations.html)
- [https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DataFrame.at.html](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DataFrame.at.html)
- [https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DataFrame.itertuples.html](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DataFrame.itertuples.html)
