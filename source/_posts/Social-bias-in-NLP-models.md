---
title: "Social Bias in NLP Models"
layout: post
date: 2020-11-14 11:00
tags:
- social good
- bias
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

한 스타트업에서 개발한 인공지능 채용솔루션(a.k.a. AI 면접관)을 벌써 여러 기업에서 사용하고 있다는 뉴스기사를 접하게 되었다. 해당 기업은 "성별이나 학력 등에 따른 차별 방지와 정확한 역량 추정"을 위해 5만 2천여명의 데이터를 확보하여 학습했다고 말한다. 5만 2천여명의 데이터와 차별 방지가 어떤 관련이 있는지는 모르겠지만, *많은* 양의 데이터를 사용한다는 걸 내세우고 싶었다면 대량의 데이터가 편향성을 줄이는 것과는 무관하다고 말하고 싶다. 5만 2천개보다 더 많은 데이터로 학습한 Language Model 도 편향성 문제가 있으며 이 이슈는 아직도 연구자들에 의해 활발히 연구되고 있다.
<!--more-->

## Introduction

2020년 6월 말에 다음과 같은 트윗이 올라왔다. 오바마를 모자이크한 이미지를 넣었는데 백인의 특징을 가진 이미지가 생성이 되었다는 것이다. 


<img src="/assets/images/twitter_obama.png?style=centerme" width=45%>
<br>
이 트윗은 흘러흘러 Yann LeCun의 귀에 들어간다.

<img src="/assets/images/twitter_yann.png?style=centerme" width=65% alt="그건 모델의 문제가 아니야! 데이터의 문제라구!">
<br>


그리고 이는 다시 조경현 교수님을 통해 Alice Oh 교수님의 AI & Ethics 특강 발표자료[^9]에서 아래의 문구와 함께 다시 한번 인용된다. 

> Too much blame on data curation, too little blame on algorithms

<img src="/assets/images/yann_cho.png?style=centerme" width=65%>


조경현 교수님의 이야기를 좀 더 들어보자. "물론 데이터도 문제가 있지만 알고리즘의 잘못이 전혀 없는 것은 아니다" 라고 주장한다.

아래의 장표에서는 ML 알고리즘의 solution space를 구획별로 나누어서 총 4가지로 구분하고 있다.

- Training solutions: 학습 데이터셋에서 좋은 성능을 보이는 solution space
- Shortcut solutions: 학습 및 주어진 테스트셋에서 좋은 성능을 보이는 solution space
- Intended solutions: 학습 및 주어진 테스트셋과 o.o.d 테스트셋 모두에서 좋은 성능을 보이는 solution space. 여기야말로 우리가 원하는 진짜 general knowledge가 있는 space
- Learnable solutions: 알고리즘이 학습을 통해 도달할 수 있는 solution space

<img src="/assets/images/cho_ethics_solution_space.png?style=centerme" width=75%>

우리가 원하는 것은 intended solution space와 learnable space가 만나는 점에 학습 모델이 수렴하게 만드는 것이다. 하지만 이는 쉽지 않다. data bias 가 있기 때문에 model 이 학습과정에서 data 내에 존재하는 bias 를 함유할 수 밖에 없는 것은 맞지만, 그럼에도 불구하고 이 모든 것이 data bias 의 잘못때문만은 아니라는 것이 조경현 교수님이 이야기하고 싶었던 내용이다. 

아래의 장표는 random seed 를 바꿈에 따라 learnable 모델의 학습 결과가 위치하는 solution space를 보여준다. seed 만 바꾸어도 solution space가 달라지는 것으로 미루어보아, 충분히 모델의 학습 과정을 잘 tuning 하면 원하는 solution space 쪽으로 학습할 여지가 있다고 보여진다.

<img src="/assets/images/cho_ethics_seed_solution_space.png?style=centerme" width=75%>
<br>

## Undesirable solution 의 이유는 무엇일까?

여러 이유 중 하나로, shortcut solution 중 일부는 잘못된 correlation 을 학습하기 때문이라는 점이 있다. 

"Correlation does not imply Causality" 라는 유명한 명제가 있다. 눈에 드러나는 X와 Y variable 의 관계에 모두 영향을 미치는 confounding factor Z 가 있고 hidden varirable 인 경우 우리는 실제 인과관계를 놓치고 상관관계를 인과관계처럼 학습할 위험이 있다.

<img src="/assets/images/casuality_graph.png?style=centerme" width=25%>


가장 유명한 예로, "초콜렛의 소비(X)가 많은 나라에서 노벨상(Y)을 많이 받는다." 가 있다. 실제로는 "고등교육에 투자하는 물질적, 시간적 여유가 많을수록(Z) 초콜렛 소비(X)가 많다." 와 "고등교육에 투자하는 물질적, 시간적 여유가 많을수록(Z) 노벨상(Y)을 많이 받는다."의 인과관계에서 Z variable 을 무시한채 해석하면 잘못된 정보를 학습할 수 있다는 것을 보여준다.

모델도 마찬가지다. 학습에 노출되는 데이터 분포에서는 Z가 명시적으로 보이지 않을 수 있다. 백인 남성이 상대적으로 경제적인 여유가 더 있어서 인터넷에 사진을 업로드하는 경우가 많았고, 이 때문에 웹에서 수집한 데이터 중의 대다수가 백인 남성일 수 밖에 없다고 했을 때, 이 사실은 데이터의 분포만 파악하는 모델의 입장에서는 학습하기 어려운 정보일 수 있다.


## 모든 bias 가 나쁠까?

꼭 그렇지는 않다. 때론 우리는 모델이 inductive bias 를 학습하길 바란다. 예를 들어, "나비가 코끼리를 포식하지 않는다" 는 정보는 모델 입장에서는 필요한 bias 이다. 그러나 social bias 와 같은 종류의 정보는 학습하지 않기를 기대한다.


## NLP에서는?

앞에서는 이미지를 예로 들었지만, 여러 논문에서도 NLP 모델이 social bias 를 학습하고 있다는 사실을 직/간접적적으로 보여주고 있다.

모델이 학습한 word embedding 을 분석해서 bias를 학습했음을 통해 보이거나[^1][^2], 학습 dataset 자체에 social bias 가 있음을 보이거나[^3], bias 를 측정하는 task와 metric을 제안하여 얼만큼 bias를 학습했는지 를 수치적으로 평가[^2][^4][^5][^6]한다. 새로운 bias 측정 task를 제안할 때 수반되는 dataset이 있는데, 여기에도 구축 방식에 따라 두가지 종류로 나눌 수 있다.  하나는 templated-based dataset[^4], 다른 하나는 crowdsourced dataset[^5][^6] 이다.


### 모델의 embedding space 분석

이 방법론은 PLM 이전, word2vec 이 많이 사용되었던 시기에 등장했다. word2vec을 통해 학습된 단어의 관계를 분석했을 때 social bias 가 얼마나 반영되었는지를 판단한다. 예를 들어, `man - woman ~= computer programmer - homemaker` 의 관계가 성립한다면 이 모델은 social bias 를 함유하고 있다고 볼 수 있다.

**Man is to computer programmer as woman is to homemaker? debiasing word embeddings**[^1]에서는 word2vec의 geometric bias에 대해 분석하였다.

<img src="/assets/images/bias_word2vec_figure.png?style=centerme" width=75%>


Figure 1의 왼쪽에는 she / he 와 가까이에 있는 직업 단어 목록이 있다. 그리고 각 직업 단어들을 crowd worker 에게 female-stereotypic, male-steretotypic, neutral 한지를 물었을 때의 결과와 corrleation을 구했고, 그 결과 0.51 정도의 moderate 한 결과가 나왔다. 

Figure 1의 오른쪽에는 she-he 의 관계와 유사한 단어들의 관계 목록을 보여주고 있다. 이 중에서 Gender stereotypical 한 analogy와 gender appropriate한 analogy의 수량을 비교하였고, 150 개의 관계 중 29개가 gender stereotypical 하다고 판단되었다.

### 데이터셋 자체에 함유된 Bias

학습한 모델을 분석하는 방법이 아닌, 학습하는 대상이 되는 데이터셋 자체를 model-agnostic 하게 분석하는 방법도 제안되었다.

**Social Bias in Elicited Natural Language Inferences**[^3]에서는 SNLI 데이터셋에 존재하는 bias 를 PMI 와 Likelihood ratio test of independence를 통해 통계적으로 분석하였다.

<img src="/assets/images/pmi_equation.png?style=centerme" width=50% alt="w1과 w2의 PMI. 값이 높을수록 함께 등장하는 확률이 높다고 볼 수 있음">
<br>
<img src="/assets/images/likelihood_confidence_equation.png?style=centerme" width=50% alt="X와 Y의 independence에 대한 기각 test로, 값이 낮을수록 dependency가 있는 것으로 해석할 수 있음. 자세한 수식은 논문 참고">


아래의 Table 1은 SNLI 데이터에서 gender, age, race/ethnicity/nationality 에 해당하는 단어들 각각에 대해 PMI가 높은 단어들을 나열해 놓은 것이다. woman & man, girls & boys, white man & african american 등 차이가 없어야할 개념들에 대해 PMI 상 가까운 단어들을 비교해보면 차이가 있다.

<img src="/assets/images/snli_bias_table.png?style=centerme" width=75%>

Table 2는 inference type 별로 gender words 들에 대해 PMI가 높은 순서대로 정렬한 결과다. 당연히 비슷할 수 밖에 없는 결과 - women & woman, females - 를 제외하고 gender stereotypical 한 단어들을 살펴보면 gender bias가 사뭇 드러난다. women - chat, smile 은 entailed 관계인 반면, women - dicussing, politics 는 contradiction 관계에서 자주 등장한다.

<img src="/assets/images/snli_bias_table_2.png?style=centerme" width=80%>


### Bias evaluation task

앞서 소개한 두 방법은 모델 학습결과의 embedding space에서의 단어 분포와 데이터 자체의 특성을 이용한 통계학적 관점을 활용하였다. 

이와 다르게 모델의 bias를 확인할 수 있는 방법으로 task based evaluation 이 있다. bias 를 판단할 수 있는 task 즉, dataset과 metric 을 설계해서 점수를 내고 비교하는 방식이다. 이는 다시, template based dataset 과  crowdsource-based dataset을 활용한 task로 나눠볼 수 있다.

**Gender Bias in Coreference Resolution: Evaluation and Debiasing Methods**[^4]에서는 winograd schema 를 바탕으로 bias를 측정할 수 있는 task를 제안하였다. Winograd Schema[^8]란, Terry Winograd 의 이름을 딴 schema 로, 아래의 예시와 같은 문장 형식을 일컫는다.

> The city councilmen refused the demonstrators a permit because they [feared/advocated] violence.

이 task의 정답을 맞추기 위해선, 대명사가 무엇을 가리키는지에 대한 이해가 필요하고, 이 이해는 councilment과 demonstrator의 관계에 대한 지식을 바탕으로 한다.

아래의 두가지 타입을 따르는 문장들이 bias를 테스트하기 위한 dataset으로 제안된다. `[entity1]`, `[entity2]`는 Male 혹은 Female entity 들이고, `[pronoun]`은 he/she 의 대명사이다. 문장 내에서 `[pronoun]`과 `[entity]`의 관계를 보고, pro-stereotypical 한 관계를 anti-stereotypical 한 관계보다 선호한다면 bias가 있는 모델로 판단한다.

<img src="/assets/images/bias_coref_type1.png?style=left" width=45%>
<img src="/assets/images/bias_coref_type2.png?style=right" width=45%>
<img src="/assets/images/winobias_figure.png?style=centerme" width=55%>



**StereoSet: Measuring stereotypical bias in pretrained language models**[^5] 은 4개의 bias domain - gender, profession, race, religion - 에 대해서 crowdsource 방식으로 수집된 데이터와 bias 측정 metric 을 제안하고 있다. crowdsource 방식은 template 기반 방식 대비 real-world 에 더 가깝게 데이터셋을 수집할 수 있다는 장점이 있다.

StereoSet에서는 LM이 배워야 할 지식과 배우지 않길 기대하는 지식 (social bias) 을 구분해서 학습했는지 여부를 테스트한다. Context Association Test (CAT) 라고 명칭한 Test는 Intrasentence와 Intersentence  두 종류의 테스트로 구성되어 있다.

<img src="/assets/images/stereoset_example.png?style=centerme" width=45%>


그리고 모델이 각 test에 대해 답변한 결과를 ranking problem (option 1을 선택한 비율이 option 2를 선택한 비율보다 많았는가, 적었는가) 으로 pose 시켜서 최종 bias score를 낸다.

앞서 언급했듯이, LM이 배워야 할 지식과 배우지 않길 기대하는 지식 (social bias) 을 구분해서 학습했는지 여부를 테스트하기 때문에 lms 와 ss 를 구분하였고, 이 두 점수를 종합해서 최종 CAT score 를 도출한다.

- **Language Modeling Score (lms)**
    - model has to rank the meaningful association higher than meaningless associaton
    - score: percentage of instances in which a language model prefers
    - ideal: 100
- **Stereotype Score (ss)**
    - score: percentage examples in which a model prefers a stereotypical association over an anti-stereotypical association
    - ideal: 50
- **Idealized CAT Score (icat)**
    - combine both lms and ss

        \\[icat = lms *\\frac{min(ss, 100-ss)}{50}\\]

    - ideal: 100 (when lms: 100 and ss: 50)
    - fully biased: 0 (when lms: 0, ss: 100 or 0)
    - random model: 50 (when lms: 50, ss: 50)

**CrowS-Pairs: A Challenge Dataset for Measuring Social Biases in Masked Language Models**[^6]은 EMNLP 2020에 억셉된 논문으로, stereoset과 마찬가지로 template 기반이 아닌 crowndsourced 데이터셋으로 구성되어 있다. 크게 9가지의 bias에 대해 문장 하나는 stereotyping한 것, 다른 하나는 덜 stereotyping 한 것으로 구성한다. 두 문장의 거리는 매우 가까워야 한다.

<img src="/assets/images/crowspair_table.png?style=centerme" width=80%>

평가의 경우, 차이가 있는 token을 제외한 나머지 token (=unmodified token)을 순차적으로 masking 해서 각각의 log-likelihood를 구한 뒤 그 합을 최종 점수로 가진다. 

<img src="/assets/images/crowspair_figure.png?style=centerme" width=85%>

<img src="/assets/images/crowspair_score.png?style=centerme" width=35%>

### Limitations on "Bias in NLP" researches

**Language (Technology) is Power: A Critical Survey of “Bias” in NLP**[^7]에서는 기존의 146개 "bias in NLP" paper에 대한 survey를 진행하면서 이전 연구들에 대한 비판과 이를 극복할 수 있는 방향으로의 연구를 제안하였다. 

비판의 포인트는 크게 3가지이다. 

1. "bias"의 정의에 대한 논의 부재
2. "bias"로 인해 발생하는 문제에 대한 고민 부족 (movitations are often vague, inconsistent, and lacking in  normative reasoning)
3. 기존 학계에서 논의되고 있는 "bias"와의 약한 연결성

예를 들어, "racial bias"에 대한 다음 기존 연구들을 살펴보자. 다루고 있는 주제는 racial bias 지만 실제로는 보다 협소한 African-American English (AAE) 에 대해서 다루고 있다. (그래놓고 제목에 racial bias를 적어둔 것은 뭘까? Asian 차별에 대해서는?) 그리고 같은 AAE에 대해 다음과 같이 다양한 방식으로 bias 가 존재한다고 주장하고 있다. 

- African American과 연관된 이름은 pleasant words보다 unpleasant words에 더 가까움
- POS tagger, Language Identification, dependency parser 에서 AAE에 연관된 term이 포함되는 경우 덜 정확함
- toxicity detection system 이 AAE 와 연관된 feature 가 있는 트윗을 더 offensive 하다고 판단내리는 경향이 있음

기존 연구들 중 어떤 것도 AAE 혹은 racial hierarchies in the US에서의 racial hierarchies, raciolinguistic 분석을 언급하지 않았다. 또, AAE를 누가 이야기하는가에 따라서도 bias 여부를 다르게 판단할 수 있으나 이에 대한 고려는 없이 오직 text 만 놓고 판단하였다. 어떤 맥락에서 AAE 가 문제가 될 수 있고, bias 되었다고 판단할 수 있는지에 대한 고려는 없었다.

개인적으로도 단순 데이터/모델 결과 분석의 방법론이 애매하다는 생각이 든다. bias가 있는 문장/단어에 대해 다른 문장/단어와 다른 결과를 도출한다고 이야기하고 있지만, 임의의 특성들에 대해서도 심층적으로 분석해보면 똑같은 결과가 나올 것 같다. 

그리고 Crowdsourcing 으로 만들어진 dataset이 bias 가 없음을 보장하는 내용도 부족했다. US 에 살고 있는 사람들이 annotator 로 참여했지만 보수적인 주의 사람들이 더 많았다거나, bias에 대한 지식이 부족한 사람들이 더 많았다면 문제가 될 여지가 있다.

## So?

올바른 방식으로 bias in NLP 주제를 tackle 하기 위해서는 언제 Biased 모델이 문제를 일으키는지를 우선 고민해봐야할 것 같다.

Biased model은 언제 문제가 될까? (Open Question) Bias 의 범위가 넓으므로 gender bias 에 국한해서 생각해보자. 만약 gender bias detection model 을 간단하게 KcBERT를 korean-hatespeech-dataset (gender bias) task에 finetuning[^10] 한 것으로 사용한다고 했을 때, 모델이 틀리는 gender-biased 예문이 o.o.d test set 에서 많이 등장한다면 문제가 될까? 혹은 정답은 제대로 맞추었더라도 잘못된 단어를 queue로 받아서 맞추는 것이라면 문제이지 않을까?

문제가 된다면, 이 문제는 무엇으로부터 기인할까?

- KcBERT 가 학습한 데이터로부터 오는 문제
- KcBERT를 finetuning 하는 방법으로부터 오는 문제
- finetuning dataset 의 문제
    - 제작 시 "gender bias"를 annotator 의 직관에 맡기기보다는 guideline을 바탕으로 tagging 되었다고 볼 수 있다. 따라서 annotator의 bias를 최소화했다고 보여진다.
- dataset 이 작기 때문에 real world 를 충분히 반영하지 못해서 나타나는 문제
    - Human-in-the-loop 으로 풀어볼 수 있지 않을까?
    - static benchmark 가 쉽게 stale 해지는 것을 막기 위해 제안된 DynaBench[^11]라는 dynamic benchmark framework을 참고하면 어떨까? DynaBench는 user들이 특정 task를 푸는 목적으로 학습된 모델이 틀리는 예문을 생성하고, 이를 바탕으로 다시 모델이 학습해서 지속적으로 발전할 수 있도록 만들어졌다.

간단하게 학습시킨 KcBERT finetuning model 을 통해 예측한 결과를 보았을 때 주어진 testset 과 생각해볼 수 있는 간단한 예문에 대해서는 나쁘지 않은 결과를 보임을 확인하였다. DynaBench를 benchmarking 한 웹사이트를 만들어보면 재밌는 결과를 수집할 수 있지 않을까? 


## References

[^1]: [Man is to computer programmer as woman is to homemaker? debiasing word embeddings](https://proceedings.neurips.cc/paper/2016/file/a486cd07e4ac3d270571622f4f316ec5-Paper.pdf) 
[^2]: [Semantics derived automatically from language corpora contain human-like biases](https://arxiv.org/abs/1608.07187) 
[^3]: [Social Bias in Elicited Natural Language Inferences](https://www.aclweb.org/anthology/W17-1609/) 
[^4]: [Gender Bias in Coreference Resolution: Evaluation and Debiasing Methods](https://www.aclweb.org/anthology/N18-2003/) 
[^5]: [StereoSet: Measuring stereotypical bias in pretrained language models](https://arxiv.org/abs/2004.09456)
[^6]: [CrowS-Pairs: A Challenge Dataset for Measuring Social Biases in Masked Language Models](https://arxiv.org/abs/2010.00133) 
[^7]: [Language (Technology) is Power: A Critical Survey of “Bias” in NLP](https://arxiv.org/abs/2005.14050)
[^8]: https://en.wikipedia.org/wiki/Winograd_Schema_Challenge
[^9]: https://kyunghyuncho.me/social-impacts-bias-of-ai/
[^10]: https://github.com/inmoonlight/detox/tree/master
[^11]: https://dynabench.org/
