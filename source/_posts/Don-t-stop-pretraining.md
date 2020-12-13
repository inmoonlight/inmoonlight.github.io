---
title: "Don't Stop Pretraining: Adapt Language Models to Domains and Tasks"
layout: post
date: 2020-11-30 15:00
tags:
- ML
- NLP
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

다양한 출처의 데이터로 학습한 pretrained model이 NLP task에서 좋은 성능을 보여주고 있다. 하지만 아직 주어진 labeled data 의 크기나 target domain의 코퍼스와 사전학습 코퍼스의 유사도가 특정 task의 결과에 얼마나 영향을 미치는지에 대해 알려진 바가 없다. 또한 RoBERTa와 같은 LM이 정말 다양한 task에 generalize될만큼의 다양한 source로 학습되었는지도 확실하지 않다. 이 논문에서는 pretrained model을 풀고자 하는 특정 task의 domain에 tailor시켜서 추가로 학습시키면 더 좋은 성능을 보일 수 있을까? 라는 질문에 대한 답을 하고 있다.
<!--more-->

결과는 "그렇다"이다. 보다 구체적으로는 다음의 3가지 findings가 있었다. 첫번째, pretrained model을 추가로 in-domain 데이터에 학습시키는 것이 성능향상에 도움을 준다 (Domain-Adaptive PreTraining; DAPT). 두번째, DAPT 이후로 풀고자 하는 task의 unlabeled data에 대해 추가로 학습하는 것(Task-Adaptive PreTraining; TAPT)도 성능향상에 도움을 준다. 세번째, DAPT를 할 corpus가 없을 때 간단한 data selection strategies으로 task corpus를 augment 한 후 TAPT를 수행해도 성능 향상을 보인다.


## Experimental settings
- Pretrained model: RoBERTa-base
    - 160GB heterogeneous data로 학습
    - bookcorpus, stories, wikipedia, and realnews
- Domains:
    - biomedical papers (BIOMED)
    - cs papers (CS)
    - newstext from realnews (NEWS)
    - Amazon reviews (REVIEWS)
- Tasks:
   - 2 text classfication tasks per each domain  
<img src="/assets/images/dapt-dataset.png?style=centerme" width=80%>

### Domain 의 기준?

내가 아는 한에서는, domain 이라는 용어에 대한 확실한 정의는 없다. 어떤 논문은 domain에 대해 두루뭉술하게 언급하고 넘어가고, 어떤 논문은 뭐라도 정의하고 넘어가는 경우가 있는데 이번 논문에서는 domain에 대한 논의가 중요하다보니 선정한 4개의 도메인과 pretraining domain이 서로 상이함을 밝힐 필요가 있었다.

<img src="/assets/images/dapt-vocab-overlap.png?style=centerme" width=40%>
<br>

위의 이미지는 Vocab overlap 을 통한 domain의 similarity 를 구한 heatmap이다. 빈번한 10K 의 vocab 중 얼마나 겹치는지를 나타내고 있으며 결과는 직관과 일치하는 것으로 보인다. PT corpus는 news, reviews와 가장 유사하며 cs와 가장 거리가 멀다. 또 reviews와 cs의 거리가 가장 멀고, news와 reviews는 cs에 비해 상대적으로 더 유사하다.


## DAPT Results

### Domain 별 LM loss 변화

<img src="/assets/images/dapt-lm-loss.png?style=centerme" width=80%>
<br>

각 도메인에 RoBERTa를 12.5K steps 씩 학습시킨 후 LM loss의 전후를 비교하였다. domain similarity 가 가장 높았던 news를 제외한 나머지 도메인에서 marginal한 성능향상이 있었다.

### PLM vs. DAPT vs. ~DAPT 모델의 classification 결과

<img src="/assets/images/dapt-table-3.png?style=centerme" width=40%>
<br>

LM loss 의 변화가 시사했던 것처럼 BM과 CS 도메인에서의 효과가 가장 컸다. 하지만 이 변화가 단순히 더 많은 데이터에 노출되었기 때문인지 아닌지를 판단하기 위해 out-of-domain corpus로 DAPT를 시킨 후의 결과와 비교했다. news의 경우 CS로 DAPT한 LM을, reviews의 경우 BIOMED LM을, cs의 경우 news LM을, biomed의 경우 reviews LM을 사용했다. DAPT가 ~DAPT 모델보다 모든 경우에서 더 좋은 성능을 낸다. 심지어 RoBERTa와 비교해보면 ~DAPT 의 결과는 더 나빠지는 경향을 보인다. 이는 단순히 더 많은 데이터에 노출되는 것이 항상 모든 도메인의 결과에서 효과적이지 않다는 사실을 시사한다.

### Fuzzy 한 domain boundary

Vocab overlap 결과에서도 알 수 있지만 domain 이라는 것이 무자르듯 나뉘는 것이 아니다. 지금까지의 실험은 news와 reviews 도메인을 구분했지만, reviews corpus가 news corpus에 아예 도움을 주지 않는다고 볼 수 없다.

## TAPT

Domain 보다 더 협소한 범위의 Task에 대해서도 DAPT와 같은 효과가 있는지를 검증하였다. DAPT와 비교해서 더 적은 corpus로 학습한다는 단점이 있지만 더 task relevant한 corpus라는 장점이 있다. 만약 최종 성능이 비슷하다면 TAPT가 더 값싼 학습방식이라고 볼 수 있다. 여기서는 labeled training data를 사용해서 second phase PLM을 진행했다.

### PLM vs. DAPT vs. TAPT vs. DAPT+TAPT 모델의 classification 결과

<img src="/assets/images/dapt-table-5.png?style=centerme" width=80%>
<br>

TAPT의 경우, corpus  사이즈를 고려해서, second phase of pretraining은 100 epoch만 진행하였다. 

- **PLM vs TAPT**
    - 결과를 보면, TAPT를 진행한 LM으로 classficiation task를 수행한 경우가 RoBERTa-base 보다 항상 결과가 더 좋다.
- **DAPT vs TAPT**
    - 하지만 DAPT와 비교해보면 언제나 더 좋은 결과를 보여주는 것은 아니었다.
- **PLM vs DAPT + TAPT**
    - DAPT 이후 TAPT 를 적용하는 것이 언제나 최고의 성능을 보여준다.
    - **PLM에 많이 활용된 데이터가 AGNews와 IMDB와 비슷했다면 그 둘의 성능 폭이 작은 것이 이해가 되지만, 아니라면 HyperPartisan이랑 Helpfulness의 성능향상이 BIOMED와 비슷하다는 점에서 꼭 PLM의 학습 코퍼스의 도메인과 성능향상이 관련있다고 보긴 어렵다.**
    - 그 task 를 잘하기 위해서는 마지막에 weight 를 옮겨주는 것이 필요하지 않을까?


### Cross-Task Transfer

<img src="/assets/images/dapt-table-6.png?style=centerme" width=80%>
<br>

같은 도메인 내의 2 task 간 transfer 효과가 있는지에 대해서 살펴보았다. BIOMED 내의  RCT, ChemProt task를 예로 들면, Transfer-TAPT는 RCT의 unlabeled data로 pretraining한 이후, ChemProt의 결과를 본 것이다. 모든 경우 Transfer-TAPT의 결과가 TAPT보다 낮았다.


### Augmenting Training data for TAPT

task: RCT, HyperPartisan, IMDB

**case 1) target task의 labeled data와 같은 distribution의 unlabeled target task corpus (by human)**

<img src="/assets/images/dapt-table-7.png?style=centerme" width=40%>
<br>

Curated TAPT의 경우 TAPT보다 더 많은 task corpus (unlabeled)로 PLM을 진행하였고, 좋은 성능을 보였다. DAPT와 함께 진행하게 되면 그 효과는 더욱 확실하다.

**case 2) Automated Data selection for TAPT**

task setup 당시에 large unlabeled corpus 조차 풀리지 않는 경우가 있다. 이 때, 자동으로 관련있는 데이터를 찾아서 이를 기반으로 학습하게 되면 어떨지를 실험하였다.

하지만 large in-domain corpus 여야 하며, 이 중에서 task와의 접점이 있는 task-relevant data를 찾는 것이다. 그리고 embedding space 내에서 접점을 찾는 것이므로 경량화모델이 필요하다. 여기서는 vampire model을 사용하였다.

<img src="/assets/images/dapt-table-8.png?style=centerme" width=40%>
<br>

- **rand-TAPT vs kNN-TAPT**
    - kNN > rand-TAPT
- **TAPT vs automated data selection**
    - 방법이 무엇이든 추가 데이터를 활용하는 것이 나쁘지는 않음
    - 아직 RoBERTa에게는 더 학습할 수 있는 여지가 남아있다고도 해석 가능 (데이터는 많을수록 좋다)
- **DAPT vs 500NN-TAPT**
    - 약 500개의 데이터만 사용해도 DAPT 효과를 어느 정도 낼 수 있음
   

## Conclusions

<img src="/assets/images/dapt-table-9.png?style=centerme" width=40%>
<br>

- Task 문제를 더 잘 풀기 위해서 관련된 distribution 의 데이터로 추가 학습을 하는 것이 효과적이다
- Task-specific data 일 필요는 없다. Domain이 비슷하면 효과를 볼 수 있다.
- 다만 아쉬운 건, PLM 자체를 학습시킬 때 DAPT+TAPT에 사용한 데이터를 활용하면 점수가 어떻게 변하는지 알 수 없었다는 점이나, 이번 논문의 scope에 들어갈 필요는 없었다고도 생각한다.


## References

https://www.aclweb.org/anthology/2020.acl-main.740.pdf
