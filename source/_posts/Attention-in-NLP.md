---
title: "Attention in NLP"
layout: post
date: 2020-01-27 21:00
thumbnail: /assets/images/attention_camera.jpg
tags:
- ML
- NLP
categories: 
- ML
- NLP
toc: true
widgets:
   - type: toc
     position: left
   - type: categories
     position: left
   - type: recent_posts
     position: left
---

{% blockquote  Raymond Mooney %}
You can't cram the meaning of a whole %&!# sentence into a single &!#* vector!
{% endblockquote %}

Attention은 single vector에 한 문장의 의미를 완벽하게 담을 수 없기 때문에 필요한 순간에, 필요한 정보를 사용하기 위한 방법이다. 기본적으로 **query** vector와 **key** vector의 조합으로 attention weight가 계산된다. 여기서 "조합"의 방법에는 크게 두가지가 있다. 하나는 Additive Attention으로 query vector와 key vector에 feed-forward network를 적용한 것이고, 다른 하나는 Dot-Product Attention으로 문자그대로 query vector와 key vector의 dot-product를 이용한 것이다. 이번 글에서는 각 Attention 방법들과 이들의 장단점을 소개하려고 한다.

<!--more-->


## Additive Attention

Additive Attention은 query vector와 key vector의 조합으로 attention 값을 얻을 때 단일 hidden layer를 가진 feed-forward network를 이용한다. query vector ($q$) 와 key vector ($k$) 가 같은 dimension을 가질 필요가 없으며, dimension의 크기에 상관없이 좋은 성능을 보인다는 장점이 있다.

### Bilinear[^2]

\\[a(q, k)= q^{T} Wk\\]

\\(q\\) 와 \\(k\\) 에 \\(W\\) 를 곱하는 방법이다. 
\\(q\\) 를 linear transform 시킨 후, \\(k\\) 와 dot-product를 한 것과 같다.


### Multi-layer Perceptron[^3]


\\[a(\\boldsymbol{q}, \\boldsymbol{k})=\\boldsymbol{w}\_{2}^{\\top} \\tanh \\left(W\_{1}[\\boldsymbol{q} ; \\boldsymbol{k}]\\right)\\]


\\(q\\) 와 \\(k\\) 를 concat시킨 후 single hidden layer와 activation 함수로 tanh를 사용한 feed-forward network를 사용하였다. 
이 방법은 익숙할 것이다.
왜냐하면 [Neural Machine Translation by Jointly Learning to Align and Translate](https://arxiv.org/abs/1409.0473)에서 이 attention을 이용했기 때문이다. 
아래의 오른쪽 수식에서 \\(a\\)는 alignment model로 위에서 언급한 feed-forward network와 같은 역할을 한다.

<img src="/assets/images/seq2seq_attention.png?style=centerme" width=55%>


## Dot-Product Attention

Dot-Product attention은 \\(\boldsymbol{q}^{\top} \boldsymbol{k}\\)을 기반으로 attention weight를 구하는 방법이다.
Additive attention과는 달리 hidden layer를 곱하는 과정이 추가되지 않아서 연산 속도와 space 측면에서 효율적이다.
하지만 반드시 \\(q\\)와 \\(k\\)의 dimension이 같아야 한다는 제약조건이 있으며, dimension이 클 때 학습에 방해가 된다는 단점이 있다.

### Dot-Product


\\[a(\\boldsymbol{q}, \\boldsymbol{k})=\\boldsymbol{q}^{\\top} \\boldsymbol{k}\\]

\\(q\\)와 \\(k\\)의 elment-wise product의 합이다.
이 연산의 특성 상, 반드시 dimension이 같아야 한다.

만약 \\(q\\)와 \\(k\\)의 각 요소가 독립적이고 평균이 0, 분산이 1 인 분포의 random variable이라면, \\({q}^{\\top}{k}\\) 는 평균이 0이고 분산이 dimension의 크기인 분포를 따른다. 
분산이 증가되면 dot-product에 softmax를 취했을 때 어떤 값은 굉장히 크지만 대부분의 값은 굉장히 작게 만든다.
작은 값들은 back-propagation 시 gradient도 작기 때문에 전체적으로 학습이 잘 되지 않게 만든다.
따라서 dimension이 큰 경우 성능이 좋지 않다.

### Scaled Dot-Product[^4]

\\[a(\\boldsymbol{q}, \\boldsymbol{k})=\\frac{\\boldsymbol{q}^{\\top} \\boldsymbol{k}}{\\sqrt{|\\boldsymbol{d}|}}\\]


Scaled Dot-Product는 [Attention is All You Need](https://papers.nips.cc/paper/7181-attention-is-all-you-need.pdf) 에서 처음으로 소개된 방법이다.
Dot-Product의 dimension이 클 때 학습이 잘 되지 않는 단점을 극복하게 위해 dot-product 결과를 \\(q\\)의 dimension \\(d\\) (\\(k\\) 의 dimension 이기도 하다) 의 root 값으로 나누어주었다.[^5]


## Conclusions

Additive attention은 dimension에 상관없이 좋은 결과를 내고, attention을 계산하는 재료인 query vector와 key vector의 dimension에 상관없이 사용할 수 있다. 
하지만 최근의 NLP trend라고 할 수 있는, 대량의 데이터를 굉장히 큰 모델로 학습시키는 방법에는 안그래도 많은 연산량에 부담이 되는 방법이다.
그래서 계산의 부담이 적으면서 dimension이 큰 경우에 대해서도 좋은 성능을 보이는 scaled dot-product 기반의 attention이 잘 쓰이는 편이다.



## References

[^1]: 전반적인 내용은 Graham Neubig의 [NN4NLP Attention 강의](http://phontron.com/class/nn4nlp2019/assets/slides/nn4nlp-09-attention.pdf)를 참고했습니다.
[^2]: [Effective Approaches to Attention-based Neural Machine Translation](https://www.aclweb.org/anthology/D15-1166.pdf)
[^3]: [Neural Machine Translation by Jointly Learning to Align and Translate](https://arxiv.org/abs/1409.0473)
[^4]: [Attention is All You Need](https://papers.nips.cc/paper/7181-attention-is-all-you-need.pdf)
[^5]: root로 나누어 준 이유는 scaled dot-product의 결과를 평균이 0, 분산이 1 인 분포로 만들기 위해서다.

