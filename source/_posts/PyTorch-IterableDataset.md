---
title: "PyTorch의 IterableDataset을 사용해서 데이터 불러오기"
layout: post
date: 2021-02-21 23:22
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

PyTorch 1.2 이상부터 `torch.utils.data` 에서는 크게 map-style dataset (`torch.utils.data.Dataset`) 과 iterable dataset (`torch.utils.data.IterableDataset`) 의 두 종류의 데이터 클래스를 지원하고 있다. 데이터 사이즈가 클 때는 `IterableDataset` 을 사용하는 것이 좋은데, `Dataset` 과는 딜리 아직 개발되어야 할 기능이 더 필요한 클래스라서 사용할 때에 유의할 점이 있어 정리해보게 되었다.


<!--more-->

## Map-style Dataset

1.2 이하 버전에서 사용되던 map-style dataset은 memory에 모든 데이터를 업로드할 수 있을 때 사용하는 가장 일반적인 dataset type 이다. custom dataset class를 생성하고자 할 때 `torch.utils.data.Dataset` 을 상속받아 `__len__` , `__getitem__` 을 구현하면 된다.

```python
from torch.utils.data import Dataset

class MyMapDataset(Dataset):

    def __init__(self, data):
        self.data = data

    def __len__(self):
        return len(self.data)

    def __getitem__(self, index):
        return self.data['text'][index]
```

## Iterable Dataset

하지만 학습 데이터가 메모리에 다 올라가지 않는 경우가 발생할 수 있다. 이 문제를 해결할 수 있는 다양한 방법 중에 하나로, `torch.utils.data.IterableDataset` 을 사용하는 방법이 있다. Map-style Dataset과 비슷하게 `torch.utils.data.IterableDataset` 을 상속받아서 custom dataset class를 생성하고, `__iter__` 를 선언하면 된다.

```python
from torch.utils.data import IterableDataset

class MyIterableDataset(IterableDataset):

    def __init__(self, data_path):
        self.data_path = data_path

    def __iter__(self):
        iter_csv = pd.read_csv(self.data_path, sep='\t', iterator=True, chunksize=1)
        for line in iter_csv:
            line = line['text'].item()
            yield line
```

`Dataset`이 batch data를 생성할 때 `map_dataset[index]`를 사용한다면, `IterableDataset`은  `next(iterable_dataset)` 을 사용한다. 이 때문에 `DataLoader`를 통해 `IterableDataset`을 불러와서 사용하게 되면 `sampler` 옵션의 사용이 어렵다. 그래서 random suffling 을 하고 싶다면 미리 데이터셋을 shuffling 한 이후에 불러오는 것이 좋다.

## Going Parallel

[PyTorch 공식문서](https://pytorch.org/docs/stable/data.html#torch.utils.data.IterableDataset)에 따르면 `IterableDataset`을 `num_workers > 0`의 조건에서 사용할 때 특별히 다음을 유념할 것을 제안하고 있다.

> When `num_workers > 0`, each worker process will have a different copy of the dataset object, so it is often desired to configure each copy independently to avoid having duplicate data returned from the workers. `get_worker_info()`, when called in a worker process, returns information about the worker. It can be used in either the dataset’s `__iter__()` method or the `DataLoader` ‘s `worker_init_fn` option to modify each copy’s behavior.

위의 문장을 이해하려면 `num_workers` 에 대한 이해와, `num_workers > 0` 일 때  `IterDataset` 에서 어떤 현상이 일어나는지 알아야한다.

<img src="/assets/images/pytorch-iterabledataset-numworkers.png?style=centerme" alt="num_workers == 2 인 경우 발생하는 모습이다. 위의 두 라인은 subprocess이며, 맨 아래 라인은 main process이다. 파란색 박스는 single datapoint를 로딩하는 것을 의미하며 붉은색 박스는 로딩된 데이터를 모델에 전달하는 프로세스를 의미한다. (image credit: https://medium.com/speechmatics/how-to-build-a-streaming-dataloader-with-pytorch-a66dd891d9dd)" width=90%>

`num_workers`는 데이터셋을 불러올 때 사용할 subprocess의 개수이다. `num_workers == 0` 은 main process에서 데이터를 불러오고 모델에 pass하는 작업을 모두 수행한다는 뜻이며, `num_workers == 2`는 subprocess 2개에서 데이터를 불러오고 main process에서는 subprocess에서 불러온 데이터를 model에 pass하는 역할만 담당한다. 따라서 `num_workers > 0` 일 때 data loading에서의 병목이 줄어들며 gpu utilization 을 100% 가까이 끌어올릴 수 있다. 

그럼, `num_workers > 0` 일 때 어떤 현상이 발생하는지 살펴보자.

### Map-Style Dataset

```python
from torch.utils.data import DataLoader, Dataset, IterableDataset
import time

class MyMapDataset(Dataset):
    
    def __init__(self, data):
        self.data = data
        
    def __len__(self):
        return len(self.data)
    
    def __getitem__(self, index):
        worker = torch.utils.data.get_worker_info()
        worker_id = worker.id if worker is not None else -1
        
        start = time.time()
        time.sleep(0.1)
        end = time.time()
        
        return self.data[index], worker_id, start, end

data = range(16)
map_dataset = MyMapDataset(data)
```

- `num_workers == 0` 인 경우

```python
loader = DataLoader(map_dataset, batch_size=4, num_workers=0)
for d in loader:
    batch, worker_ids, starts, ends = d
    print(batch, worker_ids)

-----
tensor([0, 1, 2, 3]) tensor([-1, -1, -1, -1])
tensor([4, 5, 6, 7]) tensor([-1, -1, -1, -1])
tensor([ 8,  9, 10, 11]) tensor([-1, -1, -1, -1])
tensor([12, 13, 14, 15]) tensor([-1, -1, -1, -1])
```

- `num_workers == 2` 인 경우

```python
loader = DataLoader(map_dataset, batch_size=4, num_workers=2)
for d in loader:
    batch, worker_ids, starts, ends = d
    print(batch, worker_ids)

-----
tensor([0, 1, 2, 3]) tensor([0, 0, 0, 0])
tensor([4, 5, 6, 7]) tensor([1, 1, 1, 1])
tensor([ 8,  9, 10, 11]) tensor([0, 0, 0, 0])
tensor([12, 13, 14, 15]) tensor([1, 1, 1, 1])
```

의도한대로, subprocess 별로 서로 다른 데이터를 불러오는 것을 알 수 있다.

### Iterable Dataset

```python
from torch.utils.data import DataLoader, Dataset, IterableDataset
import time

class MyIterableDataset(IterableDataset):
    
    def __init__(self, data):
        self.data = data
    
    def __iter__(self):
        for x in self.data:
            worker = torch.utils.data.get_worker_info()
            worker_id = worker.id if worker is not None else -1
        
            start = time.time()
            time.sleep(0.1)
            end = time.time()
        
            yield x, worker_id, start, end

data = range(16)
iterable_dataset = MyIterableDataset(data)
```

- `num_workers == 0`

```python
loader = DataLoader(iterable_dataset, batch_size=4, num_workers=0)
for d in loader:
    batch, worker_ids, starts, ends = d
    print(batch, worker_ids)

-----
tensor([0, 1, 2, 3]) tensor([-1, -1, -1, -1])
tensor([4, 5, 6, 7]) tensor([-1, -1, -1, -1])
tensor([ 8,  9, 10, 11]) tensor([-1, -1, -1, -1])
tensor([12, 13, 14, 15]) tensor([-1, -1, -1, -1])
```

- `num_workers == 2`

```python
loader = DataLoader(iterable_dataset, batch_size=4, num_workers=2)
for d in loader:
    batch, worker_ids, starts, ends = d
    print(batch, worker_ids)

----
tensor([0, 1, 2, 3]) tensor([0, 0, 0, 0])
tensor([0, 1, 2, 3]) tensor([1, 1, 1, 1])
tensor([4, 5, 6, 7]) tensor([0, 0, 0, 0])
tensor([4, 5, 6, 7]) tensor([1, 1, 1, 1])
tensor([ 8,  9, 10, 11]) tensor([0, 0, 0, 0])
tensor([ 8,  9, 10, 11]) tensor([1, 1, 1, 1])
tensor([12, 13, 14, 15]) tensor([0, 0, 0, 0])
tensor([12, 13, 14, 15]) tensor([1, 1, 1, 1])
```

⚠️ worker 0과 worker 1에서 같은 데이터를 로딩하고 있다. 이 점이 공식문서에서 지적하고 있는 내용이다. 각 워커별로 같은 `__iter__()`를  사용하기 때문에 같은 데이터를 로딩하게 된다. **이를 방지하기 위해서는  `worker_init_fn` 에서 직접 워커 별 데이터를 재분배 시켜줘야 한다.**

```python
def worker_init_fn(_):
    worker_info = torch.utils.data.get_worker_info()
    
    dataset = worker_info.dataset
    worker_id = worker_info.id
    split_size = len(dataset.data) // worker_info.num_workers
    
    dataset.data = dataset.data[worker_id * split_size: (worker_id + 1) * split_size]
```

```python
loader = DataLoader(iterable_dataset, batch_size=4, num_workers=2, worker_init_fn=worker_init_fn)
for d in loader:
    batch, worker_ids, starts, ends = d
    print(batch, worker_ids)

-----
tensor([0, 1, 2, 3]) tensor([0, 0, 0, 0])
tensor([ 8,  9, 10, 11]) tensor([1, 1, 1, 1])
tensor([4, 5, 6, 7]) tensor([0, 0, 0, 0])
tensor([12, 13, 14, 15]) tensor([1, 1, 1, 1])
```

`worker_init_fn` 을 통해 분배시켜준 결과 worker 0과 worker 1 에서 다른 데이터를 순차적으로 불러오는 것을 알 수 있다 🙂

## Conclusions

- `IterableDataset` 은 데이터가 메모리에 올라가지 않을만큼 클 때 사용하면 좋다.
- `Dataset`과 다르게 `__iter__()`를 선언해서 데이터를 부른다.
    - 하지만 이 특징 때문에 `Sampler` 와 함께 사용할 수 없다.
    - 또한 `num_workers > 0` 인 세팅에서는 각 워커에서 다른 데이터를 불러올 수 있도록 `worker_init_fn`을 선언해야 한다.

## References

- [How to Build a Streaming DataLoader with PyTorch | by David MacLeod | Speechmatics | Medium](https://medium.com/speechmatics/how-to-build-a-streaming-dataloader-with-pytorch-a66dd891d9dd)
- [https://inmoonlight.github.io/notebooks/html/2021-02-21-PyTorch-Dataset.html](https://inmoonlight.github.io/notebooks/html/2021-02-21-PyTorch-Dataset.html)
- [torch.utils.data — PyTorch 1.7.1 documentation](https://pytorch.org/docs/stable/data.html#torch.utils.data.IterableDataset)
