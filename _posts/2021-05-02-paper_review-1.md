---
title: paper_Review
excerpt: dense passage retrieval 논문

categories:
  - Paper
tags:
  - [Paper]

toc: true
toc_sticky: true

date: 2021-05-02
last_modified_at: 2021-05-02
---

# Dense Passage Retrieval for Open-Domain Question Answering

### Abstract
odqa(open domain question answering)은 후보 문맥을 선택하기 위해 효율적인 경로 검색에 의존한다.
(TF-IDF 또는 BM25와 같은, 전통적인 희소 벡터 공간 모델들이 사실상의 방법이다.) 

여기서는 dense 표현을 단독으로 사용함으로써 검색이 실용적으로 구현되는 것을 보여준다.
(dense representations 임베딩들은 적은 질문과 후보로부터 간단한 듀얼 인코더 프레임워크로 학습되어 지는)

넓은 범위의 open-domain QA 데이터에서 평가할 때, 우리의 dense retriever는 LuceneBM25보다 월등한 성능을 보였다.
top 20 passage retrieval accuracya 측면에서 9%-19% 더 나았고, 여러 odqa 벤치 마크에서 sota를 달성하여 종단종 QA시스템 설립에 도움을 주었다.

### Introduction

Open-domain question answering(QA) : 많은 문서의 집합에 대한 질문에 답하는 태스크
이전의 QA시스템은 복잡하고 여러 구성물로 이루어져 있다.
reading comprehension모델의 발달로 훨씬 간단한 2단계 프레임워크를 제안한다.
1. 먼저 구문 retriever는 작은 부분집합(질문에 대한 답을 포함하고 있는)을 선택한다.
2. reader는 검색된 문맥을 완전히 평가하고 올바른 정답을 식별한다.

odqa에서 읽는 시간을 감소시킨 것은 매우 좋은 전략(종종 큰 성능 감소가 관찰되었다.)이지만, 검색기능 향상의 필요성을 나타내었다.

odqa에서 검색은 보통 TF-IDF와 BM25를 사용하여 실행한다.
이것들은 역색인을 사용하여 효과적으로 키워드를 매치하고, 희소행렬(질문과 문맥의 높은 차원의 표현)으로 간주된다.
반대로 dense(잠재 의미 인코딩)는 희소 표현에 상호보완적인 설계이다.
예로, 완전히 다른 토큰으로 구성된 동의어나 구문은 서로 가까운 벡터로 매핑될 수 있다.

"반지의 제왕에서 나쁜 사람이 누구인가?"
라는 질문을 고려해보자.
문맥에서 답은 다음과 같이 할 수 있다.
"Sala Baker가 반지의 제왕 3부작에서 악당 사우론의 주역을 맡은 것으로 가장 잘 알려져 있습니다."

용어 기반 시스템은 위와 같은 검색을 하는데 어려움을 겪는다. 반면, dense 검색 시스템은 "나쁜 사람"과 "악당"을 더 잘 매치하는 것이 가능하고 올바른 문장을 찾아낸다.
dense 인코딩은 또한 임베딩을 함수를 조정함으로써 학습이 가능하고, 이것은 작업별 표현에 추가적인 유연성을 제공한다.

### 3. DPR(dense passages retriever)
#### 3.1 overview
다른 유사도 함수와 비슷한 성능을 보여서 간단한 cosine-similarity를 사용

Encoder
question, passage 인코더는 두개의 독립적인 bert모델의 cls토큰 출력을 사용한다 d=768

Inference
FAISS(오픈 소스 라이브러리로, dense 벡터의 유사도 검색 및 군집화)를 사용

#### 3.2 train

손실함수를 positive sample 을 잘 맞추도록, negative samples를 잘 못맞추도록 설정(? 확인 필요)

positive and negative samples
positive sample은 자명하지만, negative sample은 어떻게 선정해야할지
3가지 방법 제시
1. 랜덤
2. BM25 : BM25를 통해 반환된 top passage 중 답을 포함하지 않지만 많은 질문 토큰은 일치하는 passage
3. Gold : training set에서 다른 질문과도 possitive passage pair를 이루는 passage

best model은 negative sampling에 Gold와 BM25적용

In-batch negatives

크기 B인 배치에서 1개의 positive passage와 B-1개의 negative passages를 얻을 수 있다.


### 4. Experimental setup
