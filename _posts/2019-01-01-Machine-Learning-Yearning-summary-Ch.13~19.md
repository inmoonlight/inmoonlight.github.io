---
title: "Machine Learning Yearning 요약: Ch.13~19"
layout: post
date: 2019-01-01 15:47
image: /assets/images/mlyearning.jpg
categories: [ 머신러닝, 스터디, 책, 요약 ]
featured: false
---

지난 글 >> <br> [Machine Learning Yearning 요약: Ch.1~4]({{ site.baseurl }}/2018/12/25/Machine-Learning-Yearning-summary-Ch.1~4.html) <br> 
[Machine Learning Yearning 요약: Ch.5~12]({{ site.baseurl }}/2018/12/26/Machine-Learning-Yearning-summary-Ch.5~12.html)

<div class="breaker"></div>

목적에 맞는 Dev와 Test set을 구축했다면, 이제 모델이 얼마나 잘하고 있는지, 못한다면 그 이유는 무엇인지에 대한 분석을 할 수 있다. 그래서 이번에 다룰 주제는 **Basic Error Analysis**이다. 

> Ch.13: Build your first system quickly, then iterate <br>
> Ch.14: Error analysis: Look at dev set examples to evaluate ideas <br>
> Ch.15: Evaluating multiple ideas in parallel during error analysis <br>
> Ch.16: Cleaning up mislabeled dev and test set examples <br>
> Ch.17: If you have a large dev set, split it into two subsets, only one of which you look at <br>
> Ch.18: How big should the Eyeball and Blackbox dev sets be? <br>
> Ch.19: Takeaways: Basic error analysis

<div class="breaker"></div>

이해의 용이성을 위해, 내가 속한 팀이 사용자들이 올린 이미지를 classification하는 모델을 서비스한다고 가정해보자. 팀원들은 모델의 classification accuracy를 향상시키고 싶을 것이고, 이를 위해 적절한 training, dev, test set을 구축하였다. 학습시킨 모델의 정확도는 90%이다. 하지만 이 정도로는 서비스에 반영하기 어렵다. 자, 이제 우리는 어떻게 해야할까?

일단, 10%의 에러가 어떤 데이터에 대해서 발생하는지 분석(=error analysis)해야 한다. 좋은 분석은 적절한 가정들을 바탕으로 이루어진다. 우리 팀이 생각한 모델의 문제는 “개(dog) 이미지를 잘못 분류한다”이다. 

지난 Chapter에서 dev set이 error analysis를 위한 data set임을 설명했다. 이 중 모델이 **misclassify한 데이터에서 100개를 sampling**한다. 그리고 우리의 가설-개 이미지를 잘못 분류-과 일치하는 case를 센다. 만약 5개였다면, 에러율 10%의 5%가 해당 케이스인 것이므로 모델이 앞으로 “개 이미지를 제대로 분류”한다고 하더라도 `10% * 0.05 = .5%` 정도의 향상을 기대할 수 있다. 

좀 더 효율을 높여보자. 우리는 지금 가설 1개에 대해서 100개의 sampling된 데이터를 살펴보았는데, 이 작업은 가설 여러개에 대해서 동시에 진행할 수 있다. 
> 가설1: 개 이미지를 잘못 분류 <br>
> 가설2: 고양이과(사자, 표범 등) 이미지를 잘못 분류 <br>
> 가설3: 흐린 이미지를 잘못 분류 

Andrew Ng은 헷갈리지 않도록 아래 그림처럼 spreadsheet을 만들어서 작업을 진행한다고 한다.

![figure1]({{ site.baseurl }}/assets/images/mlyearning-ch13-19-figure1.png)

`% of total`을 통헤 현재 모델이 가장 critical하게 개선해야 하는 점이 무엇인지를 바로 파악할 수 있다.

모델의 성능이 좋지 않을 때 항상 염두에 두어야 하는 가설이 있다. 바로, “mislabeled data의 비율이 얼마나 되는가?”이다. 데이터의 양이 많을수록 수집과정에서 예상치 못한 에러가 발생한다. mislabeled data가 발견된다면 training data의 mislabel은 어쩔 수 없으니 내버려두고, dev set과 test set은 수정하는 것을 권장한다. (dev와 test set의 distribution은 같아야 하므로)

![figure2]({{ site.baseurl }}/assets/images/mlyearning-ch13-19-figure2.png)
 

<div class="breaker"></div>

지금까지 소개한 error analysis 방법은 misclassify된 data 중에서 100개에 적용되었다. 이 말은 모델의 성능이 95%의 accuracy라고 가정했을 때, dev set의 크기가 `100/0.05 = 2,000` 정도라는 뜻을 함유하고 있다. 만약 나의 dev set이 이 정도의 규모보다 작다면, 혹은 크다면 어떻게 해야할까?

Andrew Ng이 제안한 100개의 error data 개수는 이 정도가 모델의 major error source로서 매우 좋은 인사이트를 발견할 수 있다고 생각했기 때문이다.

> ~100개: 매우 좋은 인사이트 발견 <br>
> ~50개: 좋은 인사이트 발견 <br>
> ~20개: 개략적인 인사이트 발견 <br>
> ~10개: 불충분. 하지만 적은 dev set을 가지고 있다면 없는 것보다는 나은 정도

그래서 dev set이 작다면 모든 데이터를 error analysis를 위해 사용하는 것이 좋다. 만약 dev set이 크다면, **eyeball dev set**과 **blackbox dev set**으로 나누어 사용한다. eyeball dev set은 error analysis를 하면서 계속 모델이 잘하거나 못하는 것을 살펴볼 용도로 사용하는 dev set이고, blackbox dev set은 모델의 파라미터를 튜닝하기 위한 용도로만 사용하는 dev set이다. 만약 `eyeball error rate << blackbox error rate` 이라면 모델이 eyeball dev set에 overfitting하고 있으므로 새로운 eyeball dev set을 구축하는 것으로 그 위험을 방지한다. 

여기에도 예외가 있는데, 사람조차 풀기 어려운 과제를 위한 모델을 만들고 있다면 eyeball dev set은 전혀 도움이 되지 않을 것이므로 eyeball dev set을 만들지 않아도 좋다. 

<div class="breaker"></div>

그리고 여기까지 소개된 일련의 과정-데이터 수집 -> 모델 구성 -> 학습 -> 결과 분석 pipeline-이 최대한 빨리 iterate하는 것이 중요하다. 처음부터 완벽한 것은 없다. 일단 시작하자!