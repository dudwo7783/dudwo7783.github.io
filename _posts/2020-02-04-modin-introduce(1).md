---
layout: single
title: "[Python] - MODIN(1/2)"
categories: [data, engineering]
tags: [data, engineering, python]
---

# MODIN 소개

아직 modin은 개발 중이고 document가 완성되지 않아 설명이 부족할 수 있습니다.

해당 포스팅은 https://modin.readthedocs.io/ 를 참고하였습니다.



Python으로 데이터 처리를 하다보면 데이터에 따라 프레임워크 또는 라이브러리들이 종속적이게 된다.

대용량 데이터를 운영하기 위해서는 클러스터를 구성하고 클러스터 상에서 처리하는 많은 프레임워크들이 있다. 

Apache Spark를 예로 들 수 있다. 하지만 아주 많은 경우 분석가들은 데이터들을 자신의 laptop 상에서 전처리하고 모델링하고 예측까지 하나의 파이프라인을 만드는 경우도 굉장히 많다. 

그런데 데이터가 애매하게 많다?(데이터 사이즈가 크다 작다의 기준은 참 주관적이라고 생각한다) 얘기가 달라진다. laptop에서 돌리기에는 데이터가 커서 부담된다. 그래서 Spark 클러스터를 구성한다? trade off가 등가 교환이 안된다. Cost가 굉장히 많이 소모된다. 그렇다고 Pandas로 몇 GB를 laptop에서 돌린다? 우리에게는 시간이 없다.

이럴 때 사용할 수 있는 라이브러리가 MODIN이라고 생각한다. ***Faster pandas, even on your laptop / Modin is a DataFrame for datasets from 1KB to 1TB+*** 라고 modin Document에 작성되어 있다. 당신의 laptop에서도 빠른 pandas를 사용할 수 있고, 데이터의 스케일은 적게는 1KB to 1TB+ 까지도 가능하다 란다(light-weight). 



여기서 보는 것처럼 MODIN은 쉽게 말하면 **pandas**의 변형이라고 볼 수 있다. **pandas의 dataframe**의 구조를 변형하고 병렬처리를 통해 더  빠르게 데이터를 처리할 수 있는 기술을 가졌다.(아직 pandas의 71%의 method만 커버하고 있긴하다.)



## 1. Architecture

### 1. Dataframe Partitioning

Modin은 데이터베이스와 고성능 매트릭스 시스템의 트렌드를 따라간다. Modin은 데이터 프레임을 열과 행 모두 **patitioning**하는데, 이를 통해 Modin의 유연성과 확장성을 가져간다.

![](/assets/images/modin_dataframe_architecture.jpg)



위의 그림과 같이 pandas 데이터 프레임을 열과 행으로 나눈다. 각 나누어진 데이터 프레임도 pandas 데이터 프레임의 포맷을 가진다.(곧 In-memory [Arrow](https://arrow.apache.org/docs/python/generated/pyarrow.Table.html) Table을 제공할 예정이라고 한다.)



### 2. System Arichtecture

Modin 다른 계층과 분리되어 있다. 각 구성 요소를 추상화하여 다른 시스템에 대한 영향을 줄이고자 하기 때문이다.

어떤 새로운 컴퓨팅 커널이 릴리즈 된다면 해당 커널을 컴포넌트로써 연결하여 사용하도록 설계되어 있다.

![modin-system-arichitecture](/assets/images/modin-system-arichitecture.jpg)

위 아키텍처를 보면 Execution layer에서 Ray, Dask를 사용한 병렬처리를 할 수 있도록 디자인 되었다.



#### 주요 API 

##### 1. Execution Engine/Framework

간단한 환경 설정 등 몇 줄의 코딩을 통해 간단하게 병렬 컴퓨팅을 할 수 있다.

지원 되는 병렬 프레임워크는 Ray와 Dask가 있다. Dask Document는 아직 만들어지지 않았다.

In-memory(pyArrow) 지원은 아직 experimental 단계이다.



##### 2. Internal abstractions

Partition Manager를 통해 파티셔닝 최적화를 한다.



## 2. Installation

### Stable version

```
pip install modin
```



### Install dependency sets

modin은 병렬 프레임워크인 Ray, Dask에 의존한다.

따라서 dependency sets을 설치하기 위해서

```
pip install ray, dask, distributed, pandas
```

```
pip install "modin[ray]" # If you want to use the Ray backend
pip install "modin[dask]" # If you want to use the Dask backend
```

그리고 공식적으로 Pandas 0.23.4에 의존한다.

실제로 `import ray` 하면 많은 warning들이 발생하는데 Resource Monitoring, Debuging을 위한 용도로 필요한 라이브러리를 설치하지 않아서 나오는 경고이다. 다음 포스팅에서 같이 설명하도록 하겠다.



## 3. Using Modin

이번 포스팅에서는 간단한 사용법과 지원되는 프레임워크에 대해서만 설명한다.



### Single Node

흔히 `import pandas as pd` 한다 대신에

```
import modin.pandas as pd
```

를 사용하여 modin pandas dataframe을 사용하면 거의 다른게 없다.



### Cluster Mode

Ray는 bulit-in autuscaled 클러스터를 지원한다. [autoscaler documentation](https://ray.readthedocs.io/en/latest/autoscaling.html) 을 통해 configrue를 설정해줘야 한다.



### Customize Ray environment

다음의 과정을 통해 ray의 환경 변수를 설정하여 resource 사용을 조정할 수 있다.

```
import ray
os.environ["MODIN_ENGINE"] = "ray"  # Modin will use Dask
ray.init(num_cpus=4)
```



### Exceeding memory(out-of-core)

데이터가 너무 크다. 메모리에 올릴 수가 없다. 이럴 때 out-of-core라고 하는데 메모리 오버플로우에 대해서 disk를 사용할 수 있다.(대신에 disk 사용시 성능 저하 예상, Random access에서 one sequence로 접근)



환경 변수와 파이썬 코드로 설정 가능.

```
export MODIN_OUT_OF_CORE=true # out-of-core 사용
export MODIN_MEMORY=200000000000 # Set the number of bytes to 200GB
```





## 4. Pandas on Ray

현재 Modin에서 지원하는 Pandas API는

- DataFrame
- Series
- utilities
- I/O

![modin_api_cover](/assets/images/modin_api_cover.png)



아직 위만큼 지원한다.

[https://modin.readthedocs.io/en/latest/UsingPandasonRay/index.html](https://modin.readthedocs.io/en/latest/UsingPandasonRay/index.html) 에서 지원되는 API를 확인 할 수 있다.



padnas on ray를 사용하기 위해 MODIN_ENGINE 환경 변수를 세팅하면 된다.(Python 내에서도 가능)

```
export MODIN_ENGINE=ray
```



### 1. Ray 최적화

#### Serialization of tasks and parameters

Task 및 매개변수 pre serialization을 통한 최적화

각 파티션에 직렬화하고 Map 작업 시 적용



#### Memory Management

Ray와 Arrow에 관련되어 있음. Ray는 arrow plasma를 사용한다고 하였다.

pandas는 간단한 작업에도 많은 copy를 만든다. 따라서 더이상 객체의 메모리 참조가 되지 않으면 메모리를 비운다.      (experimental)





### 2. Pandas 최적화

partitioning으로 인해 Modin Dataframe은 여러개의 인덱싱정보(행, 열)을 복제하여 사용한다.  이 문제를 해결하기 위해 [RangeIndex](https://pandas.pydata.org/pandas-docs/version/0.23.4/generated/pandas.RangeIndex.html)를 사용한다.(RangeIndex를 사용하면 메모리 절약 가능, 하지만 고정 cost 발생)





## 마치며

이번 포스팅에서는 pandas를 위한 병렬 처리 라이브러리인 modin에 대해 알아 보았다. 아직 개발 중인 모습이 많아 보이지만 조금 더 사용해보면서 자주 사용하는 pandas API를 커버하는지에 대해 확인하고 성능에 대해 테스트 해볼 예정이다.

다음 포스팅에서는 Modin 사용법과 성능 benchmark에 대해 알아보겠다.
