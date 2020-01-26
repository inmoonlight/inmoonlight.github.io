---
title: "General Language Understanding Evaluation (GLUE) benchmark"
layout: post
date: 2019-12-22 22:22
image: /assets/images/glue.png
categories: [ nlp, dataset, 자연어처리, 데이터셋 ]
featured: false
---

General Language Understanding Evaluation benchmark, 줄여서 GLUE benchmark 라고 불리는 이 데이터셋은 NLP 분야에서 Language Model 검증을 위해 사용된다.
ICLR 2019와 BlackboxNLP workshop 2018에 모두 publish 되었으며, [전자](https://openreview.net/pdf?id=rJ4km2R5t7)는 설명이 상세하고 [후자](https://www.aclweb.org/anthology/W18-5446.pdf)는 요약되어 있다. 
이 글은 가장 최근(2019.2.22)에 업데이트된 [arXiv에 있는 논문](https://arxiv.org/pdf/1804.07461.pdf)을 기반으로 작성되었다. 

<div class="breaker"></div>

### Table of Contents

[1. GLUE overall](#1-glue-overall)<br>
&nbsp;&nbsp;&nbsp;&nbsp;[Corpus of Linguistic Acceptability (CoLA)](#corpus-of-linguistic-acceptability-cola)<br>
&nbsp;&nbsp;&nbsp;&nbsp;[Stanford Sentiment Treebank (SST-2)](#stanford-sentiment-treebank-sst-2)<br>
&nbsp;&nbsp;&nbsp;&nbsp;[Microsoft Research Paraphrase Corpus (MRPC)](#microsoft-research-paraphrase-corpus-mrpc)<br>
&nbsp;&nbsp;&nbsp;&nbsp;[Quora Question Pairs (QQP)](#quora-question-pairs-qqp)<br>
&nbsp;&nbsp;&nbsp;&nbsp;[Semantic Textual Similarity Benchmark (STS-B)](#semantic-textual-similarity-benchmark-sts-b)<br>
&nbsp;&nbsp;&nbsp;&nbsp;[Multi-Genre NLI corpus (MNLI)](#multi-genre-nli-corpus-mnli)<br>
&nbsp;&nbsp;&nbsp;&nbsp;[The Recognizing Textual Entailment (RTE)](#the-recognizing-textual-entailment-rte)<br>
&nbsp;&nbsp;&nbsp;&nbsp;[The Stanford Question Answering NLI (QNLI)](#the-stanford-question-answering-nli-qnli)<br>
&nbsp;&nbsp;&nbsp;&nbsp;[The Winograd Schema Challenge NLI (WNLI)](#the-winograd-schema-challenge-nli-wnli)<br>
[2. Download](#2-download)<br>
[3. Leaderboard](#3-leaderboard)<br>
[References](#references)

<div class="breaker"></div>


## [1. GLUE overall](#table-of-contents)

GLUE는 총 9개의 task로 구성되었으며 각 task는 언어의 특정한 성질을 평가하기 위한 목적으로 만들어졌고, 최종 점수는 각 task 별 점수의 평균 값을 가져간다.
task는 크게 3가지 - Single-Sentence Tasks (CoLA, SST-2), Similarity and Paraphrase Tasks (MRPC, QQP, STS-B), Inference Tasks (MNLI, RTE, QNLI, WNLI) - 로 구분할 수 있다.
세부 task에 대해 살펴보기 전에 전반적인 task의 특징을 아래의 표에 정리했다.
원 논문에 정리되어 있는 것을 바탕으로 재구성하였고 직접 다운로드 받은 데이터를 기준으로 측정했기 때문에 corpus의 size가 다를 수 있다. 


| data                                          | \|train\| | \|dev\| | \|test\| | domain   | input     | task   | metrics   |
|-----------------------------------------------|---------|-------|--------|-----------|---|------|----------------------------------|
| [Corpus of Linguistic Acceptability (CoLA)](#corpus-of-linguistic-acceptability-cola)     | 8.5k    | 1.0k  | 1.2k   | linguistics literature  | single-sentence    | - grammatical acceptability <br> - binary classification <br> (grammatical / ungrammatical)     | Matthews correlation             |
| [Stanford Sentiment Treebank (SST-2)](#stanford-sentiment-treebank-sst-2)   | 67k     | 872   | 1.8k   | movie reviews    | single-sentence   | - sentiment <br> - binary classification <br> (positive / negative)  | acc.   |
||
| [Microsoft Research Paraphrase Corpus (MRPC)](#microsoft-research-paraphrase-corpus-mrpc)   | 3.7k    | 408   | 1.7k   | news    | two sentences  | paraphrase  | acc./F1 |
| [Quora Question Pairs (QQP)](#quora-question-pairs-qqp)    | 364k    | 40k   | 391k   | social QA questions    | two sentences    |  paraphrase    | acc./F1     |
| [Semantic Textual Similarity Benchmark (STS-B)](#semantic-textual-similarity-benchmark-sts-b) | 5.8k    | 1.5k  | 1.4k   | news <br> caption <br> forum   | two sentences   | - sentence similarity <br> - regression   | Pearson / Spearman corr.   |
||
| [Multi-Genre NLI corpus (MNLI)](#multi-genre-nli-corpus-mnli)   | 393k    | 20k   | 20k    | fiction <br> face-to-face <br> government <br> letters <br> 9/11 <br> oxford university press (oup) <br> slate<br> telephone <br> travel <br> verbatim | two sentences      | ternary classification <br> (entailment / contradiction / neutral) | matched acc. / mismatched acc. |
| [The Recognizing Textual Entailment (RTE)](#the-recognizing-textual-entailment-rte)      | 2.5k    | 276   | 3.0k   | news <br> wikipedia   | two sentences                      | binary classification <br> (entailment / not_entailment)           | acc.                             |
| [The Stanford Question Answering NLI (QNLI)](#the-stanford-question-answering-nli-qnli)    | 105k    | 5.5k  | 5.5k   | wikipedia    | two sentences (question, sentence) | binary classification <br> (entailment / not_entailment)           | acc.                             |
| [The Winograd Schema Challenge NLI (WNLI)](#the-winograd-schema-challenge-nli-wnli)      | 634     | 71    | 146    | fiction books     | two sentences                      | binary classification <br> (entailment / not_entailment) | acc.                             |

<br>

<div class="breaker"></div>

### [Corpus of Linguistic Acceptability (CoLA)](#1-glue-overall)[^1]

공개된 언어학 문헌(publised liguistics literature)에서 추출된 약 21k 문장들로 구성되어 있다.
이 문장들은 문법적으로 옳은지, 그른지가 표기되어 있다.

```
1 They drank the pub dry.
0 * They drank the pub.
```

문법적으로 옳고 그름을 판단하는 기준은 다양하다. 
아래의 표는 corpus를 제작하면서 기준에서 포함된 것들과 제외된 것들을 나타낸다. 

<div style="text-align:center">
<img class="image" src="{{ site.baseurl }}/assets/images/cola_dataset_description.png" width="80%">
</div>
<br>

* **Included**
<br> 
**(a) Morphological Violation**: "should leave" 가 올바른 표현이지만 "should leaving"으로 작성되었다. 동사의 형태(verbal inflection)가 맞지 않는 경우에 해당한다.
<br>
**(b) Syntactic Violation**: "What did Bill buy?" 혹은 "Bill bought potatoes and _" 이 되어야 한다. 통사 구조가 틀린 경우에 해당한다.
<br>
**(c) Semantic Violation**: 의미적으로 말이 되지 않는 문장에 해당한다.

* **Excluded**
<br>
**(d) Pragmatic Anomalies**: grammar와 상관없는 외부 지식이 필요하므로 제외되었다.
<br>
**(e) Unavailable Meanings**: 문장만보고는 판단이 애매하므로 제외되었다.
<br>
**(f) Prescriptive Rules**: 사람도 누군가의 가르침없이는 터득하기 어려운 rule이기 때문에 제외되었다.
<br>
**(g) Nonce Words**: "arrivable"과 같이 typical word-level NLP 모델의 vocab에는 등장하지 않는 단어가 포함된 경우이다. NLP 모델의 scope이 아니라고 판단되어 제외되었다.

#### testset and metrics

testset은 In-Domain과 Out-of-Domain으로 구성되어 있다. 
In-Domain은 training set이 추출된 source와 같은 source에서, Out-of-Domain은 training set이 추출되지 *않은* source에서 구성되었다. 
GLUE는 원래 구분된 두 testset을 하나로 합쳐 단일 testset을 구축하였고 총 1,160 문장이다.

<div style="text-align:center">
<img class="image" src="{{ site.baseurl }}/assets/images/cola_by_source.png" width="40%">
</div>
<br>

이 task의 평가는 unbalanced binary classification task에서 사용되는 Matthews correlation으로 한다.


<div class="breaker"></div>



### [Stanford Sentiment Treebank (SST-2)](#1-glue-overall)

`rottentomatoes.com`의 영화 리뷰 corpus로 구성되었으며 AMT(Amazon Mechanical Turk)를 통해 리뷰의 sentiment가 labeling 되었다.
1은 긍정, 0은 부정을 나타낸다.

```
that loves its characters and communicates something rather beautiful about human nature 1
on the worst revenge-of-the-nerds clichés the filmmakers could dredge up 0
```

#### testset and metrics

일반적인 binary classification 문제로 accuracy를 통해 평가한다.

<div class="breaker"></div>



### [Microsoft Research Paraphrase Corpus (MRPC)](#1-glue-overall)

MRPC는 온라인 뉴스에서 추출된 문장들로 구성되었으며 2개의 문장이 의미적으로 같은지 다른지를 평가하는 task이다.

```
They had published an advertisement on the Internet on June 10 , offering the cargo for sale , he added . 
On June 10 , the ship 's owners had published an advertisement on the Internet , offering the explosives for sale .
1

Yucaipa owned Dominick 's before selling the chain to Safeway in 1998 for $ 2.5 billion . 
Yucaipa bought Dominick 's in 1995 for $ 693 million and sold it to Safeway for $ 1.8 billion in 1998 .
0
```

#### testset and metrics

testset이 label이 불균등(68% positive,  32% negative)하므로 accuracy와 F1 score를 metric으로 한다.


<div class="breaker"></div>



### [Quora Question Pairs (QQP)](#1-glue-overall)[^2]

`https://www.quora.com/`의 질문들로 구성되었으며, 두 개의 질문이 의미상 같은지 다른지가 표기되어있다.


```
How do you start a bakery?
How can one start bakery business?
1

What are natural numbers?
What is a least natural number?
0
```

#### testset and metrics

MRPC와 마찬가지로 불균등(37% positive, 63% negative)하므로 accuracy와 F1 score가 metric으로 활용된다.


<div class="breaker"></div>




### [Semantic Textual Similarity Benchmark (STS-B)](#1-glue-overall)

문장의 유사도는 번역, 요약, 문장 생성, QA, 대화 모델링 등등 다양한 NLP 분야에서 중요하게 다뤄진다. 
STS shared task는 모델이 문장들의 유사도를 얼마나 잘 파악하는지를 평가하기 위해 등장하였고, 2012년부터 2017년까지 매년 개최되었으며 그 때마다 다른 dataset이 사용되었다.
이 때문에 각 연도의 데이터셋을 적절히 조합한 common evaluation set으로 STS-B가 소개되었다.

이 전의 task와는 다르게 STS는 regression task이다. 
human annotator들은 두 문장의 의미적인 유사도를 1~5점으로 평가하였고 모델은 score를 예측해야한다.

```
A plane is taking off.  An air plane is taking off.     5.000
Three men are playing chess.    Two men are playing chess.      2.600
A man is smoking.       A man is skating.       0.500
```

#### testset and metrics

Regression task이므로 human label과의 Pearson correlation으로 평가된다.

<!-- collection of sentence pairs drawn from news headlines video and image captions and NLI data. -->

<!-- 
<div style="text-align:center">
<img class="image" src="{{ site.baseurl }}/assets/images/stsb_dataset_description.png" width="50%">
</div>
<br> -->


<div class="breaker"></div>



### [Multi-Genre NLI corpus (MNLI)](#1-glue-overall)[^3]

MNLI는 SNLI(Stanford NLI) dataset의 단점을 개선시킨 데이터셋이다.
SNLI는 image caption으로만 구성되었기 때문에 장면을 표현하는 짧고 간단한 문장이 많고 NLU(Natural Language Understanding) task와 무관한 단어들이 많이 등장한다.
그래서 NLU task의 benchmark로 사용되기는 어렵기 때문에 다양한 도메인(논문에서는 genre라고 표현)의 조합인 MNLI benchmark dataset이 등장하였다.

<div style="text-align:center">
<img class="image" src="{{ site.baseurl }}/assets/images/mnli_dataset_description.png" width="90%">
</div>
<br>

위의 표에서 나와있듯이 MNLI는 총 10개의 Genre로 구성되었다.
Fiction을 제외한 9개의 domain은 Open American National Corpus에서 추출되었고 Fiction은 fiction literature에서 가져왔으며 mystery, humor, sci-fi 등 그 안에서도 다양한 장르로 구성되었다.

> OANC data constitutes the following nine genres: transcriptions from the Charlotte Narrative and Conversation Collection of two-sided, in-person conversations that took place in the early 2000s (FACE-TO-FACE); reports, speeches, letters, and press releases from public domain government websites (GOVERNMENT); letters from the Indiana Center for Intercultural Communication of Philanthropic Fundraising Discourse written in the late 1990s–early 2000s (LETTERS); the public report from the National Commission on Terrorist Attacks Upon the United States released on July 22, 2004 2 (9/11); five non-fiction works on the textile industry and child development published by the Oxford University Press (OUP); popular culture articles from the archives of Slate Magazine (SLATE) written between 1996–2000; transcriptions from University of Pennsylvania’s Linguistic Data Consortium Switchboard corpus of two-sided, telephone conversations that took place in 1990 or 1991 (TELEPHONE); travel guides published by Berlitz Publishing in the early 2000s (TRAVEL); and short posts about linguistics for non-specialists from the Verbatim archives written between 1990 and 1996 (VERBATIM).

> For our tenth genre, FICTION, we compile several freely available works of contemporary fiction written between 1912 and 2010, spanning various genres, including mystery (The Mysterious Affair at Styles, 3 Christie, 1921; The Secret Adversary, 4 Christie, 1922; Murder in the Gun Room, 5 Piper, 1953), humor (Password Incorrect, 6 Name, 2008), western (Rebel Spurs, 7 Norton, 1962), science fiction (Seven Swords, 8 Shea, 2008; Living History,9  Essex, 2016; The Sky Is Falling, 10 Del Rey, 1973; Youth, 11 Asimov, May 1952), and adventure (Captain Blood, 12 Sabatini, 1922).


선별된 문장을 premise로 두고 human annotator들이 premise와 같은 결론을 도출하는 문장(entailment), 반대되는 문장(contradiction), 두 경우 모두 아닌 문장(neutral)을 생성하고 label을 단다.

```
How do you know? All this is their information again.   
This information belongs to them.      
entailment

Vrenna and I both fought him and he nearly took us.     
Neither Vrenna nor myself have ever fought him.      
contradiction

There was nothing like that emotion now.        
There are few emotions that come close.      
neutral
```

#### testset and metrics

testset은 CoLA처럼 matched(in-domain)와 mismatched(cross-domain)로 구성되었다. 
mismatched에는 9/11, FACE-TO-FACE, LETTERS, OUP, VERBATIM처럼 training set에는 없는 domain이 포함되어 있다. (위의 표 참고)
각각의 경우를 나누어서 accuracy로 평가한다. 


<div class="breaker"></div>



### [The Recognizing Textual Entailment (RTE)](#1-glue-overall)

RTE도 STS처럼 RTE1부터 RTE7까지의 데이터셋에서 만들어졌다. 
구체적으로는 RTE1, RTE2, RTE3, RTE5로 구성되었고, 나머지 데이터셋 중 RTE4는 공개되지 않아서, RTE6와 7은 NLI task로는 부적합해서 제외했다고 한다. 
취합하는 과정에서 일부는 세 개의 class, 일부는 두 개의 class로 labeling이 되어있어 이를 일괄적으로 두 개의 class(entailment, not_entailment)로 구분지었다.

```
Swansea striker Lee Trundle has negotiated a lucrative image-rights deal with the League One club.      
Lee Trundle is in business with the League One club.    
entailment

No Weapons of Mass Destruction Found in Iraq Yet.       
Weapons of Mass Destruction Found in Iraq.      
not_entailment
```

#### testset and metrics

일반적인 binary classification 문제이므로 accuracy로 측정한다.

<div class="breaker"></div>



### [The Stanford Question Answering NLI (QNLI)](#1-glue-overall)

Stanford에서 구축한 Machine Comprehension 목적의 QA Dataset, a.k.a SQuAD,을 NLI task에 맞게 변형한 데이터셋이다.
SQuAD는 wikipedia에서 paragraph를 가져와서 annotator들이 적절한 질문을 던지는데 이에 대한 답을 paragraph 내에 있는 문장, 구, 단어로 답할 수 있게 구성되었다.
QNLI는 질문과 paragraph 내의 한 문장을 비교하여 이 둘이 entailment되었는지 아닌지를 판단하도록 바뀌었다.

```
What two things does Popper argue Tarski's theory involves in an evaluation of truth?   
He bases this interpretation on the fact that examples such as the one described above refer to two things: assertions and the facts to which they refer.   
entailment

Who was elected as the Watch Tower Society's president in January of 1917?      
His election was disputed, and members of the Board of Directors accused him of acting in an autocratic and secretive manner.       
not_entailment
```

#### testset and metrics

일반적인 binary classification 문제이므로 accuracy로 측정한다.

<div class="breaker"></div>


### [The Winograd Schema Challenge NLI (WNLI)](#1-glue-overall)

이 데이터셋도 entailment를 평가하는 목적으로 만들어졌다.
original sentence와 이 문장에서 대명사를 일반명사로 치환한 문장 사이의 entailment가 있는지 없는지가 label로 달려있다.
아래 예시의 첫 번째 문장에서 "it" had a hole의 it이 "The carrot"으로 바뀐 문장이 두 번째 문장이고 이 두 문장의 관계가 entailment 되어 있으므로 label 1이 달린다.

```
I stuck a pin through a carrot. When I pulled the pin out, it had a hole.       
The carrot had a hole.  
1

John was jogging through the park when he saw a man juggling watermelons. He was very impressive.       
John was very impressive.       
0
```

#### testset and metrics

[GLUE FAQ](https://gluebenchmark.com/faq)의 12번 문항에는 WNLI에서 이상한 결과를 얻을 수 있는 이유가 적혀있다.
같은 문장이 포함된 다른 example 끼리는 반대의 label이 달려있는데 이 때문에 training set에 overfit된 모델은 dev set에서 성능이 매우 나쁠 수 있다는 것이다.
실제로 [BERT](https://arxiv.org/pdf/1810.04805.pdf)는 이 이유로 WNLI의 성능은 report 하지 않았다.


<div class="breaker"></div>


## [2. Download](#table-of-contents)

[링크](https://github.com/nyu-mll/jiant/blob/master/scripts/download_glue_data.py)에 있는 python script를 다운로드한 이후 실행시키면 된다.
지정한 dir에 전체 task를 받을 수도 있고 일부 task만 받을 수도 있다.

```bash
python download_glue_data.py --data_dir data --tasks all
```

<div class="breaker"></div>


## [3. Leaderboard](#table-of-contents)

여태까지 제출한 모델의 성능은 [GLUE leaderboard](https://gluebenchmark.com/leaderboard)에 정리되어있다.

<div style="text-align:center">
<img class="image" src="{{ site.baseurl }}/assets/images/glue_leaderboard.png" width="95%">
<figcaption class="caption">2019.12.22 기준 top 3</figcaption>
</div>
<br>

Leaderboard에는 순기능과 역기능이 모두 공존하지만, 아직까지는 순기능이 더 많다고 생각한다.
상대적으로 공정하게 비교할 수 있는 데이터셋이고 덕분에 다양한 Language Model이 주목받을 수 있었기 때문이다.
너무 낡아버리기 전에 새로운 데이터셋이 나와야한다고도 생각했는데, Neurips 2019에 "Stickier Benchmark"라는 부제와 함께 [SuperGLUE](http://papers.nips.cc/paper/8589-superglue-a-stickier-benchmark-for-general-purpose-language-understanding-systems)가 등장했다!!

이로 인해 열릴 새로운 LM들의 등장을 기대해본다 :)


## References

[^1]: [https://arxiv.org/pdf/1805.12471.pdf](https://arxiv.org/pdf/1805.12471.pdf)
[^2]: [https://www.quora.com/q/quoradata/First-Quora-Dataset-Release-Question-Pairs](https://www.quora.com/q/quoradata/First-Quora-Dataset-Release-Question-Pairs)
[^3]: [https://www.aclweb.org/anthology/N18-1101.pdf](https://www.aclweb.org/anthology/N18-1101.pdf)