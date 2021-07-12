---
title: "With Little Power Comes Great Responsibility"
layout: post
date: 2020-12-13 18:33
tags:
- ML
- NLP
- evaluation
- paper
categories: 
- Paper
toc: true
widgets:
   - type: toc
     position: left
   - type: categories
     position: left
   - type: recent_posts
     position: left
---

요즘 등장하는 NLP model 페이퍼들은 주로 GLUE 벤치마크에 성능을 report 하면서 아주 미세한 성능 개선을 근거로 "우리 방법론은 효과적이었다!"를 주장하고 있다. 과연 이 결과가 실제로 그 모델이 더 나은 모델임을 주장할 수 있을만큼 근거가 탄탄할까?

이번에 소개하는 논문에서는 NLP research에서 모델의 성능 개선을 주장하는 실험 결과에 대해 그 결과가 "정말 유의미한 모델의 성능 개선을 보장할 수 있는가?"에 대해 분석한다. 더불어, 분석 결과를 통해 발견된 문제점을 개선할 수 있는 간단한 overview 까지 제안하고 있다.
<!--more-->

## TMI

참고로 이 논문의 제목은 Spiderman의 유명한 대사에서 유래했다. 제목에서부터 느껴지는 현 시대의 NLP model evaluation에 대한 강한 부정적인 뉘양스는 논문의 교신저자가 Dan Jurafsky 이기에 가능하지 않았을까 싶다.

<img src="/assets/images/with-great-power-spiderman.png?style=centerme" width=55% alt="source: Spiderman(2002)">

## Introduction

본격적인 분석 방법론 소개에 앞서 앞으로 등장하게 될 용어에 대해 간단하게 짚고 넘어가려고 한다.

> **Power**
> 
> Power 란, 샘플 데이터에서 관측한 결과가 true distribution 데이터에 대해서도 적용될 수 있는지에 대한 확률로, "통계적 유의미함"과는 관련은 있으나, 다른 metric이다. 뒤에서 언급하겠지만, Power는 r 번의 반복수행 동안 통계적으로 유의미한 결과가 몇번 등장했는가를 바탕으로 측정된다. 

> **Type-S error**
>
> sign 에 대한 에러. 쉬운 예를 들면, 실제로 모델 A가 모델 B에 비해 성능이 좋은데 실험 결과는 반대로 나오는 경우에 해당한다.

> **Type-M error**
> 
> Magnitude에 대한 에러. 예를 들어, 실제 모델의 예측값의 차이가 적은데 관측된 결과에 따르면 예측값의 차이가 큰 경우에 해당한다.

> **MDE (minimum detectable effect) size**
>
> 유의미한 성능 차이(effect)를 보장할 수 있는 최소 데이터 사이즈. 이 논문에서 유의미한 effect는 80%  Power를 의미한다. 만약 테스트셋의 사이즈가 작으면 MDE는 커진다.


이 용어들을 활용해서 다시 본 논문의 contribution을 정리하면 다음과 같다. 

NLP 커뮤니티에서의 실험 결과에 대한 Power를 고려하지 않는 실험 세팅과 결과 report 때문에 statistical noise와 유의미한 모델의 성능 향상을 구분하기 어렵다. Underpowered result는 실험 결과를 과장하거나, 실제 효과를 반대로 해석할 수 있는 여지가 있다. 너무 적은 샘플에 대해서는 유의미한 차이를 판단하기 어렵고, 유의미한 차이를 보이는 경우이더라도 샘플 수가 적을수록 그 효과를 과장해서 평가할 수 있다. 

따라서 모델을 평가할 때에 주어진 조건들 - 테스트셋의 크기, 평가자 인원수 등 - 에 대해 결과가 유의미함을 보장하는지 생각해 볼 필요가 있다. 이를 위해 기존의 평가 framework에 대한 Power Analysis 가 필요하고, 동시에 유의미한 성능 개선을 보여줄 수 있는 조건에 대해서도 논의될 필요가 있다. 

먼저, NLP 에서의 Power Analysis에 대해 알아보자.


## Power Analysis for NLP


NLP의 task의 평가 format은 그동안 다른 과학적 방법론에서 적용되는 Power Analysis를 수행하기에 적합하지 않다. NLP 평가의 모든 시나리오를 커버할 수 없기 때문에 저자들은 최대한 일반화 가능한 simluation 기반의 power analysis를 제안한다. 

저자들이 제안하는 Power Analysis 알고리즘은 다음과 같다.

<img src="/assets/images/with-little-power-figure-2.png?style=centerme" width=50%>

Simulation을 하기 위해서는 특정 parameter들이 요구된다. 저자들은 총 6개의 parameter를 제안하고 각각은 다음과 같다.

- \\(n\\): 평가에 사용되는 데이터의 개수
- \\(e*\\): 관심있는 평가 수치 (e.g., \\(\\Delta_{acc}\\) 등)
- \\(h\\): 관련있는 다른 수치 (e.g., variance 등)
- \\(T\\): 유의미성을 판단할 수 있는 statistical test
- \\(\\alpha\\): 유의미성을 판단할 수 있는 threshold
- \\(r\\): 반복 수행 횟수

알고리즘을 간단하게 설명하면, 총 r번의 실험에 대해 n 개의 데이터에 대한 평가 결과를 생성하고 이 결과를 바탕으로 significance test를 수행해서 p-value 를 얻는다. 최종적으로 유의미한 결과 (p-value가 특정 threshold 이하인 경우)의 횟수가 r번 중 몇번인지를 바탕으로 power 를 계산한다. 아래는 위 알고리즘을 바탕으로 power analysis를 수행한 예시이다.

<img src="/assets/images/with-little-power-figure-1.png?style=centerme" width=50%>


제일 왼쪽의 이미지는 실제 정답으로, B모델이 A모델에 비해 65% 정도 선호되는 것을 나타낸다. 두 모델의 선호도가 같음(50%)을 기준으로 약 15% 정도 더 선호된다고 볼 수 있다. 가운데 이미지는 \\(r\\)=10, \\(n\\)=100인 관측 결과이다. 샘플 수가 많기 때문에 10번의 trial 중, 유의미하게 B모델이 A모델보다 선호되는 경우가 8번으로 power는 80%라고 할 수 있다. 반면 샘플 수가 적은 마지막 이미지는 유의미한 trial은 3번으로 power는 30%라고 판단된다. 

또한 샘플 수가 많은 경우에 유의미한 결과의 평균(B 모델이 선호되는 정도)과 null hypothesis(50%)의 차이가 실제 분포의 차이와 유사하다 (파란색 라인과 검은색 라인의 차이). 샘플 수가 적은 경우, 그 차이가 과장된다 (Type-M error).

이제 위의 Power Analysis가 여러 NLP 평가 시나리오에서 어떻게 수행되는지에 대한 예시를 보여준다. 크게 classfication task와 generation task, 그리고 human evaluation 시나리오를 보여주는데, 이 글에서는 그 중 classification task와 human evaluation에 대해서 중점적으로 다뤄보고자 한다.


### Classification Task

Classification Task를 수행하는 모델들의 Accuracy 차이가 유의미한지를 판단한다. 위의 Power Analysis 시나리오를 만들기 위해서 필요한 parameter를 소개한다.

- **\\(n\\): 100, 500, ...**
- **\\(T\\) (Significance test)**

classification 결과의 유의미성을 확인하는 statistical test로 가장 많이 쓰이고 있는 것은 McNemar's test로 모델 간의 불일치하는 정도를 바탕으로 두 모델의 차이에 대한 유의성을 검증한다.

\\[\\chi^2 = \\frac{(p_{10} - p_{01})^2}{p_{10} + p_{01}}\\]

<img src="/assets/images/with-little-power-table-1.png?style=centerme" width=50%>

- **\\(e\*\\) (관심있는 평가 수치)**

모델의 accuracy 차이 (\\(\\Delta_{acc}\\))와 두 모델이 같은 결과를 예측할 확률 (\\(P_a\\)) 을 제안한다. 예를 들어, 모델 A가 baseline과 비교했을 때 0.02 만큼의 accuracy 차이를 보이고, 90%의 예측 결과가 일치한다고 하자. 이는 다시 말해 10%의 결과는 다르다는 것이며, 2%라는 성능 차이에 따라서 6%는 모델 A가 맞고, 4%는 baseline의 결과가 맞을 것이다.

with-little-power-table-4.png

- **\\(\\alpha\\): 0.05**


위의 파라미터에서 \\(n\\)을 100부터 5000까지 늘리면서 실험한 결과는 다음과 같다.

<img src="/assets/images/with-little-power-figure-3.png?style=centerme" width=45%>

두 모델이 같은 결과를 예측할 확률 (\\(P_a\\)) 이 낮을수록 두 모델의 성능차이(\\(\\Delta_{acc}\\))가 적게 나는 경우는 상식적으로 발생하기 어렵다. \\(n\\)이 5000으로 많은 경우에도 80% Power를 얻는 시점의 (\\(\\Delta_{acc}\\))는 \\(P_a\\)가 높을 때가 더 낮다. \\(n\\)이 작은 경우에는 더더욱 유의미한 성능차이를 주장하기 위해 필요한 성능 폭이 증가한다.

#### Assessing power in the literature

위의 결과를 바탕으로 다음의 질문에 대한 답을 할 수 있다. 주어진 테스트셋에 대한 baseline 대비 유의미한 성능 차이는 어느 정도일까?

GLUE와 SQuAD 2.0 에 대해 결과를 report한 모델들의 baseline 대비 평균 accuracy 차이 (\\(|\\Delta_{acc}|\\)) 를 구한다. 이를 바탕으로 선형회귀 방식을 통해 구해진 \\(P_a\\)를 도출하며, 도출된 두 값과 테스트셋의 사이즈를 바탕으로 80% power를 만족(MDE; minimum detectable effect)하는 \\(\\Delta_{acc}\\)를 구한다.

<img src="/assets/images/with-little-power-table-2.png?style=centerme" width=50%>

Est. MDE 보다 \\(|\\Delta_{acc}|\\)이 낮은 task들(WNLI, MRPC, SST-2)은 성능 개선을 주장하는 대부분의 모델이 실제로 그 task를 더 잘한다고 말할 수 없다. 평균 값이므로, 만약 어떤 모델이 Est. MDE 보다 높은 성능 개선을 보여주었다고 하더라도 데이터셋의 크기가 작기 때문에 Type-M error를 의심하게 된다. 

저자들은 이 분석 결과를 두고 다음과 같이 강하게 주장하고 있다. *~~(패기무엇)~~*

> In extreme cases, such as MRPC and SST-2, it is worth considering whether it is time to retire these datasets as the basis for model comparison.


### Likert-Scale Human Evaluations

대화와 같은 NLG 태스크에서는 적절한 evaluation metric 이 부재하기 때문에 주로 Human evaluation 결과를 report한다. Human evaluation 결과도 다양하게 measure되는데 여기서는 그 중에서 Likert-scale 로 report하는 경우에 대해서 power analysis를 수행하였다. 

#### Meta-analysis

Power Analysis에 앞서 논문에서 수행한 Human evaluation에 대해 간단한 통계량을 정리하였다. 놀랍게도 69%의 논문에서 100개 미만의 테스트셋을 사용했고, 18%만이 200개 이상의 테스트셋을 사용했다. 또한 34%의 논문에서 테스트 데이터 건당 rating 수를 제공하지 않았고, 28%의 논문에서 전체 어노테이터 수를 공개하지 않았다. 57%의 실험에서 아이템 당 3명의 어노테이터를 두었다. 

정리하면, typical 한 성능 평가 시나리오는 3명의 작업자가 100개의 결과를 평가하는 것이다. 

#### Power analysis for human Likert ratings

Power analysis를 위해서는 앞서 언급한 simulation을 수행해야 한다. 여기에는 작업자의 편차, 작업자 수, 평가한 문장 수 등의 정보가 필요하다. 이 parameters를 도출하기 위해 저자들은 기존 논문에서 report한 성능 평가 데이터를 기반으로 hierarchical mixed effects model을 사용한다. 그리고 도출된 parameter를 바탕으로 power analysis를 수행해서 아래의 그래프를 얻는다.


<img src="/assets/images/with-little-power-figure-6.png?style=centerme" width=45%>


이 그래프를 통해 충분한 power를 가진 성능 평가를 위해 몇명의 어노테이터와 몇개의 평가 인스턴스가 필요한가? 에 대한 답을 알 수 있다. 분석 결과를 정리하면 다음과 같다.

- 대부분의 human evaluation은 underpowered 일 가능성이 높다.
    - 가장 흔한 평가 방식은 3명의 어노테이터에게 100개의 데이터를 평가시키는 것이다. 보수적으로 작업자들의 편차가 크다고 한다면 유의미한 차이를 보이는 점수차이는 [0,1]로 normalize 시켰을 때 0.2 이상이어야 한다.
- 일반적인 시나리오에서 작업자들의 편차가 적더라도, 작은 성능 차이를 유의미하다고 인지하기에는 부족하다
    - 일반적인 시나리오(3명의 어노테이터, 100개의 데이터)에서 작업자들의 편차가 작다고 가정하더라도 0.1 이상의 점수차이여야 유의미하다고 볼 수 있다.
    - 10명의 작업자들이 500개의 데이터에 평가를 한다고 할 때에야 0.05 정도의 차이만으로도 유의미하다고 이야기할 수 있다.
- 대부분의 human evaluation은 제대로 결과를 report하지 않는다. 즉, 평가의 디테일이 빠져있다.
    - 그러나 (안타깝게도) 대부분의 성능 결과에서 위의 정보를 제공하지 않아 애초에 평가의 유의미함을 제대로 판단하지 못한다.



## Overall Recommendations

- baseline 결과와 비교하기 전에 power analysis 가 선행되어야 한다. Underpowered 실험은 개선되었다고 주장하면 안된다.
- 새로운 데이터셋과 shared task에서 결정하는 데이터셋 사이즈는 MDE를 고려해서 결정되어야 한다.
- GLUE task에서 성능 개선의 유의미함을 판단할 수 없는 MRPC, SST-2의 task는 평가 대상에서 빠지거나 테스트셋을 확장시켜야 한다.
- Power analysis를 위해 fine-tuned model의 checkpoint가 공개되어야 한다.
- Anonymized human evaluation 결과가 공유되어야 하고, human evalutation을 수행함에 앞서 적절한 sample size를 도출하기 위한 power analysis가 필요하다.


## References

[https://arxiv.org/abs/2010.06595](https://arxiv.org/abs/2010.06595)
