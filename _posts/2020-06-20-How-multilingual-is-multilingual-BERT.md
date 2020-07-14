---
title: "How multilingual is Multilingual BERT?"
layout: post
toc: true
toc_sticky: true
date: 2020-06-20 17:00
image: /assets/images/mbert.png
categories: [ machine-learning, nlp, multilingual, BERT, 머신러닝, 자연어처리 ]
featured: false
---

"How multilingual is Multilingual BERT?" [^1] 는 ACL 2019 에 억셉된 논문으로, Telmo Pires 가 Google AI Residency 프로그램 중에 작성하였다. Unbabel 에서 Autumatic Post-Editing (APE) 쪽 연구를 진행했었던 연구자였고, 그래서 multilingual BERT에 대해 분석한 논문을 쓴 것이 아닐까?

<div class="breaker"></div>

### Table of Contents

[Abstract](#abstract)<br>
[Motivation](#motivation)<br>
[Main Idea](#main-idea)<br>
[Main Findings](#main-findings)<br>
[Detailed Experiments and Results](#detailed-experiments-and-results)<br>
&nbsp;&nbsp;&nbsp;&nbsp;[0. Preliminaries](#0-preliminaries)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[0.1. NER task](#01-ner-task)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[0.2. POS task](#02-pos-task)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[0.3. Code-switching (CS) and Transliterate (Tlit) task](#03-code-switching-cs-and-transliterate-tlit-task)<br>
&nbsp;&nbsp;&nbsp;&nbsp;[1. `M-BERT` 의 cross-lingual transferability 는 vocab overlap 때문일까? ➡️   NO](#1-m-bert-의-cross-lingual-transferability-는-vocab-overlap-때문일까--no)<br>
&nbsp;&nbsp;&nbsp;&nbsp;[2. `M-BERT`의 cross-lingual transferability 는 언어의 typological 특징 때문일까? ➡️   YES](#2-m-bert의-cross-lingual-transferability-는-언어의-typological-특징-때문일까--yes)<br>
&nbsp;&nbsp;&nbsp;&nbsp;[3. `M-BERT`의 cross-lingual transferability 는 CS 혹은 Tlit 까지 적용될 수 있을까? ➡️  CS (YES) / Tlit (NO)](#3-m-bert의-cross-lingual-transferability-는-cs-혹은-tlit-까지-적용될-수-있을까--cs-yes--tlit-notable-of-contents)<br>
&nbsp;&nbsp;&nbsp;&nbsp;[4. `M-BERT`의 feature space](#4-m-bert의-feature-space)<br>
[My Thoughts on the Results](#my-thoughts-on-the-results)<br>
&nbsp;&nbsp;&nbsp;&nbsp;[1. vocab overlap 실험에서 `EN-BERT`와의 비교는 정당한가?](#1-vocab-overlap-실험에서-en-bert와의-비교는-정당한가)<br>
&nbsp;&nbsp;&nbsp;&nbsp;[2. SOV order 가 중요한 점이었을까?](#2-sov-order-가-중요한-점이었을까)<br>
&nbsp;&nbsp;&nbsp;&nbsp;[3. Feature space](#3-feature-space)<br>
&nbsp;&nbsp;&nbsp;&nbsp;[4. 논문의 분석 내용](#4-논문의-분석-내용)<br>
[References](#references)

<div class="breaker"></div>

## [Abstract](#table-of-contents)
In this paper, we show that Multilingual BERT (`M-BERT`), released by Devlin et al. (2018) as a single language model pre-trained from monolingual corpora in 104 languages, is **surprisingly good at zero-shot cross-lingual model transfer**, in which task-specific annotations in one language are used to fine-tune the model for evaluation in another language. To understand why, we present a large number of probing experiments, showing that **transfer is possible even to languages in different scripts**, that **transfer works best between typologically similar languages**, that monolingual corpora can train models for code-switching, and that the model can find translation pairs. From these results, we can conclude that `M-BERT` does create multilingual representations, but that these representations exhibit systematic deficiencies affecting certain language pairs.

<div class='breaker'></div>

## [Motivation](#table-of-contents)
Pretrained LM 이 다양한 NLP downstream task 에서 좋은 성능을 보여주었다. Pretrained LM의 probing 연구들은 모델이 학습한 representation 이 syntactic and named entity 에서 특히 유용한 정보를 가지고 있다는 사실을 보여주었지만, 이 연구들은 영어에 대해서만 집중적으로 진행되어 왔다. (2019 년 6월 기준) (참고로, BERT 는 2018년 11월에 첫 release) 

<div class='breaker'></div>

## [Main Idea](#table-of-contents)
영어에 대해서 Pretrained LM이 가지고 있는 정보, 그 중에서도 syntactic and named entity information 이 언어에 상관없이 잘 generalize 되는지 분석
- Main task: NER, POS
- Main method: Zero-shot cross-lingual transfer (Multilingual BERT 모델을 한 언어에 대해 finetuning 시키고, 다른 언어에 대해 같은 task의 성능을 평가)

<div class='breaker'></div>

## [Main Findings](#table-of-contents)
* 아래 내용으로 학습한 Multilingual BERT (`M-BERT`) 는 NER과 POS task 에 대해 cross-lingual transfer ability 가 좋다.
    - language identifier 없이 
    - 위키피디아의 문서로 학습 (140 개 언어)
    - w/ shared word piece vocab

- 모든 언어쌍에 대해 zero-shot transfer 가 잘 된 것은 아니었는데 그렇다면 왜 이런 차이가 발생할까?
    - finetuning 언어와 evaluation 언어의 vocab overlap 때문은 아님
    - 오히려 언어의 typological 특징 때문
      - typological 특징도 여러 종류가 있는데 (여기서는 subject/object/verb order, adjective/noun order에 대해서만 결과를 보여줌), 그 중 **SVO order** 에 가장 큰 영향을 받음
    - transfer 하기 위한 언어에 대해 학습한 적이 있을 때 transfer 가능
    - `M-BERT` 의 중간 layer (8/12) 에서  cross-lingual information 이 높음

<div class='breaker'></div>

## [Detailed Experiments and Results](#table-of-contents)

**Main Question**: 무엇이 `M-BERT`의 zero-shot cross-lingual transferability를 만들어내는가?


### [0. Preliminaries](#table-of-contents)
#### [0.1. NER task](#table-of-contents)
  - dataset: [CoNLL-2002](https://www.clips.uantwerpen.be/conll2002/ner/), [2003 dataset](https://www.clips.uantwerpen.be/conll2003/ner/), Google in-house dataset
    - There are four types of phrases: person names (`PER`), organizations (`ORG`), locations (`LOC`) and miscellaneous names (`MISC`)
  - lang: `nl`, `es` (2002) / `en`, `de` (2003) 총 4개 + in-house dataset with 16 languages (Arabic, Bengali, Czech, German, English, Spanish, French, Hindi, Indonesian, Italian, Japanese, Korean, Portuguese, Russian, Turkish, and Chinese)
  - example (`en`)
     - NER tagged plain text: `[PER Wolff ] , currently a journalist in [LOC Argentina ] , played with [PER Del Bosque ] in the final years of the seventies in [ORG Real Madrid ] .`
     - NER data:
```
        Wolff B-PER
            , O
    currently O
            a O
   journalist O
           in O
    Argentina B-LOC
            , O
       played O
         with O
          Del B-PER
       Bosque I-PER
           in O
          the O
        final O
        years O
           of O
          the O
    seventies O
           in O
         Real B-ORG
       Madrid I-ORG
            . O
```

#### [0.2. POS task](#table-of-contents)
- dataset: Universal Dependencies (UD) data (Universal dependencies v1: A multilingual treebank collection, Nivre, 2019) for 41 languages
  - Arabic, Bulgarian, Catalan, Czech, Danish, German, Greek, English, Spanish, Estonian, Basque, Persian, Finnish, French, Galician, Hebrew, Hindi, Croatian, Hungarian, Indonesian, Italian, Japanese, Korean, Latvian, Marathi, Dutch, Norwegian (Bokmaal and Nynorsk), Polish, Portuguese (European and Brazilian), Romanian, Russian, Slovak, Slovenian, Swedish, Tamil, Telugu, Turkish, Urdu, and Chinese
- evaluation set: Multilingual Parsing from Raw Text to Universal Dependencies, Zemman et al. 2017 (CoNLL 2017 shared Task)
  - [example (`ko`)](https://raw.githubusercontent.com/UniversalDependencies/UD_Korean-PUD):
```
# sent_id = n01007012
# text = 이 부분에서 게임과 우리 일상 생활 사이의 유사점을 찾을 수 있습니다.
# text_en = There are parallels to draw here between games and our everyday lives.
# translit = .i .bu.bun.e.seo .ge.im.gwa .u.ri .il.sang .saeng.hwal .sa.i.yi .yu.sa.jeom.eul .chaj.eul .su .iss.seub.ni.da.
1	이	_	DET	DT	_	2	det	_	Translit=.i|LTranslit=_
2	부분에서	부분	NOUN	NN+CM	Case=Advb|Polite=Form	9	advmod	_	MSeg=부분-에서|Translit=.bu.bun.e.seo|LTranslit=.bu.bun
3	게임과	게임	NOUN	NN+CP	Polite=Form	7	compound	_	MSeg=게임-과|Translit=.ge.im.gwa|LTranslit=.ge.im
4	우리	_	PRON	PRP	Person=1	6	compound	_	Translit=.u.ri|LTranslit=_
5	일상	_	NOUN	NN	_	6	compound	_	Translit=.il.sang|LTranslit=_
6	생활	_	NOUN	NN	_	3	conj	_	Translit=.saeng.hwal|LTranslit=_
7	사이의	사이	NOUN	NN+CM	Case=Gen|Polite=Form	8	nmod:poss	_	MSeg=사이-의|Translit=.sa.i.yi|LTranslit=.sa.i
8	유사점을	유사점	NOUN	NN+CM	Case=Acc|Polite=Form	9	obj	_	MSeg=유사점-을|Translit=.yu.sa.jeom.eul|LTranslit=.yu.sa.jeom
9	찾을	_	VERB	VV	Form=Adn	10	acl:relcl	_	Translit=.chaj.eul|LTranslit=_
10	수	_	NOUN	NNB	_	11	nsubj	_	Translit=.su|LTranslit=_
11	있습니다	_	ADJ	JJ	Mood=Ind|VerbForm=Fin	0	root	_	SpaceAfter=No|Translit=.iss.seub.ni.da|LTranslit=_
12	.	.	PUNCT	.	_	11	punct	_	Translit=.|LTranslit=_
```

#### [0.3. Code-switching (CS) and Transliterate (Tlit) task](#table-of-contents)
- Code-switching: 여러 언어가 한 문장에 등장하는 경우
  - ex 1)  `I thought मौसम different होगा बस fog है`
  - ex 2) `Hate speech를 investigate하기 위한 social media platform으로는 대표적으로 Twitter가 많이 utilize되며, 종종 facebook도 consider된다.` 
- Tlit: 음차 표기
  - ex 1) `I thought mosam different hoga bas fog hy`
  - ex 2)
<div style="text-align:center">
<img class="image" src="https://user-images.githubusercontent.com/24843996/85153201-de219900-b290-11ea-9ab3-ff2ccebc45d8.png" width="80%">
</div>
<br>

<div class='breaker'></div> 

### [1. `M-BERT` 의 cross-lingual transferability 는 vocab overlap 때문일까? → NO](#table-of-contents)
- vocab overlap: fine-tuning dataset (train) 의 word piece 와 evaluation dataset (test) 의 word piece 간의 overlap
$$overlap = | E_{train} ∩ E_{eval} | / | E_{train} ∪ E_{eval} |$$
- 검증 방식:
  - NER task 중 16개의 언어에 대한 in-house 데이터셋으로 가능한 언어쌍 (16 * 15 = 240 개) 에 대해 overlap을 구하고, trasfer score (F1) 를 report 
  - _**(결과) `M-BERT`는 vocab overlap 과 무관하게 generally 성능이 좋다. vocab overlap 이 0인 언어쌍에 대해서도 최소 40%의 F1 score 를 보인다. 반면 EN-BERT 는 vocab overlap 에 굉장히 많이 영향을 받는다.**_
<div style="text-align:center">
<img class="image" src="https://user-images.githubusercontent.com/24843996/85154700-bfbc9d00-b292-11ea-8f7b-f25ca48b965b.png" width="85%">
</div>
<figcaption class="caption">credit) http://www.dhgarrette.com/papers/pires_multilingual_bert_acl2019_slides.pdf</figcaption>
<br>

<div class='breaker'></div>

### [2. `M-BERT`의 cross-lingual transferability 는 언어의 typological 특징 때문일까? → YES](#table-of-contents)
- 근거: POS accuracy of `ur` → `hi` (91%) while `en`→ `ja` (49.4%) (둘 다 다른 script 를 사용하는 언어쌍, a.k.a vocab overlap ~= 0)
- typological features [^4]
<div style="text-align:center">
<img class="image" src="https://user-images.githubusercontent.com/24843996/85156077-8553ff80-b294-11ea-8a8d-5478762d6b43.png" width="85%">
</div>
<br>

- _**(결과) 공통된 typological feature 의 개수가 많을수록 transferabiltiy 향상**_
<div style="text-align:center">
<img class="image" src="https://user-images.githubusercontent.com/24843996/85156248-c3e9ba00-b294-11ea-8ca1-b3e74ea88860.png" width="60%">
</div>
<br>

- _**(결과) 여러 typological features 중에서 SOV order 와 AN order의 영향을 비교했을 때 전자가 더 영향이 큼**_
<div style="text-align:center">
<img class="image" src="https://user-images.githubusercontent.com/24843996/85156350-ea0f5a00-b294-11ea-862f-64ab5e3166f3.png" width="60%">
</div>
<br>

<div class='breaker'></div>

### [3. `M-BERT`의 cross-lingual transferability 는 CS 혹은 Tlit 까지 적용될 수 있을까? → CS (YES) / Tlit (NO)](#table-of-contents)
- CS 실험 목적: Generalizing to code-switching is similar to other cross-lingual transfer scenarios, but would beneﬁt to an even larger degree from a shared multilingual representation.
- Tlit 실험 목적: Generalizing to transliterated text is similar to other cross-script transfer experiments, but has the additional caveat that _`M-BERT` was not pre-trained on text that looks like the target_.
- **_(결과) `M-BERT`는 CS text 에 좋은 성능을 보임 (90.56% ⇒ 86.59%). 하지만 Tlit 은 이러한 종류의 데이터에 학습되지 않고서는 trasferability를 기대하기 어려움 (85.64% ⇒ 50.41%)_**
<div style="text-align:center">
<img class="image" src="https://user-images.githubusercontent.com/24843996/85159242-34dea100-b298-11ea-9922-3b1acc66ef7b.png" width="60%">
<figcaption class="caption">credit) https://www.youtube.com/watch?v=ZGZy_GrFkAY</figcaption>
</div>
<br>

<div class='breaker'></div>

### [4. `M-BERT`의 feature space](#table-of-contents)
- WMT16 병렬 코퍼스를 사용해서 언어쌍 간의 NN accuracy 측정
- **_(결과) 중간 layer에서 linguistic information을 공유하고 이는 언어에 관계없이 비슷하게 나타남_**
<div style="text-align:center">
<img class="image" src="https://user-images.githubusercontent.com/24843996/85160445-7ae83480-b299-11ea-8ff2-3209922e78e7.png" width="60%">
</div>
<br>

<div class='breaker'></div>

## [My Thoughts on the Results](#table-of-contents)
### [1. vocab overlap 실험에서 `EN-BERT`와의 비교는 정당한가?](#table-of-contents)
- 제 추측으로는, vocab overlap 실험에서 `M-BERT` 만을 보았을 때, 이 경향성이 overlap에 영향을 받는 것인지 아닌지 판단하기 어려웠기 때문에 상대 비교를 할만한 결과가 필요해서 EN-BERT의 실험 결과를 넣은 것 같습니다.
  - 아래 이미지에서 corr 을 보면 어느 정도 유의미해 보이는 양의 상관관계가 나올 것 같고, 그렇다면 영향을 받는다고 해석해야할 수도 있지만,
  - vocab overlap이 0임에도 40%의 성능을 보이므로 영향을 받는다고 하기도 애매한 상황이 아니었을까요
<div style="text-align:center">
<img class="image" src="https://user-images.githubusercontent.com/24843996/85162060-ac61ff80-b29b-11ea-90a2-7e497a5196a9.png" width="70%">
</div>
<br>

- 그렇지만 EN-BERT와의 비교가 정당하다고 생각하긴 어려운 것 같습니다.
  - NER 예측 task 는 sent -> model (either `M-BERT` or EN-BERT) -> last activation -> add.layer -> NER prediciton 로 진행되는데, sent 가 영어가 아닌 경우 tok 단계에서부터 `unk`으로 인식될 가능성이 높기 때문에 transfer는 고사하고 fine-tuning도 어려울 수 있습니다. 이 때문에 논문에서 EN-BERT와 XLM을 비교하는데요, Indo-european 인 (`de`, `nl`, `es`) 에 대해서만 나와있습니다. (Table 3)
  - 영어와 alphabet 이 비슷하면, unk 이 나오지 않을 가능성이 높다고 생각해서 위의 결과가 EN-BERT로 얻어진 것에 대해 크게 거부감이 들지 않았고, 오히려 CJK 에 대해서 보였어야 하는 것 아닌가 하는 의심이 들었습니다.
  - 그래서 🤗  로 간단하게 tokenize 결과를 비교해보았습니다.
  - sent 가 `es` 인 경우
    - sent: `Por su parte , el Abogado General de Victoria , Rob Hulls , indicó que no hay nadie que controle que las informaciones contenidas en CrimeNet son veraces .` 
    - `M-BERT` tok: `Por su parte , el Ab ##oga ##do General de Victoria , Rob Hull ##s , ind ##ic ##ó que no hay nadie que controle que las informa ##ciones conte ##nida ##s en Crime ##Net son vera ##ces .`
    - EN-BERT tok: `Po ##r su part ##e , el A ##bo ##gado General de Victoria , Rob Hull ##s , in ##dic ##ó que no hay na ##die que control ##e que las inform ##ac ##ione ##s con ##ten ##idas en Crime ##Net son ve ##race ##s .`
  - sent 가 `ko` 인 경우
     - sent: `언어(言語)에 대한 정의는 여러가지 시도가 있었다.`
     - `M-BERT` tok: `언 ##어 ( 言 語 ) 에 대한 정 ##의 ##는 여러 ##가지 시 ##도가 있었다 .`
     - EN-BERT tok: `[UNK] ( [UNK] [UNK] ) [UNK] [UNK] [UNK] [UNK] [UNK] [UNK] .`
  - **_위의 결과로 미루어보아, vocab overlap 이 낮으면서 성능도 낮았던 점들의 언어쌍에는 EN-BERT에서 unk이 많이 나왔던 언어가 포함되어 있지 않을까하는 생각이 들었고, 비교가 공정하지 않다는 생각이 들었습니다._**
 - 또한 tlit. 을 `M-BERT` 가 못하는 이유로, pre-train step에서 tlit. corpus 가 없었기 때문이라고 언급하였는데 EN-BERT 또한 같은 이유로 성능이 낮을 수 밖에 없었을 것이라고 생각합니다.

<div class='breaker'></div>

### [2. SOV order 가 중요한 점이었을까?](#table-of-contents)
- 논문에서는 아래 표에서 SVO -> SVO (81.55) > SVO -> SOV (66.52) 라는 점 때문에 SOV order 가 가장 중요하다고 주장합니다.
  - 하지만 반대로 SOV -> SOV (64.22) > SOV -> SVO (63.98) 의 차이가 적게 나는 점은 설명할 수 없습니다.
<div style="text-align:center">
<img class="image" src="https://user-images.githubusercontent.com/24843996/85165966-93f4e380-b2a1-11ea-9290-8451958499e3.png" width="60%">
</div>
<br>

- 제 추측으로는, 오히려 각 그룹을 구성하는 언어와 그 언어들이 Wikipedia 에서 차지하는 비율, 즉 `M-BERT` 학습에 영향을 많이 끼친 언어가 중요한 역할을 했을 수도 있을 것 같습니다.
  - SVO languages: Bulgarian, Catalan, Czech, Danish, English, Spanish, Estonian, Finnish, French, Galician, Hebrew, Croatian, Indonesian, Italian, Latvian, Norwegian (Bokmaal and Nynorsk), Polish, Portuguese (European and Brazilian), Romanian, Russian, Slovak, Slovenian, Swedish, and Chinese.
  - SOV Languages: Basque, Farsi, Hindi, Japanese, Korean, Marathi, Tamil, Telugu, Turkish, and Urdu.
    - Urdu -> Hindi 의 성능이 91% 였던 점을 고려하면 이 중 어떤 언어쌍에서 굉장히 낮은 성능을 보였을 것으로 예상됩니다. (평균이 64.22 이어야하므로) 
    - 그룹의 평균치를 보지 않았다면 다른 해석이 가능했을 수도..?
- SVO order 가 비슷하면 -> transfer 가 잘된다! 라는 주장을 하고 싶었다면 비교하려는 대상 언어쌍들간에 SVO order 빼고는 조건을 동일하게 만족시켰어야 하지 않을까하는 아쉬움이 남습니다.

<div class='breaker'></div>

### [3. Feature space](#table-of-contents)
- MLM은 보통 중간 layer 에서 semantic 한 성질이 가장 두드러지게 나타나는 것 같습니다.
<div style="text-align:center">
<img class="image" src="https://user-images.githubusercontent.com/24843996/85170841-ed144580-b2a8-11ea-9db9-099c019568f3.png" width="60%">
<figcaption class="caption">credit) The Bottom-up Evolution of Representations in the Transformer: A Study with Machine Translation and Language Modeling Objectives</figcaption>
</div>
<br>
<div style="text-align:center">
<img class="image" src="https://user-images.githubusercontent.com/24843996/85170937-1208b880-b2a9-11ea-81e8-c1102286646a.png" width="80%">
<figcaption class="caption">credit) BERTScore: Evaluating Text Generation with BERT</figcaption>
</div>
<br>
<div class='breaker'></div>

### [4. 논문의 분석 내용](#table-of-contents)
- 논문에서 분석한 결론이 지나친 일반화가 아닐까하는 생각도 듭니다.
  
- 일단 `M-BERT`가 zero-shot transferability 가 높은 이유는 어쩌면 같은 내용의 Wikipedia 문서로 학습했기 때문일 수 있습니다.
  - 만약에 한국어는 동일 내용에 대한 번역된 문서가 없는 (e.g., 네이버 블로그) 로 학습하고, 영어도 영어 나름대로의 번역문이 없는 문서로 학습된 BERT 였더라도 같은 결과가 도출되었을지는 모르겠습니다.
- `M-BERT` 의 학습 데이터의 특징에서 기인한 특징이 얼마나 되는지도 궁금합니다. 오히려 여기에서 영향을 많이 받았을 수도 있을 것 같아요.

<div class='breaker'></div>

## [References](#table-of-contents)
[^1]: [How multilingual is Multilingual BERT?](https://www.aclweb.org/anthology/P19-1493.pdf)
[^4]: [Selective Sharing for Multilingual Dependency Parsing](https://www.aclweb.org/anthology/P12-1066.pdf)
