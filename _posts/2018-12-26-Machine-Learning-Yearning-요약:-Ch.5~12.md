---
title: "Machine Learning Yearning 요약: Ch.5~12"
layout: post
date: 2018-12-26 01:24
image: /assets/images/mlyearning.jpg
categories: [ 머신러닝, 스터디, 책, 요약 ]
featured: true
hidden: true
---

지난 글 >>
[Machine Learning Yearning 요약: Ch.1~4](https://inmoonlight.github.io/%EB%A8%B8%EC%8B%A0%EB%9F%AC%EB%8B%9D/%EC%8A%A4%ED%84%B0%EB%94%94/%EC%B1%85/%EC%9A%94%EC%95%BD/2018/12/25/Machine-Learning-Yearning-%EC%9A%94%EC%95%BD-Ch.1~4.html)

<div class="breaker"></div>

책을 읽으면서 Andrew Ng이 쉽게 읽힐 수 있는 책을 만들기 위해 노력한 점이 보였다. Chapter를 소제목으로 묶어 두었고, 소제목으로 묶인 마지막 Chapter는 앞서 기술한 내용을 요약한 Takeaways가 담겨 있었다. 

<div class="breaker"></div>

이번에 요약할 Chapter 5~12 의 소제목은 **Setting up development and test sets** 이고, 아래와 같은 제목으로 구성되어 있다. 

> Ch.5: Your development and test sets <br>
> Ch.6: Your dev and test sets should come from the same distribution <br>
> Ch.7: How large do the dev/test sets need to be? <br>
> Ch.8: Establish a sinlge-number evaluation metric for your team to optimize <br>
> Ch.9: Optimizing and satisficing metrics <br>
> Ch.10: Having a dev set and metric speeds up iterations <br>
> Ch.11: When to change dev/test sets and metrics <br>
> Ch.12: Takeaways: Setting up development and test sets

Machine Learning / Deep Learning 모델을 전략적으로 향상시키기 위해서는 Idea를 내고, Code로 구현하고, 검증하고, 재학습시키는 일련의 iteration이 정확하고 빠르게 진행되어야 한다. 

![figure1]({{ site.baseurl }}/assets/images/mlyearning-ch5-12-figure1.png)

코드를 통해 구현된 모델을 실험한 후, 올바른 방향으로의 재학습이 이루어지기 위해서는 error analysis가 필수적이다. 신뢰도 있는 Error analysis는 신뢰도 있는 dev/test set과 evaluation metric을 통해 얻어진다. 그래서 이번 Chapter는 좋은 dev/test set 그리고 evalutaion metric에 대한 조건에 대해 서술하고 있다. 

<div class="breaker"></div>

### Dev/Test set
먼저, training set, dev set, test set에 대해 정의해보자.

> Training set: 우리가 알고리즘을 학습할 때 사용하는 데이터
> Dev set: 알고리즘의 parameter를 튜닝하고, feature를 선택하는 등의 결정을 내리기 위한 데이터
> Test set: 알고리즘의 성능을 평가하기 위한 데이터

즉, 모델이 잘하고 있는지 아닌지의 여부는 test set에 달려있다. 그리고 “잘한다”라는 기준은 서비스에 대해서는 당연히 “사용자의 만족도”이다. 그러므로 test set은 “사용자의 만족도를 향상시키기 위해 세운 팀의 목적”과 부합해야 하며, “서비스가 잘 하고 싶은 영역”과 “서비스 사용자를 통해 들어올 법한 데이터”를 반영해야 한다.

그리고 dev set은 반드시 test set과 같은 distribution을 가져야 한다. 왜냐하면 모델의 세부사항은 dev set을 통해 결정되기 때문에 만약 distribution이 다르다면 “죽 쒀서 개 준 꼴”이 날 것이다. 뿐만 아니라 추후 dev set에서 error analysis를 통한 모델의 진단이 어려워진다. 

적절한 Dev set의 크기는 학습한 모델들의 차이를 인지할 수 있을 정도이어야 한다. 보통 1,000에서 10,000개 정도가 적당한데, 이미 기본적인 모델의 성능이 높고, 0.01%의 차이에 민감해야 한다면 10,000개보다 커야 한다. 

Test set의 경우에는 모델의 성능에 대해 높은 confidence를 줄 수 있을 정도의 사이즈가 적당하다. 교과서에서는 전체 데이터의 30%를 쓰라고 하지만, 빅데이터 시대에는 그보다 비율이 작아도 된다. 

### Evluation metric
2개 이상의 metric을 쓰는 것보다 단일 metric을 쓰는 것이 좋다. A와 B의 측면에서 평가해야하는데 A를 버리라는 뜻이 아니다. A와 B를 적절히 섞은 (평균, 가중 평균 등) 단일 수치를 만들라는 것이다. 단일 metric이어야 빠르게 모델의 성능을 비교하고 결정할 수 있다. 

그러나 A와 B와 같은 metric을 적절히 섞기 어려운 경우가 있을 수 있다. 아래 그림의 정확도와 running time처럼 (`Accuracy - 0.5*RunningTime`와 같은 단일 metric이 부자연스럽다).

![figure1]({{ site.baseurl }}/assets/images/mlyearning-ch5-12-figure2.png)

이럴 땐, satisficing metric과 optimizing metric을 나누어서 생각하면 된다. 총 N개의 metric이 존재할 때, N-1 개의 satisficing metric과 1개의 optimizing metric으로 나눈다. 위의 예제의 경우, Running time은 satisficing metric으로, Accuracy는 optimizing metric으로 두었을 때, Running time은 특정 기준을 넘기기만 하면 되고, Accuracy는 그 안에서 최대인 모델이 가장 좋은 모델이라고 평가할 수 있다. 

### Dev/Test set 및 evaluation metric의 업데이트
위의 조건을 만족하는 dev/test set과 evaluation metric을 결정하는 것은 쉽지 않을 것이다. 하지만 완벽하진 않더라도 빠르게 무엇이라도 구축하고, 업데이트하는 것이 좋다.

dev/test set이 더이상 실제 사용자의 distribution을 대표하지 않을 때, dev set에 너무 overfit 되어버렸을 때, metric이 팀의 방향을 대변하지 못할 때, 과감히 변화를 시도하자. 