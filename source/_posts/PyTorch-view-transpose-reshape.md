---
title: "PyTorch의 view, transpose, reshape 함수의 차이점 이해하기"
layout: post
date: 2021-03-03 04:22
tags:
- pytorch
categories: 
- Tech
- Engineering
toc: true
widgets:
   - type: toc
     position: left
   - type: categories
     position: left
   - type: recent_posts
     position: left
---

최근에 pytorch로 간단한 모듈을 재구현하다가 loss와 dev score가 원래 구현된 결과와 달라서 의아해하던 찰나, tensor 차원을 변경하는 과정에서 의도하지 않은 방향으로 구현된 것을 확인하게 되었다. 그리고 그 이유는 `transpose` 와 `view` 의 기능을 헷갈려했기 때문이었다. 두 함수의 차이는 `contiguous` 를 이해해야 알 수 있는 내용이었고, 혹시 이 개념이 헷갈리는 사람들을 위해 간단한 예시를 바탕으로 정리해보았다.

<!--more-->

`contiguous` 란 matrix 의 눈에 보이는 (advertised) 순차적인 shape information 과 실제로 matrix 의 각 데이터가 저장된 위치가 같은지의 여부이다. 아래의 예시에서 `t` 는 `contiguous` 하다. 왜냐하면 `t[0][0][0]` → `t[0][0][1]` → `t[0][1][0]` ... 의 데이터 포인터 위치 (`0` → `1` → `2` ... )  또한 연속적이기 때문이다. 아직 이해가 되지 않아도 괜찮다. 예시를 좀 더 보자!

```python
t = torch.tensor([[[0, 1], [2, 3], [4, 5]], \
                 [[6, 7], [8, 9], [10, 11]], \
                 [[12, 13], [14, 15], [16, 17]], \
                 [[18, 19], [20, 21], [22, 23]]])  # (4, 3, 2)
```

{% codeblock print(t) lang:Shell >folded %}
tensor([[[ 0,  1],
         [ 2,  3],
         [ 4,  5]],

        [[ 6,  7],
         [ 8,  9],
         [10, 11]],

        [[12, 13],
         [14, 15],
         [16, 17]],

        [[18, 19],
         [20, 21],
         [22, 23]]])
{% endcodeblock %}

## view


`view` 를 통해 `t` 라는 tensor의 shape를 변경시켜보았다.

```python
tv = t.view(4, 2, 3)
```

{% codeblock print(tv) lang:Shell >folded %}
tensor([[[ 0,  1,  2],
         [ 3,  4,  5]],

        [[ 6,  7,  8],
         [ 9, 10, 11]],

        [[12, 13, 14],
         [15, 16, 17]],

        [[18, 19, 20],
         [21, 22, 23]]])
{% endcodeblock %}

shape이 `(4, 2, 3)` 으로 잘 바뀐 것을 확인할 수 있다. 그리고 `tv[0][0][0]` → `tv[0][0][1]` → `tv[0][0][2]` ... 의 데이터 포인터 위치 (`0` → `1` → `2` ... ) 또한 연속적이기 때문에 `tv` 는 `contiguous`  하다. 

```python
tv.is_contiguous()
---
True
```

데이터의 tensor index 순서대로 tensor를 flatten 시켜주는 함수를 통해 `t` 와 `tv` 를 비교하면 동일하게 나오는 것을 확인할 수 있다. 

```python
t.flatten() == tv.flatten()
---
tensor([True, True, True, True, True, True, True, True, True, True, True, True,
        True, True, True, True, True, True, True, True, True, True, True, True])
```

또한 `t` 와 `tv` 의 데이터는 pointer 값이 동일하여 한 쪽의 tensor data 를 수정하면 다른 쪽의 값도 동시에 변경된다. 

```python
t.storage().data_ptr() == tv.storage().data_ptr()  # data pointer 값이 일치함
---
True
```

```python
# Modifying view tensor changes base tensor as well.
t[0][0][0] = 99
tv[0][0][0]
---
tensor(99)
```

## transpose


`transpose` 를 통해 `t` 라는 텐서의 shape을 변경시켜보았다. shape은 `tv`와 동일하나, 구성이 조금 다른 것을 확인할 수 있다. 참고로, 보통 `(batch_size, hidden_dim, input_dim)` 을 `(batch_size, input_dim, hidden_dim)` 으로 바꿔주는 작업을 할 때에 `transpose` 를 사용한다.

```python
tt = t.transpose(2, 1)  # (4, 2, 3)
```

{% codeblock print(tt) lang:Shell >folded %}
tensor([[[ 0,  2,  4],
         [ 1,  3,  5]],

        [[ 6,  8, 10],
         [ 7,  9, 11]],

        [[12, 14, 16],
         [13, 15, 17]],

        [[18, 20, 22],
         [19, 21, 23]]])
{% endcodeblock %}

앞선 예시에서 `t` 의 데이터 포인터는 `0` → `1` → `2` ... 순서대로 저장된 것을 알 수 있었다. `t`와 `tv` 모두 데이터 포인터의 물리적 순서와 shape 상에서의 데이터 순서가 같았기 때문에  `contiguous` 했다. 하지만 `tt` 의 경우  `0` → `1` → `2` ... ≠ `tt[0][0][0]` → `tt[0][0][1]` → `tt[0][0][2]` ... 이기 때문에 `contiguous` 하지 않다.

```python
tt.is_contiguous()
---
False
```

`tt` 를 flatten 시키면 물리적 순서 (`0` → `1` → `2` ... ) 와 shape 상의 순서가 다른 것을 확인할 수 있다.

```python
tt.flatten()
---
tensor([ 0,  2,  4,  1,  3,  5,  6,  8, 10,  7,  9, 11, 12, 14, 16, 13, 15, 17,
        18, 20, 22, 19, 21, 23])
```

## contiguous


그렇다면 강제로 물리적 위치를 연속적으로 만들어버리면 어떻게 될까? 겉보기에는 `tt` 와 별 차이가 없어보인다.

```python
tt.contiguous() == tt
---
tensor([[[True, True, True],
         [True, True, True]],

        [[True, True, True],
         [True, True, True]],

        [[True, True, True],
         [True, True, True]],

        [[True, True, True],
         [True, True, True]]])
```

하지만 각 데이터 포인터를 비교하면 `tt.contiguous()` 는 `0` → `2` → `4` ... 이고 `tt` 는 `0` → `1` → `2` 이기 때문에 아래의 값이 False가 나오는 것을 예상해볼 수 있다.

```python
tt.contiguous().storage().data_ptr() == tt.storage().data_ptr()
---
False
```

## reshape

`contiguous` 개념을 이해했다면,  `reshape` 과 `view` 함수의 차이도 알 수 있다. 쉽게 얘기하면 `reshape() == contiguous().view()` 와 같다.

`view` 는 `contiguous` 하지 않은 `tensor`  에 대해서는 적용할 수 없다.

```python
tt.view(4, 3, 2)  # tt.shape() == (4, 2, 3)
---------------------------------------------------------------------------
RuntimeError                              Traceback (most recent call last)
<ipython-input-89-785954c0ff12> in <module>
----> 1 tt.view(4, 3, 2)  # tt.shape() == (4, 2, 3)

RuntimeError: view size is not compatible with input tensor's size and stride (at least one dimension spans across two contiguous subspaces). Use .reshape(...) instead.
```

```python
tt.contiguous().view(4, 3, 2)
---
tensor([[[ 0,  2],
         [ 4,  1],
         [ 3,  5]],

        [[ 6,  8],
         [10,  7],
         [ 9, 11]],

        [[12, 14],
         [16, 13],
         [15, 17]],

        [[18, 20],
         [22, 19],
         [21, 23]]])
```

하지만 `reshape` 은 강제로 tensor를 `contiguous` 하게 만들면서 shape을 변경하기 때문에 가능하다.

```python
tt.reshape(4, 3, 2)
---
tensor([[[ 0,  2],
         [ 4,  1],
         [ 3,  5]],

        [[ 6,  8],
         [10,  7],
         [ 9, 11]],

        [[12, 14],
         [16, 13],
         [15, 17]],

        [[18, 20],
         [22, 19],
         [21, 23]]])
```

```python
tt.reshape(4, 3, 2).is_contiguous()
---
True
```

## Summary


- `view` : tensor 에 저장된 데이터의 물리적 위치 순서와 index 순서가 일치할 때 (`contiguous`) shape을 재구성한다. 이 때문에 항상 `contiguous` 하다는 성질이 보유된다.
- `transpose` :  tensor 에 저장된 데이터의 물리적 위치 순서와 **상관없이** 수학적 의미의 transpose를 수행한다. 즉, 물리적 위치와 transpose가 수행된 tensor 의 index 순서는 같다는 보장이 없으므로 항상 `contiguous` 하지 않다.
- `reshape` : tensor 에 저장된 데이터의 물리적 위치 순서와 index 순서가 일치하지 않아도 shape을 재구성한 이후에 강제로 일치시킨다. 이 때문에 항상 `contiguous` 하다는 성질이 보유된다.


## References
- [https://inmoonlight.github.io/notebooks/html/2021-03-03-PyTorch-view-transpose-reshape.html](https://inmoonlight.github.io/notebooks/html/2021-03-03-PyTorch-view-transpose-reshape.html)
- [https://discuss.pytorch.org/t/contigious-vs-non-contigious-tensor/30107/2](https://discuss.pytorch.org/t/contigious-vs-non-contigious-tensor/30107/2)
