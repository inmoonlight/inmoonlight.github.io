---
title: "Exploring the Limits of Transfer Learning with a Unified Text-to-Text Transformer (a.k.a. T5)"
layout: post
date: 2020-08-29 17:00
tags:
- ML
- NLP
- paper
- LM
- Google
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

최근 NLP task에서 좋은 성능을 보이는 모델은 대량의 monolingual corpus를 통해 unsupervised pre-training을 한 LM을 task에 맞게 supervised fine-tuning을 하는 transfer learning에 기반하고 있다. 같은 transfer learning framework 안에서도 다양한 모델이 존재한다. 우리가 아는 모델만 하더라도 BERT, GPT, ELMO 등이 있고, GLUE benchmark에 대해서 테스트한 점수가 있다. 

하지만 과연 점수가 더 높다고 더 좋은 모델이라고 할 수 있을까? 우리가 모델이라고 부르는 것 안에는 학습 방식 외에도 학습에 사용한 데이터셋, optimizer, 모델의 크기 등 많은 내용이 함축되어 있다. 그래서 각 모델의 아이디어 중 과연 **"어떤 특징이 좋은 모델 성능을 내는데에 도움이 되었을까?"**라고 묻는다면 쉽게 대답하기 어렵다. 

이 논문에서 소개하는 Text-to-Text Transfer Transformer (T5) 는 그 답을 찾기 위해 고안한 framework이다. 
<!--more-->

## Transfer learning framework: Text-to-Text Transfer Transformer (T5)

T5는 모든 task를 transformer의 building block으로 하는 seq2seq framework 이다 (주의! encoder-decoder 와는 다르다. sequence X가 입력되면 sequence Y가 출력되는 것일 뿐). 다양한 downstream tasks (question answering, document summarization, semtiment classification, machine translation, etc) 를 하나의 모델 안에서 해결해야 서로 다른 transfer learning 방식의 효과를 공정하게 비교할 수 있기 때문에 이와 같은 unified framework 가 제안되었다. 분석하고 싶은 내용과 무관한 특징들 -- 사용한 데이터, tokenizer, vocab size, learning rate 등 -- 은 task에 상관없이 고정된다. 

<img src="https://1.bp.blogspot.com/-o4oiOExxq1s/Xk26XPC3haI/AAAAAAAAFU8/NBlvOWB84L0PTYy9TzZBaLf6fwPGJTR0QCLcBGAsYHQ/s1600/image3.gif?style=centerme" alt="T5의 학습 framework. output이 text sentence이든, class이든, score이든 모두 text-to-text framework로 학습하는 것을 볼 수 있다. <br> source: https://ai.googleblog.com/2020/02/exploring-transfer-learning-with-t5.html" width=80%>

<img src="https://1.bp.blogspot.com/-89OY3FjN0N0/XlQl4PEYGsI/AAAAAAAAFW4/knj8HFuo48cUFlwCHuU5feQ7yxfsewcAwCLcBGAsYHQ/s1600/image2.png?style=centerme" alt="T5의 transfer learning schema. Pre-training과 Fine-tuning step으로 구분되어 있으며 기본적인 text-to-text framework는 그대로 고수한다. <br> source: https://ai.googleblog.com/2020/02/exploring-transfer-learning-with-t5.html" width=80%>


## Pre-training dataset: Colossal Clean Crawled Corpus (C4)

Transfer learning framework에 사용되는 pre-trained model은 어떤 종류의 corpus를 사용했는지, 얼만큼의 corpus를 사용했는지에 따라서도 특징이 달라진다. 이에 대한 실험을 위해 논문에서는 common crawl로 수집한 아주 많은 양의 데이터를 소개한다. 

양에 따른 모델 성능을 비교하기 위해 우선 기존의 Wikipedia corpus 보다 2배 이상 큰 사이즈의 데이터를 crawling 한다. 인터넷에서 crawling한 문서는 보통 매우 지저분하다. 이런 지저분한 데이터를 학습에 바로 사용하게 되면 side effect 가 있을 수 있기 때문에 중복 제거, 미완성 문장 제거, 공격적이거나 bias가 있는 문장 제거 등의 cleansing process를 거치고 깔끔한 데이터를 남긴다. 이 데이터셋의 이름이 Colossal Clean Crawled Corpus (C4) 이다. 여러 필터링을 거쳤음에도 Wikipedia corpus 크기의 2배 이상이라고 한다.

[TF dastasets](https://www.tensorflow.org/datasets/catalog/c4)에서 다운로드 가능하다.


## Main Contributions of the paper

앞서 소개했듯이, 이 논문에서 풀고 싶은 질문은 "다양한 NLP transfer learning framework 중에서 어떤 feature가 좋은 성능을 내는데에 도움이 될까?"이다. 따라서 조금씩 training schema를 달리해가며 여러 downstream task의 성능을 비교해야 한다. 

이 논문에서는 크게 1) [Pre-training architecture](#pre-training-architecture) 2) [Pre-training objective](#pre-training-objective) 3) [Pre-training dataset](#pre-training-dataset) 4) [Pre-training datasize](#pre-training-datasize) 5) [Training strategy](#training-strategy) 6) [Scaling strategy](#scaling-strategy)를 주목하고 있다. 그리고 중요한 Disclaimer로, ***여기서 소개하는 architecture와 objective는 GPT, ELMO 등의 구현을 아주 정확하게 replicate 하고 있지 않다***고 언급한다. 어느 정도 이 모델들의 구조에 motivate된 내용이 보이지만 정확하게 같지는 않다.


### Pre-training architecture

#### Encoder-Decoder (Baseline) vs. Language Model vs. Prefix LM

크게 세가지 모델 구조를 실험하였다. Encoder-Decoder에서 Encoder는 fully-visible attention을 적용하였고 decoder만 causal attention을 적용하였다. LM은 전부 causal attention이며, 이는 uni-directional하게 정보를 습득하는 것을 나타낸다. PrefixLM의 경우 encoder-decoder 와 유사한 것으로 보인다. input인 x에 대해서는 bi-directional하게 정보를 습득하고 y만 causal attention이 적용된다. 

<img src="/assets/images/t5-architecture-variants.png?style=centerme" width=50% alt="모든 모델의 입력과 출력은 text와 text이다. building block은 transformer이며, block과 block을 연결하는 attention의 방식이 <br> 아래 그림의 Fully-visible, Causal, Causal with prefix 중에 해당하는 것을 볼 수 있다.">
<img src="/assets/images/t5-attention-variants.png?style=centerme" width=60%>

**Q. 어떤 Pre-training model architecture 가 가장 효과적일까?**

<img src="/assets/images/t5-architecture-results.png?style=centerme" width=90%>

Encoder-Decoder architecture가 가장 효과적이었다. LM의 성능이 가장 좋지 않았는데, 이를 통해 bi-directional context를 input으로 넣어주는 것이 효과적이라는 사실을 알 수 있다.


### Pre-training objective

아래 이미지에서 확인할 수 있듯, 크게 네단계로 나누어서 생각해볼 수 있다: 1) High-level approaches 2) Corruption strategies 3) Corruption rate 4) Corrupted span length

<img src="/assets/images/t5-flowchart-unsupervised-objectives.png?style=centerme" width=60%>

#### High-level approaches

세가지 방식을 비교한다. Prefix language modeling 은 문장의 앞부분을 context로 제공하는 방식, BERT-style 은 Masked-LM (MLM) 방식, 그리고 Deshuffling 은 문장의 token을 뒤섞고 원 문장을 맞추는 방식이다. 

Objective | Inputs | Targets
-- | -- | --
Prefix language modeling | Thank you for inviting | me to your party last week .
BERT-style | Thank you \<M\> \<M\> me to your party apple week. | (original text)
Deshuffling | party me for your to . last fun you inviting week Thank | (original text)

**Q. 어떤 High-level approach 가 가장 효과적일까?**

<img src="/assets/images/t5-high-level-result.png?style=centerme" width=80%>

위의 표의 결과에 따르면 BERT-style (MLM) objective 가 가장 좋은 성능을 보인다. 


#### Corruption strategies

이번에는 High-level approaches 중 가장 좋은 성능을 보인 BERT-style approach 에 대해서 적용해 볼 수 있는 다양한 corruption strategies 에 따라 실험한다. 
총 세가지 전략이 있다. 모두 i.i.d. 로 masking을 하는 것은 동일하지만, token 단위로 masking 하는 방식이 있고 (i.i.d. noise, mask tokens) span 단위로 masking 해서 예측하는 문장은 입력에서 masking된 부분을 예측하고 아닌 부분을 masked token으로 예측하는 방식 (i.i.d. noise, replace spans (a.k.a. Denoising, Baseline)), 그리고 원문에서 token을 제외하고 제외된 부분을 예측하는 방식 (i.i.d. noise, drop tokens) 이 있다.

Objective | Inputs | Targets
-- | -- | --
i.i.d. noise, mask tokens | Thank you \<M\> \<M\> me to your party \<M\> week. | (original text)
i.i.d. noise, replace spans (a.k.a, Denoising, Baseline) | Thank you \<X\> me to your party \<Y\> week. | \<X\> for inviting \<Y\> last \<Z\>
i.i.d. noise, drop tokens | Thank you me to your party week. | for inviting last

**Q. BERT-style approach의 다양한 corruption strategy 중 무엇이 가장 효과적일까?**

<img src="/assets/images/t5-strategy-result.png?style=centerme" width=80% alt="별표시가 baseline을 의미한다">

위의 표에 따르면 Denoising 방식이 가장 좋은 성능을 낸다.

#### Corruption rate

마찬가지로 위에서 가장 성능이 좋았던 Denoising 방식에서 corruption rate을 10%, 15%, 25%, 50%로 다르게하며 살펴본다. 

**Q. 원 문장의 얼만큼을 corrupt 하는 것이 가장 좋을까?**

<img src="/assets/images/t5-corruption-result.png?style=centerme" width=80%>

표에 따르면 15%가 적당하다는 결론이 나온다. 

#### Random spans

15% 정도 denoising 방식으로 corrupt 할 때 평균적인 span의 길이를 다르게 조정해볼 수 있다. Baseline으로 사용한 i.i.d.와 2, 3, 5, 10 의 길이를 비교할 수 있다.

**Q. corrupt할 때 가장 적정한 평균 span 길이는 몇일까?**

<img src="/assets/images/t5-span-length-result.png?style=centerme" width=80%>

표에 따르면 i.i.d 에 따라 corrupt하는 것이 가장 효과적이다.



### Pre-training Dataset

Pre-training 모델의 성능은 데이터셋에 따라 달라질 수 있다. 이를 실험하기 위해 논문에서 수집한 C4와 C4, unfiltered 그리고 다른 특징을 가진 데이터셋으로 pre-training 한 후 task 마다의 성능을 비교하였다. 결과는 C4를 쓰는 것이 가장 좋았다.

<img src="/assets/images/t5-dataset-result.png?style=centerme" width=80% alt="TBC는 Toronto Book Corpus의 약자">


### Pre-training datasize

GLUE benchmark의 상위권 모델은 보통 pre-training에 사용한 datasize가 크며 모델의 사이즈도 굉장히 크다. T5는 다행히도 모델의 capacity가 큰 편이라 데이터 사이즈를 조절해가며 실험을 할 수 있었고, C4 전체를 사용했을 때 가장 성능이 좋았다. 다시 말해, 아직 모델의 capacity 가 더 크다고 이해할 수 있다. 

<img src="/assets/images/t5-datasize-result.png?style=centerme" width=80%>


### Training strategy

Transfer learning 은 Training strategy에 따라서도 성능이 달라질 수 있다. Training strategy는 Fine-tuning, Multi-task learning 방식으로 나뉘며 이를 어떻게 조합하는지 또한 전략에 포함되었다.

#### Fine-tuning methods
Fine-tuning 은 모든 파라미터를 사용했을 때 가장 성능이 좋다.
<img src="/assets/images/t5-finetuning-result.png?style=centerme" width=80%>

#### Multi-task learning (pre-training)
T5 의 task는 다양한데, 과연 각 task를 얼만큼 학습시키는 것이 좋을지에 대한 실험이다. Equal 의 경우 task dataset의 사이즈에 상관없이 같은 수의 문장을 학습시키는 방식이다. K threshold 를 사용한 경우, 기본적으로 task의 문장 사이즈만큼 샘플링하되, K 이상의 데이터는 K 만큼만 학습에 활용한다. 

결과는 sampling 하지 않는 것이 가장 좋다.
<img src="/assets/images/t5-multitask-result.png?style=centerme" width=80%>

#### Combining fine-tuning and multi-task learning

위에서 소개한 다양한 방식을 조합하여 실험하였고, multi-task pre-training과 fine-tuning 조합이 가장 좋은 성능을 보였다.
<img src="/assets/images/t5-combination-result.png?style=centerme" width=80%>



### Scaling strategy

scale-up 할 수 있는 hyperparameter의 값을 조절해가며 실험하였다. training steps, batch size, model size 를 조절한 결과, model size 를 키우고 training steps 을 늘린 경우에 가장 좋은 성능을 보였다.

<img src="/assets/images/t5-scaling-result.png?style=centerme" width=80%>



## Conclusions

위의 여러 실험들의 결과를 종합하면, **1) Encoder-Decoder architecture 2) Span prediction objective 3) C4 dataset 4) Multi-task pre-training 5) Bigger models trained longer** 의 구조로 학습할 때 가장 좋은 transfer learning 효과를 얻을 수 있다. 

## References
[1] https://youtu.be/eKqWC577WlI <br>
[2] https://ai.googleblog.com/2020/02/exploring-transfer-learning-with-t5.html
