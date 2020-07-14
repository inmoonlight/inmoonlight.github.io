---
title: "Positional Encoding in NLP"
layout: post
date: 2020-01-26 17:00
thumbnail: /assets/images/positional_encoding.png
tags:
- ML
- NLP
categories: 
- ML
- NLP
toc: true
widgets:
   - type: toc
     position: right
   - type: categories
     position: right
   - type: recent_posts
     position: right
---

Positional encoding 혹은 position encoding은 모델 구조에서 자연스럽게 sequential information을 얻지 못하는 경우에 대해 정보를 강제하는 방식이다. 보통 sequential data를 Recurrent Neural Network (RNN) 외의 다른 모델로 다루고 싶을 때 많이 사용된다. 이번 글에서는 Convolutional Neural Network (CNN), End-to-End Memory Network (MemN2N), Transformer에서 sentence embedding을 위해 사용된 positional encoding에 대해 소개하려고 한다.
<!--more-->

## Introduction

일반적으로 NLP 모델은 각 문장을 구성하는 token을 one-hot vector가 아닌 distributed vector로 표현한다. 그 이유는 distributed representation이 1) 비슷한 의미지만 다른 lexical form을 가진 token을 더 잘 표현할 수 있기 때문이고, 2) embedding dimension을 감소시킬 수 있기 때문이다.

문장의 embedding은 문장을 이루는 각 token의 embedding을 조합하는 방식으로 얻어진다. 이 때 position에 대한 정보가 없다면 모델은 `handful of chocolate`과 `chocolate of handful`을 같은 의미로 인식하게 된다. RNN은 모델 구조 자체에 time information이 녹아져 있다. 그래서 `handful --> of --> chocolate` 의 순서가 담긴 sentence embedding을 자연스럽게 얻을 수 있다. 반면 CNN이나 Attention 기반의 Transformer는 순서에 대한 정보를 강제해야 하고, 이 때 positional encoding이 사용된다.

Positional encoding (PE) 은 token embedding vector에 곱해지는 정보로, sentence에서 해당 token이 어디에 위치해 있는지를 나타낸다. J개의 token \\(t_j \\in \\mathbb{R}^d\\)으로 구성된 sentence \\(s = [t_1, t_2, ..., t_J]\\)가 있다고 하자. PE \\(\\in \\mathbb{R}^{J \\times d}\\) 의 row \\(j\\)마다 다른 값을 가지도록 하여 문장 맨 처음의 `handful`과 맨 뒤의 `handful`을 다르게 인식하도록 한다.

PE를 구성하는 방식에는 크게 두 종류가 있다. 하나는 학습기반, 다른 하나는 position과 dimension을 입력으로 한 함수를 이용하는 방법이다.


## Learned Positional Embeddings

학습기반의 PE를 구성하는 방식은 Convolutional Sequence to Sequence Learning (ConvS2S)[^1]에서 사용되었다. 평균 0, 표준편차 0.1을 따르는 normal distribution으로 initialize되고 학습을 통해 position 정보를 배우길 기대한다.

PE를 encoder와 decoder 모두에 사용한 경우, encoder에만 사용한 경우, decoder에만 사용한 경우, 아예 사용하지 않은 경우로 나누어 번역 task에 실험해보았을 때의 결과는 다음과 같다.

<img src="/assets/images/learned_pe_table.png?style=centerme" width=50%>

BLEU를 기준으로 분석해보면 encoder에서의 PE역할이 decoder보다 조금 더 중요하다. PE를 아예 쓰지 않을 때의 점수가 가장 낮지만 점수 차이를 생각해보면 모델 성능에는 크게 영향을 미치지 않는다고 해석해 볼 수 있다.

학습 기반이므로 학습 시 다루지 않았던 길이의 문장이 입력으로 들어온 경우, 외삽이 불가능하다는 단점이 있다.[^5]



## Function-based Positional Encoding 

함수 기반의 PE는 문장에서 몇 번째에 위치한 토큰인지, 토큰의 embedding dimension이 무엇인지를 정해주면 값이 정해진다. 이 때, 다른 위치의 정보가 같은 값으로 mapping되지 않아야 한다. 어떻게 구현할 수 있을까?

### The Intuition

0부터 15까지의 숫자를 2진법으로 나타내보자.

<img src="/assets/images/PE_intuition.png?style=centerme" width=55%>

다른 색으로 구분지어 표현한 2진수의 자리수마다 다른 주기를 가지는 것을 볼 수 있다. 붉은색은 주기가 1이고, 노란색은 주기가 2, 초록색은 주기가 4, 파란색은 주기가 8이다.

위 예시에서의 자리수를 embedding dimension이라고 생각해보면 PE에도 같은 원리를 확장시켜볼 수 있다.

### In MemN2N

End-to-End Memory Network (MemN2N)[^4]에서는 아래의 함수를 사용했다.

\\[PE\_{k j}=(1- \\frac{j}{J})-\\frac{k}{d}(1- \\frac{2j}{J})\\]

\\[j \\in {1, ..., J}\\]

\\[k \\in {1, ..., D}\\]

임의의 문장 *"The same representation is used for questions, memory inputs and memory outputs."*에 적용되는 PE를 시각화해보면 다음과 같다.[^5]

<img src="/assets/images/PE_example_1.png?style=centerme" width=90%>

여기서는 dimension에 관계없이 같은 주기를 가지지만 시작값이 전부 다르다. 결과적으로는 position마다 다른 vector를 곱하게 되어 position 정보를 전달할 수 있다.

다른 문장 길이를 가지는 경우에 대해서 적용해보면 어떨까? 이번에는 *"We therefore propose a second representation that encodes the position of words within the sentence."*에 대해 시각해보았다.[^5]

<img src="/assets/images/PE_example_2.png?style=centerme" width=90%>

position이 늘어난만큼 position encoding 값의 변화도가 줄었다. J는 문장마다 달라지므로 첫번째, 두번째의 절대적인 위치보다는 각 순서를 구분짓기 위한 목적에 치중하였다.

ConvS2S에서와는 달리 MemN2N에서 PE의 효과는 꽤나 컸던 것으로 보인다.

<img src="/assets/images/MemN2N_PE.png?style=centerme" width=90%>

### In Transformer

Attention is all you need[^5]에서 사용된 PE는 **주기**함수로 유명한 sin 함수와 cos 함수를 기반으로 한다. (a.k.a, sinusoidal functions)

\\[
\\begin{aligned} 
P E\_{(\\text {pos, 2k} )} &=\\sin \\left(\\text {pos} / 10000^{2 k / d}\\right) \\\\ P E\_{(\\text {pos,2k+1})} &=\\cos \\left(\\text {pos} / 10000^{2 k / d}\\right) 
\\end{aligned}
\\]

잠시 고등학교 때 배운 수학을 떠올려보자. \\(sin(ax + b)\\) 의 주기는 \\(\\frac{2\\pi}{|a|}\\) 이다. 따라서 PE의 특정 position vector 값의 주기는 \\(2\\pi \\cdot 10000^{2 k / d}\\) 와 같다.

MemN2N에서의 PE와는 달리, position vector의 주기가 vector의 dimension마다 변화한다. 전체 벡터 크기(\\(d\\))가 128이라고 가정할 때, \\(k\\)가 작을수록 주기가 짧고 \\(k\\)가 클수록 주기도 길어진다. (아래 그림 참고)

<img src="/assets/images/positional_encoding.png?style=centerme" width=85% alt="Image credit: https://kazemnejad.com/blog/transformer_architecture_positional_encoding">

왜 Transformer에서는 MemN2N과 다르게 sinusoidal 함수를 썼을까? 논문에서 그 이유를 짧게 기술하고 있다.

> We chose this function because we hypothesized it would allow the model to easily learn to attend by relative positions, since for any fixed offset \\(k\\), \\(P E\_{pos+k}\\) can be represented as a linear function of \\(P E\_{pos}\\). 

sinusodial 함수의 특징을 이용해 첫번째, 두번째마다 같은 position 정보를 주면서도 \\(n + k\\)번째 vector가 \\(n\\)번째 vector와 관계가 있을 때 이를 학습할 수 있는 여지를 남겨주기 위함이다. (참고로 이에 대한 수학적인 증명은 [이 article](https://timodenk.com/blog/linear-relationships-in-the-transformers-positional-encoding/)에 기술되어 있다.)

또한 PE vector 간의 distance는 대칭적이고 거리에 따라 일정한 비율로 감소한다. Transformer의 self-attention 연산에서 빛을 발하는 특징이다.

<img src="/assets/images/PE_pros_1.png?style=centerme" width=50% alt="Image credit: https://kazemnejad.com/blog/transformer_architecture_positional_encoding">

## Conclusions

PE는 크게 학습을 통해 정해질 수 있고 미리 지정한 함수로 정해질 수도 있다. 학습을 통한 방식은 학습시 보지 않았던 새로운 길이가 등장했을 때 외삽이 불가능하지만 함수 기반의 PE는 가능하다. 함수도 어떤 함수를 쓰느냐에 따라 종류가 구분되는데, 절대적인 위치에 따라 같은 값을 가지면서도 상대적 위치의 관계도 학습할 수 있는 \\(sin\\)과 \\(cos\\) 기반의 함수가 가장 좋은 방법이라고 생각된다.


## References

[^1]: [https://arxiv.org/abs/1705.03122](https://arxiv.org/abs/1705.03122)
[^2]: [https://kazemnejad.com/blog/transformer_architecture_positional_encoding/](https://kazemnejad.com/blog/transformer_architecture_positional_encoding/)
[^3]: [https://arxiv.org/abs/1503.08895](https://arxiv.org/abs/1503.08895)
[^4]: [https://github.com/inmoonlight/notebooks/blob/master/notebooks/2020-01-26-MemN2N-Position-Encoding.ipynb](https://github.com/inmoonlight/notebooks/blob/master/notebooks/2020-01-26-MemN2N-Position-Encoding.ipynb)
[^5]: [https://papers.nips.cc/paper/7181-attention-is-all-you-need.pdf](https://papers.nips.cc/paper/7181-attention-is-all-you-need.pdf)