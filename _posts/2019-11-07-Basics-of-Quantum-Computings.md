---
title: "수학으로 이해하는 양자컴퓨터의 기초"
layout: post
date: 2019-11-07 00:07
image: /assets/images/quantum_computer_main.jpg
categories: [ Quantum Computer, Basic, Qbit, Superposition, Entanglement, Quantum Supremacy, 양자컴퓨터, 기초, 양자 중첩, 양자 얽힘, 양자 우월성 ]
featured: true
hidden: true
---

최근에 있었던 구글의 양자 우월성 (Quantum Superemacy) 달성은 전세계적으로 큰 화제였다.
물 들어올 때 노 저으라고 하지 않았던가, 이 때가 아니면 또 언제 양자컴퓨터에 대해 공부할까 싶어서 텍스트와 동영상을 넘나들며 관련 지식을 습득해보았다.

아마 나와 같이 관련 기사나 여러 블로그 글, 유튜브 등을 찾아본 사람들이라면 어렵지 않게 아래의 정보는 얻었을 것이다.
 - n개의 qbit은 bit와 달리 $$2^n$$의 state를 표현할 수 있다. 
 - superposition이란 동시에 0과 1의 상태를 띠는 성질로, 병렬연산이 가능해져서 고전컴퓨터에 비해 계산 속도의 이점이 생긴다. 

텍스트만 보면 "아 그렇구나." 싶은 내용들이다. 
이해가 된 것일까 싶었지만 스스로에게 세 질문을 던졌을 때 답하지 못하는 것을 보며 제대로 이해하지 못했음을 인지했다. <br>

&nbsp;&nbsp;&nbsp;&nbsp;**Q1.** n개의 bit로도 $$2^n$$을 표현할 수 있는거 아닌가? 3개의 bit가 000, 001, 010, 011, 100, 101, 110, 111 이렇게 8개의 상태를 표현할 수 있으니까. <br>
&nbsp;&nbsp;&nbsp;&nbsp;**Q2.** 양자 세계는 불확정성 원리에 지배받는다고 하는데, 대체 양자컴퓨터로 어떻게 연산하고 있는 것이며, 이 성질이 어떻게 계산 비용을 감소시킬 수 있는걸까? <br>
&nbsp;&nbsp;&nbsp;&nbsp;**Q3.** Entanglement는 qbit들이 어떻게 된 상태인거지? 

이 질문들에 제대로 답하기 위해서는 수학이 필요하다는 생각이 들었다.
4차원 이상의 공간을 제대로 시각화하지 못하듯이 양자 세계를 자연어로 표현한다는 것 자체가 말이 되지 않는 것 같았기 때문이다.
그래서 수학으로 설명된 자료를 찾으려고 부던히 애를 썼고, 끝내 "내 수준에 맞는" (= 이 글을 읽을 모두가 다 이해할 수 있는) 수학으로 설명된 자료를 찾았다. 

<iframe width="560" height="315" src="https://www.youtube.com/embed/F_Riqjdh2oM" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

[발표자료](https://speakerdeck.com/ahelwer/quantum-computing-for-computer-scientists)

이 영상을 보는 것을 추천하지만 무려 한시간이 넘는지라 글로도 정리를 해보았다. 아래에 기술된 내용은 내 방식대로 위의 영상과 자료를 재구성한 것이다. 

사실 이 자료를 다 보더라도 양자컴퓨터에 대해 많은 것을 알았다고 보긴 어렵다.
python을 처음 접한 사람이 `print('Hello World!')`를 성공했다고 해서 python을 잘 알았다고 하기 어려운 것처럼.
그리고 딥러닝에 관심있는 사람이 tutorial을 따라해보며 CNN을 돌려봤다고 해서 딥러닝을 잘 알았다고 하기 어려운 것처럼.

그렇지만 양자 세계에 한 번은 `'Hello World!'`를 날려봐야 하지 않을까?

<div class="breaker"></div>

### Table of Contents

[1. Introduction](#1-introduction)<br>
[2. Basics](#2-basics)<br>
&nbsp;&nbsp;&nbsp;&nbsp;[2.1 Qubit / Qbit](#21-qubit--qbit)<br>
&nbsp;&nbsp;&nbsp;&nbsp;[2.2 Superposition](#22-superposition)<br>
&nbsp;&nbsp;&nbsp;&nbsp;[2.3 Tensor product](#23-tensor-product)<br>
&nbsp;&nbsp;&nbsp;&nbsp;[2.4 1-bit operations](#24-1-bit-operations)<br>
&nbsp;&nbsp;&nbsp;&nbsp;[2.5 CNOT (one of the 2-bit operations)](#25-cnot-one-of-the-2-bit-operations)<br>
[3. The Deutsch-Jozsa problem](#3-the-deutsch-jozsa-problem)<br>
&nbsp;&nbsp;&nbsp;&nbsp;[3.1 Classical computer](#31-classical-computer)<br>
&nbsp;&nbsp;&nbsp;&nbsp;[3.2 Quantum computer](#32-quantum-computer)<br>
[4. Entanglement](#4-entanglement)<br>
[5. Conclusion](#5-conclusion)<br>
[References](#references)


<div class="breaker"></div>


## 1. Introduction

The Deutsch-Jozsa problem 이라는 아주 간단한 문제를 통해 양자컴퓨터가 고전컴퓨터에 비해 어떻게 연산 속도에서 이점을 보이는지 알아보려고 한다.
이 과정을 이해하기 위해 양자컴퓨터가 연산하는 방법에 대해 소개할 것이며 matrix 연산과 기초적인 논리회로에 대한 내용을 짚고 넘어갈 것이다. 

추가로, entanglement에 대한 간단한 설명이 있다.

<div class="breaker"></div>

## 2. Basics

### 2.1 Qubit / Qbit

Qubit 혹은 Qbit은 양자컴퓨터 계산의 기본적인 단위이다. 조금이라도 양자컴퓨터에 대해 알아본 사람들이라면 qbit은 지겹도록 보고 들었을 것이다. 

Qbit은 언제나 다음의 조건을 만족시킨다. <br>
> A qbit, represented by 
> $$ \begin{pmatrix}
 \alpha \\
 \beta
 \end{pmatrix} $$
 where $$\alpha$$ and $$\beta$$ are complex numbers must be constrained by the equation 
> $$ ||\alpha||^2 + ||\beta||^2 = 1 $$

따라서 아래의 예시들은 qbit에 해당된다.

$$ \begin{pmatrix}
\frac{1}{\sqrt{2}} \\
\frac{1}{\sqrt{2}}
 \end{pmatrix} $$
$$ \begin{pmatrix}
\frac{1}{2} \\
\frac{\sqrt{3}}{2}
 \end{pmatrix} $$
$$ \begin{pmatrix}
 1 \\
 0
 \end{pmatrix} $$
$$ \begin{pmatrix}
 0 \\
 -1
 \end{pmatrix} $$

그리고 이 모든 벡터들의 basis가 되는 
$$ \begin{pmatrix}
 1 \\
 0
 \end{pmatrix} $$
 과 
 $$ \begin{pmatrix}
 0 \\
 1
 \end{pmatrix} $$
은 각각 $$ \mid 0\rangle $$ 과 $$ \mid 1\rangle $$ 이라는 특별한 기호로 정의한다.


### 2.2 Superposition

양자컴퓨터의 qbit을 설명할 때 빠지지 않는 성질이다. 
"동시에 0과 1을 가진다."는 문장으로 자주 설명되지만 이보다는 슈뢰딩거의 고양이 느낌이 물씬 나는 "When we measure a qbit, it collapses to an actual value of 0 or 1." 이라는 문장이 더 좋은 설명인 것 같다.

위에서 qbit이라고 언급했던 벡터 하나를 예시로 들어보자.

$$ \begin{pmatrix}
\frac{1}{\sqrt{2}} \\
\frac{1}{\sqrt{2}}
 \end{pmatrix} $$

이 qbit은 $$0$$ 혹은 $$1$$ 로 collapse될 확률이 (
$$ \frac{1}{2} = || \frac{1}{\sqrt{2}} || ^2 $$
) 이다.

감사하게도 [IBM은 자사의 양자컴퓨터를 사용할 수 있는 API](https://quantum-computing.ibm.com/)를 만들어 놓았다. 
여기서 이 qbit을 만들고 1024번 관측해보면 0과 1이 50%씩 나오는 것을 확인할 수 있다. 

<div style="text-align:center">
<img class="image" src="{{ site.baseurl }}/assets/images/qubit_1_2_example.png" height="200px" style="margin-right:20px">
<img class="image" src="{{ site.baseurl }}/assets/images/qubit_1_2_result.png" height="200px">
<figcaption class="caption">(왼) H gate를 통해 |0> 은 예시로 든 qbit으로 바뀐다. (이 내용은 밑에서 다룬다.) (오) 1024번 관측한 결과다. 50% 확률로 0과 1을 나타낸다.</figcaption>
</div>
<br>

 $$ \begin{pmatrix}
\frac{1}{2} \\
\frac{\sqrt{3}}{2}
 \end{pmatrix} $$
은 $$ 0 $$ 으로 collapse 될 확률이 $$\frac{1}{4}$$, $$ 1 $$ 로 collapse 될 확률이 $$\frac{3}{4}$$인 qbit이다.


$$ |0\rangle $$ 
은 0으로만 collapse 한다.

<div style="text-align:center">
<img class="image" src="{{ site.baseurl }}/assets/images/qubit_0_example.png" height="200px" style="margin-right:20px">
<img class="image" src="{{ site.baseurl }}/assets/images/qubit_0_result.png" height="200px">
<figcaption class="caption">(왼) |0> (오) 1024번 관측한 결과로, 100% 0으로 관측된다. </figcaption>
</div>
<br>

### 2.3 Tensor product

여러 개의 qbit을 나타내기 위해 Tensor product 개념이 필요하다.
수학적으로 엄밀한 표현은 아니지만, n개의 qbit 연산을 표현하기 위해서는 아래의 표기 방식을 따르는 것이 좋다.

$$ \binom{x_0}{x_1} \otimes \binom{y_0}{y_1} 
= \begin{pmatrix} x_0  \binom{y_0}{y_1} \\ 
{x_1  \binom{y_0}{y_1}} \end{pmatrix} 
= \begin{pmatrix} x_0 y_0 \\ x_0 y_1 \\ x_1 y_0 \\ x_1 y_1 \end{pmatrix}
$$

이를 응용하면 2개, 3개의 qbit도 벡터처럼 표현할 수 있다.

$$
|01\rangle = \binom{1}{0} \otimes \binom{0}{1} = \begin{pmatrix}
  0 \\ 1 \\ 0 \\ 0
\end{pmatrix}
\hspace{10pt}
|100\rangle = \binom{0}{1} \otimes \binom{1}{0} \otimes \binom{1}{0} = \begin{pmatrix}
  0\\ 0\\0\\0 \\ 1 \\ 0 \\ 0 \\ 0
\end{pmatrix}
$$

이와 같이 tensor product의 결과로 표현된 벡터는 product state라고 한다.
여기서 우리는 n개 qbit의 product state 크기가 $$2^n$$ 이라는 것을 알 수 있다.
만약 
$$
\binom{\frac{1}{\sqrt{2}}}{\frac{1}{\sqrt{2}}} \otimes \binom{\frac{1}{\sqrt{2}}}{\frac{1}{\sqrt{2}}} = 
\begin{pmatrix}
 \frac{1}{2} \\ \frac{1}{2} \\ \frac{1}{2} \\ \frac{1}{2}
\end{pmatrix} 
$$
의 multiple qbits 이 있다면 $$ \mid 00\rangle $$, $$ \mid 01\rangle $$, $$ \mid 10\rangle $$, $$ \mid 11\rangle $$ 으로 collapse될 확률이 모두 $$ \frac{1}{4} $$이므로 동시에 4개의 state를 표현할 수 있게 된다. 
즉, qbit이 bit와는 다르게 $$2^n$$ 개의 state를 표현할 수 있다고 한 것은 *동시에* 가질 수 있는 최대 state 관점에서 비교한 것이다.
bit는 절대로 동시에 2개 이상의 state를 가질 수 없으므로 한 번에 계산할 수 있는 정보는 1개 뿐이다.

또한 product state는, 뒤의 entanglement와 구분되는 중요한 성질로, **독립적인 state들로 factorize가 가능**하다는 점이 있다.

Multiple qbits의 product state 또한 single qbit과 같은 성질을 만족시킨다.

$$
\binom{a}{b} \otimes \binom{c}{d} = 
\begin{pmatrix}
  ac \\ ad \\ bc \\ bd
\end{pmatrix} 
$$

$$ \text{where, } ||ac||^2 + ||ad||^2 + ||bc||^2 + ||bd||^2 = 1 $$

<br>

### 2.4 1-bit operations

1-bit에서 가능한 연산은 Identity, Negation, Constant-0, Constant-1의 총 4가지가 있다. 

<div style="text-align:center">
<img class="image" src="{{ site.baseurl }}/assets/images/1bit_operations.png" width="40%">
<figcaption class="caption">1-bit 연산의 4 종류</figcaption>
</div>
<br>

각각의 연산은 matrix로 표현할 수 있다.

$$ \text{Identity} = \begin{pmatrix}
1 & 0 \\ 
0 & 1
\end{pmatrix} $$, 
$$ \text{Negation} = \begin{pmatrix}
0 & 1 \\ 
1 & 0
\end{pmatrix} $$, 
$$ \text{Contant-0} = \begin{pmatrix}
1 & 1 \\ 
0 & 0
\end{pmatrix} $$, 
$$ \text{Contant-1} = \begin{pmatrix}
0 & 0 \\ 
1 & 1
\end{pmatrix} $$


<div style="text-align:center">
<img class="image" src="{{ site.baseurl }}/assets/images/1bit_matrix.png" width="70%">
<figcaption class="caption">1-bit 연산의 matrix 표현</figcaption>
</div>
<br>


### 2.5 CNOT (one of the 2-bit operations)

CNOT 연산은 control bit와 target bit로 구성된 2-bit가 있을 때 control bit가 0이면 target bit를 바꾸지 않고, control bit가 1일 때 target bit를 바꾸는 연산이다.

<div style="text-align:center">
<img class="image" src="{{ site.baseurl }}/assets/images/CNOT_operation.png" width="20%">
<figcaption class="caption">CNOT</figcaption>
</div>
<br>

마찬가지로 이 연산도 matrix로 표현할 수 있다.

$$ C = \begin{pmatrix}
1 & 0 & 0 & 0 \\ 
0 & 1 & 0 & 0 \\ 
0 & 0 & 0 & 1 \\ 
0 & 0 & 1 & 0 \\ 
\end{pmatrix} $$


<div style="text-align:center">
<img class="image" src="{{ site.baseurl }}/assets/images/CNOT_examples.png" width="60%">
<figcaption class="caption">2-qbits에 적용한 CNOT 예시</figcaption>
</div>
<br>

<div class="breaker"></div>

[2.4](#24-1-bit-operations)와 [2.5](#25-cnot-one-of-the-2-bit-operations)에서 opertaion들을 matrix화 한 것에 주목하자. 
확률이 지배하는 양자 세계에서 deterministic한 연산을 하기 위해서는 matrix를 관측하지 않은 qbit에 곱하는 것이 유일한 방법이기 때문이다.
아래의 예시에서 우리가 확신할 수 있는 정보는 qbit이 0과 1로 관측될 확률이 반대가 되었다는 것이다.

$$ \begin{pmatrix}
0 & 1 \\ 
1 & 0
\end{pmatrix} \begin{pmatrix}
  \frac{1}{2} \\ \frac{\sqrt{3}}{2} 
\end{pmatrix} = \begin{pmatrix}
  \frac{\sqrt{3}}{2} \\ \frac{1}{2}
\end{pmatrix}$$


항상 0 혹은 1로 관측되는 $$\mid0\rangle$$이나 $$\mid1\rangle$$을 쓰면 matrix 연산을 고집하지 않아도 되지만 이런 qbit만 사용할거라면 고전컴퓨터를 쓰면 그만이다.
굳이 0K 가까이 되는 험악한 조건을 유지해가며 계산할 필요가 없다.

그래서 matrix 연산은 양자 컴퓨팅에서 굉장히 중요하다. 여기에는 한가지 추가조건이 있는데, 반드시 연산에 사용되는 matrix는 **reversible**해야한다는 것이다.
따라서 앞서 본 1-bit 연산 중 Constant-1과 Constant-0를 계산하기 위해서는 단순 matrix를 곱하는 것 외의 다른 방법이 필요하다. 

<div class="breaker"></div>

## 3. The Deutsch-Jozsa problem [^1]

이 문제는 양자컴퓨터가 고전컴퓨터에 비해 계산적인 이점을 가지는 아주 간단한 (~~*동시에 쓸데없는*~~) 문제다.

> 1-bit를 입력받아서 1-bit를 내뱉는 어떤 함수가 있다고 하자. 
> 만약 이 함수가 Constant(Contant-0, Constant-1)인지, 아니면 Variable(Identity, Negation)인지 알기 위해서는 최소 몇 번의 query를 날려야 할까?


### 3.1 Classical computer

고전컴퓨터에서는 0과 1을 입력해야하므로 총 두 번의 연산이 필요하다.

### 3.2 Quantum computer

예상했듯이 정답은 한 번이다. 왜인지 알기위해서는 추가적인 개념이 필요하다.

<div class="breaker"></div>

#### - Hadamard gate

앞서 언급된 적 있는 H gate이다.

$$ H = \begin{pmatrix}
  \frac{1}{\sqrt{2}} & \frac{1}{\sqrt{2}} \\
  \frac{1}{\sqrt{2}} & \frac{-1}{\sqrt{2}}
\end{pmatrix}
$$

Hadamard gate는 0- 혹은 1-qbit을 받아서 0과 1을 같은 확률로 가지는 qbit으로 바꿔준다.

$$
H\mid0\rangle = \begin{pmatrix}
  \frac{1}{\sqrt{2}} & \frac{1}{\sqrt{2}} \\
  \frac{1}{\sqrt{2}} & \frac{-1}{\sqrt{2}}
\end{pmatrix} \begin{pmatrix}
  1 \\ 0
\end{pmatrix} = \begin{pmatrix}
  \frac{1}{\sqrt{2}} \\ \frac{1}{\sqrt{2}}
\end{pmatrix}
$$

$$
H\mid1\rangle = \begin{pmatrix}
  \frac{1}{\sqrt{2}} & \frac{1}{\sqrt{2}} \\
  \frac{1}{\sqrt{2}} & \frac{-1}{\sqrt{2}}
\end{pmatrix} \begin{pmatrix}
  0 \\ 1
\end{pmatrix} = \begin{pmatrix}
  \frac{1}{\sqrt{2}} \\ \frac{-1}{\sqrt{2}}
\end{pmatrix}
$$

Hadamard gate는 또 다른 중요한 성질이 있다. 0과 1을 같은 확률로 가지는 qbit을 다시 0- 과 1-qbit으로 돌려보낸다는 것이다.

$$
\begin{pmatrix}
  \frac{1}{\sqrt{2}} & \frac{1}{\sqrt{2}} \\
  \frac{1}{\sqrt{2}} & \frac{-1}{\sqrt{2}}
\end{pmatrix} \begin{pmatrix}
  \frac{1}{\sqrt{2}} \\ \frac{1}{\sqrt{2}}
\end{pmatrix} = \begin{pmatrix}
  1 \\ 0
\end{pmatrix}
$$

$$
\begin{pmatrix}
  \frac{1}{\sqrt{2}} & \frac{1}{\sqrt{2}} \\
  \frac{1}{\sqrt{2}} & \frac{-1}{\sqrt{2}}
\end{pmatrix} \begin{pmatrix}
  \frac{1}{\sqrt{2}} \\ \frac{-1}{\sqrt{2}}
\end{pmatrix} = \begin{pmatrix}
  0 \\ 1
\end{pmatrix}
$$


#### - X gate

X gate는 qbit의 위 아래를 바꿔준다.

$$ X = 
\begin{pmatrix}
   0 & 1 \\
   1 & 0
\end{pmatrix}
$$

$$
\begin{pmatrix}
   0 & 1 \\
   1 & 0
\end{pmatrix} 
\begin{pmatrix}
  0 \\ 1
\end{pmatrix}
= \begin{pmatrix}
  1 \\ 0
\end{pmatrix}
$$

$$
\begin{pmatrix}
   0 & 1 \\
   1 & 0
\end{pmatrix} 
\begin{pmatrix}
  \frac{-1}{\sqrt{2}} \\ \frac{1}{\sqrt{2}}
\end{pmatrix}
= \begin{pmatrix}
  \frac{1}{\sqrt{2}} \\ \frac{-1}{\sqrt{2}}
\end{pmatrix}
$$


H gate와 X gate 연산을 이해하기 쉽게 표현하면 아래의 그림이 된다. 붉은 색이 X gate, 노란색이 H gate 연산의 방향이다. 

<div style="text-align:center">
<img class="image" src="{{ site.baseurl }}/assets/images/gates-visualization.png" width="80%">
<figcaption class="caption"></figcaption>
</div>
<br>

$$
\begin{pmatrix}
  1 \\ 0
\end{pmatrix}
$$
에 X - H - X - H - X gate를 씌운 결과는 그림으로 보면 더 쉽게 이해된다. 
$$
\begin{pmatrix}
  1 \\ 0
\end{pmatrix}
$$
에서 출발해 각 gate가 연산되는 방향으로 화살표를 움직이면 마지막에 도달하는 곳이 결과값이 된다. 

<div style="text-align:center">
<img class="image" src="{{ site.baseurl }}/assets/images/quantum_logic_operation_example.png" width="80%">
</div>
<br>


#### - non-reversible matrix 

앞서 양자컴퓨터는 non-reversible한 matrix 를 곱하는 연산은 불가능하다고 했다.
1-bit 연산 중에서 Constant-0과 Constant-1은 non-reversible하다. 그래서 양자 컴퓨팅에서는 2개의 qbit을 사용한다.

<div style="text-align:center">
<img class="image" src="{{ site.baseurl }}/assets/images/quantum_non_reversible.png" width="60%">
</div>
<br>

이 그림의 input과 output notation을 보면 "응?" 이라는 생각이 절로 들 것이다. 영상에서도 사람들이 대체 왜 "Output"이 Input 쪽에 가 있는지에 대해 끊임없이 묻는다.
아쉽게도 강연자는 속시원하게 답변을 해주지 않아서 그런가보다 하고 넘어갔는데 다시 보니 이해가 된 부분이 있어 글로 설명해보려고 한다.

`Input'`과 `Output'`이 실제 1-bit 연산의 input과 output을 나타낸다. 
그리고 `Input`과 `Output`은 `Input'`과 `Output'` 각각의 input들이다. 
`Input'`과 `Output'`이 BB 이후에 있기 때문에 BB 이전에 `Input'`과 `Output'`이 1-bit 연산의 input과 output이 되도록 넣어주는, 양자컴퓨터 연산 방식 때문에 필요한 input들이다. 

이 약속에 따라서 양자컴퓨터가 1-bit 연산을 어떻게 수행하는지 아래의 예시를 통해 좀 더 이해해보자. 

<div style="text-align:center">
<img class="image" src="{{ site.baseurl }}/assets/images/quantum_constant_0.png" height="200px" width="300px" style="margin-right:20px; margin-bottom:10px">
<img class="image" src="{{ site.baseurl }}/assets/images/quantum_constant_1.png" height="200px" width="300px" style="margin-bottom:10px">
<img class="image" src="{{ site.baseurl }}/assets/images/quantum_identity.png" height="200px" width="300px" style="margin-right:20px; margin-bottom:10px">
<img class="image" src="{{ site.baseurl }}/assets/images/quantum_negation.png" height="200px" width="300px" style="margin-bottom:10px">
<figcaption class="caption">왼쪽 위 부터 차례대로 Constant-0, Constant-1, Identity, Negation. Identity와 Negation에 있는 이상한 operation은 CNOT이다.</figcaption>
</div>
<br>

Constant-0은 `Input'`이 $$\mid 0 \rangle$$일 때와 $$\mid 1 \rangle$$일 때 모두 `Output'`이 $$\mid 0 \rangle$$이어야 한다. 
어떤 gate도 없는 왼쪽 위 그림의 회로에서 `Input`에 $$\mid 0 \rangle$$ 혹은 $$\mid 1 \rangle$$을 대입해보면 `Input'`과 `Output'`이 Constant-0의 관계를 가지는 것을 확인할 수 있다.

Indentity는 `Input'`이 $$\mid 0 \rangle$$일 때는 `Output'`이 $$\mid 0 \rangle$$이고 `Input'`이 $$\mid 1 \rangle$$일 때는 `Output'`이 $$\mid 1 \rangle$$인 함수다. 
왼쪽 아래 그림의 회로는 CNOT gate를 표현하고 있다. 색이 채워진 원이 control bit 쪽을 나타내고 그렇지 않은 쪽 원은 target bit를 나타낸다.
`Input`이 $$\mid 0 \rangle$$ 이면 control bit가 0이므로 target bit도 그대로 유지한다. 그래서 `Input'`도 $$\mid 0 \rangle$$, `Output'`도 $$\mid 0 \rangle$$이 된다.
`Input`이 $$\mid 1 \rangle$$ 이면 control bit가 1이므로 target bit가 바뀐다. 그래서 `Input'`도 $$\mid 1 \rangle$$, `Output'`도 $$\mid 1 \rangle$$이 된다.

<div class="breaker"></div>

그럼 다시 The Deutsch-Jozsa problem로 돌아가서, 양자컴퓨터에서는 어떻게 한 번에 구할 수 있을까? 정답은 아래의 그림이 설명해준다.

<div style="text-align:center">
<img class="image" src="{{ site.baseurl }}/assets/images/quantum_one_query.png" width="60%">
<figcaption class="caption">양자컴퓨터가 한 번에 문제를 푸는 법</figcaption>
</div>
<br>

이 연산대로라면 BB가 Constant(Contant-0, Constant-1)이었을 경우, 측정시에 $$\mid11\rangle$$로 나타나고 Variable(Identity, Negation)이었을 경우에는 $$\mid01\rangle$$로 나타난다.

BB의 경우의 수를 따져가며 이해해보자.

#### preprocessing (BB 입력 직전까지의 연산)
<br>
<div style="text-align:center">
<img class="image" src="{{ site.baseurl }}/assets/images/quantum_preprocessing.png" width="80%">
<figcaption class="caption"></figcaption>
</div>
<br>

BB에 들어가기 전 input과 output qbit ($$ \mid 0 \rangle$$) 모두 X와 H gate를 거쳐서
$$ \begin{pmatrix}
  \frac{1}{\sqrt{2}} \\
  \frac{-1}{\sqrt{2}}
\end{pmatrix}
$$
가 된다.

#### case 1) BB가 Constant-0 이었을 경우
<br>
Constant-0은 input과 output에 어떤 gate도 씌우지 않는다. 
따라서 BB가 Constant-0이었을 때 Input과 Output은 H gate만 통과한 이후 관측된다.

<div style="text-align:center">
<img class="image" src="{{ site.baseurl }}/assets/images/const_0.png" width="60%">
<figcaption class="caption">Constant-0</figcaption>
</div>

<div style="text-align:center">
<img class="image" src="{{ site.baseurl }}/assets/images/quantum_bb_const_0.png" width="80%">
<figcaption class="caption">BB가 Constant-0인 경우 Input'과 Output'</figcaption>
</div>
<br>


#### case 2) BB가 Contstant-1 이었을 경우
<br>
Constant-1은 output에만 X gate를 적용한다. 
따라서 BB가 Constant-1이었을 때는 Output에 X gate가 추가되고, 이후 Input과 Output 모두에 H gate가 적용된다. 
<div style="text-align:center">
<img class="image" src="{{ site.baseurl }}/assets/images/const_1.png" width="60%">
<figcaption class="caption">Constant-1</figcaption>
</div>

<div style="text-align:center">
<img class="image" src="{{ site.baseurl }}/assets/images/quantum_bb_const_1.png" width="80%">
<figcaption class="caption">BB가 Constant-1인 경우 Input'과 Output'</figcaption>
</div>
<br>



#### case 3) BB가 Identity 이었을 경우
<br>
Identity는 CNOT gate를 통해 연산된다.
앞에서 CNOT 연산은 
$$ \begin{pmatrix}
1 & 0 & 0 & 0 \\ 
0 & 1 & 0 & 0 \\ 
0 & 0 & 0 & 1 \\ 
0 & 0 & 1 & 0 \\ 
\end{pmatrix} $$
을 곱하는 것과 같다고 설명했다.
Preprocessing을 거친 Input과 Output은 
$$ \begin{pmatrix}
  \frac{1}{\sqrt{2}} \\
  \frac{-1}{\sqrt{2}}
\end{pmatrix}
$$
이므로 CNOT연산은 아래와 같이 표현할 수 있다.

$$
C \begin{pmatrix}
  \begin{pmatrix}
    \frac{1}{\sqrt{2}} \\
    \frac{-1}{\sqrt{2}}
  \end{pmatrix} \otimes
    \begin{pmatrix}
    \frac{1}{\sqrt{2}} \\
    \frac{-1}{\sqrt{2}}
  \end{pmatrix}
\end{pmatrix} = C \begin{pmatrix}
  \frac{1}{2} \\
  \frac{-1}{2} \\
  \frac{-1}{2} \\
  \frac{1}{2}
\end{pmatrix} = \frac{1}{2} \begin{pmatrix}
  1 & 0 & 0 & 0 \\
  0 & 1 & 0 & 0 \\
  0 & 0 & 0 & 1 \\
  0 & 0 & 1 & 0 \\
\end{pmatrix} \begin{pmatrix}
  1 \\
  -1 \\
  -1 \\
  1
\end{pmatrix} = \frac{1}{2} \begin{pmatrix}
  1 \\
  -1 \\
  -1 \\
  1
\end{pmatrix} = \begin{pmatrix}
    \frac{1}{\sqrt{2}} \\
    \frac{1}{\sqrt{2}}
  \end{pmatrix} \otimes
    \begin{pmatrix}
    \frac{1}{\sqrt{2}} \\
    \frac{-1}{\sqrt{2}}
  \end{pmatrix}
$$

즉 Input은 
$$\begin{pmatrix}
    \frac{1}{\sqrt{2}} \\
    \frac{-1}{\sqrt{2}}
\end{pmatrix}$$
에서 
$$\begin{pmatrix}
    \frac{1}{\sqrt{2}} \\
    \frac{1}{\sqrt{2}}
\end{pmatrix}$$ 
로 바뀌고 Output은 그대로 
$$\begin{pmatrix}
    \frac{1}{\sqrt{2}} \\
    \frac{-1}{\sqrt{2}}
\end{pmatrix}$$
이 된다.
이 상태에서 H gate가 각각 적용되어 최종 결과는 $$ \mid 01 \rangle$$이 된다.

<div style="text-align:center">
<img class="image" src="{{ site.baseurl }}/assets/images/identity.png" width="60%">
<figcaption class="caption">Identity</figcaption>
</div>

<div style="text-align:center">
<img class="image" src="{{ site.baseurl }}/assets/images/quantum_bb_identity.png" width="80%">
<figcaption class="caption">BB가 Identity인 경우 Input'과 Output'</figcaption>
</div>
<br>

#### case 4) BB가 Negation 이었을 경우
<br> 
Negation은 Indentity의 결과 중 Output에만 X gate가 추가되는 연산이다. 
따라서 아래 그림처럼 연산이 이루어지고 Identity와 마찬가지로 최종 결과는 $$ \mid 01 \rangle$$이 된다.

<div style="text-align:center">
<img class="image" src="{{ site.baseurl }}/assets/images/negation.png" width="60%">
<figcaption class="caption">Negation</figcaption>
</div> 

<div style="text-align:center">
<img class="image" src="{{ site.baseurl }}/assets/images/quantum_bb_negation.png" width="80%">
<figcaption class="caption">BB가 Negation인 경우 Input'과 Output'</figcaption>
</div>
<br>


정리하면, 양자컴퓨터에서는 특정 설계 상황에서 고정된 BB input에 대한 BB output을 "한 번"만 관측하면 BB가 Constant인지 Variable인지 확인할 수 있다!

<div class="breaker"></div>


## 4. Entanglement

Entanglement는 지금까지의 흐름에서는 동떨어진 이야기지만 양자컴퓨터에서 항상 소개되는 내용이기 때문에 추가하였다. 

앞서 qbit과 product state의 성질을 수학적으로 나타낸 것처럼 entanglement도 수학적인 성질로 표현할 수 있다. 
$$
\begin{pmatrix}
  \frac{1}{\sqrt{2}} \\
  0 \\
  0 \\
  \frac{1}{\sqrt{2}}
\end{pmatrix}
$$
는 entangle된 qbit인데, 그 모양새가 product state 닮아있다. 하지만 product state와는 중요한 성질에서 차이를 보인다.
위에서 설명했듯이 product state는 개별적인 qbit으로 factorize된다. 하지만 entanlged qbit은 개별적인 qbit으로 factorize 되지 않는다. 
(If the product state of two qbits cannot be factored, they are said to be **entanlged**.)

$$
\begin{pmatrix}
  \frac{1}{\sqrt{2}} \\
  0 \\
  0 \\
  \frac{1}{\sqrt{2}}
\end{pmatrix} = \begin{pmatrix}
  a \\
  b
\end{pmatrix} \otimes \begin{pmatrix}
  c \\
  d
\end{pmatrix}
$$
를 만족하는 $$a$$, $$b$$, $$c$$, $$d$$는 존재하지 않는다. 즉 개별적인 qbit으로 factorize되지 못하므로 entanlge되어 있는 qbit인 것이다.

Entanlged qbit은 CNOT과 H gate를 통해 쉽게 생성할 수 있다.

<div style="text-align:center">
<img class="image" src="{{ site.baseurl }}/assets/images/entanlge.png" width="60%">
<figcaption class="caption">Entangled qbit</figcaption>
</div>

$$
CH_1
\begin{pmatrix}
  \begin{pmatrix}
    1 \\
    0
  \end{pmatrix} \otimes \begin{pmatrix}
    1 \\
    0
  \end{pmatrix}
\end{pmatrix} = C \begin{pmatrix}
  \begin{pmatrix}
    \frac{1}{\sqrt{2}} \\
    \frac{1}{\sqrt{2}} 
  \end{pmatrix} \otimes \begin{pmatrix}
    1 \\ 0
  \end{pmatrix}
\end{pmatrix} = 
\begin{pmatrix}
  1 & 0 & 0 & 0 \\ 
0 & 1 & 0 & 0 \\ 
0 & 0 & 0 & 1 \\ 
0 & 0 & 1 & 0 \\ 
\end{pmatrix} \begin{pmatrix}
  \frac{1}{\sqrt{2}} \\
  0 \\
  \frac{1}{\sqrt{2}} \\
  0 
\end{pmatrix} = \begin{pmatrix}
  \frac{1}{\sqrt{2}} \\
  0 \\
  0 \\
  \frac{1}{\sqrt{2}}
\end{pmatrix}
$$

만약 이후에 이런 게이트의 조합을 본다면 곧바로 'entanlge 시키려고 했군!' 이라고 생각하면 된다 :)


<div class="breaker"></div>


## 5. Conclusion

개인적으로 이 영상을 본 이후, 게비스콘을 먹은 것 같이 속이 뚫리는 기분이 들었다. 아직 matrix로 표현되는 qbit이 물리적으로 어떤 모습인지, gate들이 물리적으로 어떻게 qbit에 적용되는지는 모르지만
(이건 실제 양자컴퓨터를 눈으로 보면 이해가 되지 않을까) 이 정도라도 양자컴퓨터와 고전컴퓨터의 연산과정에서의 차이를 구체적으로 알 수 있었기 때문에 만족할 수 있었다.

양자컴퓨터의 연산 과정을 이해하고나니 양자 우월성은 그냥 달성되는 것은 아니었으며, 잘 설계된 gate가 뒷받침되었을 때 가능한 것임을 깨닫게 되기도 했다.

이 정도면 양자 세계에 ``Hello World!'`를 했다고 볼 수 있지 않을까? 

<div class="breaker"></div>

## References

[^1]: https://en.wikipedia.org/wiki/Deutsch%E2%80%93Jozsa_algorithm#Problem_statement