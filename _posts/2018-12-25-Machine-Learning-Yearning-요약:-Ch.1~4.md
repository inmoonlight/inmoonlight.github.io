---
title: "Machine Learning Yearning 요약: Ch.1~4"
layout: post
date: 2018-12-25 17:17
image: /assets/images/mlyearning.jpg
categories: [ 머신러닝, 스터디, 책, 요약 ]
featured: true
hidden: true
---

Deep learning으로 예전에는 풀지 못했던 문제들을 풀게 되면서 다양한 기업에서 자신들의 서비스에 Deep learning을 활용하려는 시도가 많아졌다. 그러나 생각보다 Deep learning을 서비스에 적용하는 과정은 간단하지 않다. 연구 목적으로 사용되는 것보다 데이터가 훨씬 크고, 이 때문에 한번 모델을 학습시키는데 소요되는 시간이 더 길다. 더욱이 서비스로 배포되기 위해서는 매우 정확해야 하므로 (서비스의 특징에 따른 차이는 있겠지만 대체적으로) 모델의 검증과 재학습의 iteration이 더 많이 이루어진다. 그러므로 연구실에서보다 회사에서 더 전략적인 판단의 과정이 필요하다. 

우리 팀도 NMT를 서비스에 반영하면서 다양한 고민에 부딪혀왔다. 그래서 Andrew Ng이 쓴 <<Machine Learning Yearning>>을 같이 읽고 있는데, 책 구석구석 고민해왔던 문제들에 대한 Andrew Ng 만의 해결책이 적혀있어 속이 뻥 뚫리는 기분을 맛보고 있다. (아무래도 Andrew Ng 은 Geoffrey Hinton, Yann Lecun, Joshua Bengio와 다르게 산업적인 측면에서의 Deep Learning에 더 관심이 많은 분인 듯하다.) 책 자체가 쉽게 쓰여져 있고, 각 챕터가 1~2장 정도밖에 되지 않아 부담은 적지만 큰 주제 별로 요약하면 더 많은 사람들이 쉽게 내용을 이해하고, 각자의 분야에 접목시킬 수 있을 것 같아서 이 글을 쓰게 되었다. 

*주의: 모든 요약이 그렇듯이 요약자의 주관이 개입되어 있습니다.*

<div class="breaker"></div>

Chapter 1~4는 “왜 이 책이 필요한지, 어떻게 이 책을 활용하면 좋을지”에 대한 내용이 담겼다. 목차를 보면 다음과 같다.

> Ch.1: Why Machine Learning Strategy <br>
> Ch.2: How to use this book to help your team <br>
> Ch.3: Prerequisites and Notation <br>
> Ch.4: Scale drives machine learning progress

Chapter 2는 “이 책은 한 챕터가 짧으니 인쇄해서 팀원에게 보여줘라”로 요약할 수 있고, Chapter 3은 “Supervised Learning과 Neural Network에 대한 지식~~Andrew Ng의 MOOC를 들어라~~”으로 요약가능하다. 

Chapter 1과 4를 종합하면 아래의 내용이다. 

![figure1]({{ site.baseurl }}/assets/images/mlyearning-ch1-4-figure1.png)

(서비스에 적용하기 위해 도달해야 할) 높은 정확도의 모델은 많은 데이터를, 이를 감당할 수 있을만큼 큰 Neural Network에 학습시킬 때 얻을 수 있다. 그러나 이 과정은 (당연히) 한 번에 이루어지지 않고, 다양한 시행착오가 수반된다. 모델의 성능이 좋지 않을 때, 우리가 택할 수 있는 전략은 다양하다.

* 더 많은 데이터를 얻는다.
* 다양한 training set을 구한다.
* 더 오래 학습시킨다.
* 더 큰 Neural Network를 학습시킨다.
* 더 작은 Neural Network를 학습시킨다.
* Regularization을 추가한다.
* Neural Network 구조를 바꾼다.
* …

이 중 무엇을 우선적으로 시도해 보아야할까? 그 답은 현재 내가 학습시킨 model에 있다. 그리고 이 책은 model이 남긴 단서들을 어떻게 활용해서 답을 찾을 수 있는지에 대한 내용을 담고 있다. 