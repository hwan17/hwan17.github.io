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
#### 4.1 위키피디아 데이터 전처리
반정형 데이터(표, 정보상자, 리스트) 제거
#### 4.2 Question Answering Datasets
1. Natural Questions(NQ)
2. TriviaQa
3. WebQuestions(WQ)
4. CuratedTREC(TREC)
5. SQuAD v1.1

positive passage 선택

TREC, WQ, TriviaQA는 질문 대답쌍만이 주어지기 때문에, 정답을 포함하는 구절 중 BM25에서 가장 높은 순위를 가진 구절을 positive passage로 사용한다.


### Experiments: Passage Retrieval

#### 5.1 Main Result
2번 표는 5가지 QA데이터 셋에서 구절점색 시스템을 비교하고 있다.(top 20, 100 accuracy)

SQuAD를 제외하고는 DPR이 대체로 BM25보다 성능이 좋았다. 성능 차이는 k가 클때보다 작을 때 차이가 많이 났다.

SQuAD데이터 셋에서 성능이 안좋은 이유

1. 답안 작성자가 구절을 읽고 답안을 작성했다.
2. 데이터가 약 500개의 위키피디아에서 모아졌기 때문에, 학습 예시의 분포가 상당히 편향되어 있다.

#### 5.2 Ablation Study on Model Training

1. Sample efficientcy 
   - 얼마나 많은 예시가 필요한가를 확인
   - NQ에서 1000개를 사용했을 때 이미 BM25의 성능 추월
   - 예시가 많아질수록 검색 정확도 향상

2. In-batch negative training
    - negative passage 선택 방법 적용(random, BM25, Gold) / top-k accuracy에서 k가 20보다 크면 그렇게 성능 차이가 크게 나타나지 않는다.
    - in-batch negative training은 새로만드는 것보다 이미 배치 내의 negative examples를 재사용하는 쉽고, 메모리 효율적인 방법이다. -> 배치사이즈가 커지면 성능이 향상된다.
    - BM25 점수는 높지만, 답을 포함하지 않는 추가적인 '어려운' negative passages 에 대한 실험. 이 추가적이인 구절들은 같은 배치의 모든 질문에 대해 negative passage로 사용되었다. 단일 BM25 negative passage를 추가하는 것은 성능을 향상 시켰지만, 두개를 추가하는 것은 그렇게 도움이 되지 못했다.

3. Impact of gold passages
   -  Our experiments on Natural Questions show that switching to distantly-supervised passages (using the highest-ranked BM25 passage that contains the answer), has only a small impact: 1 point lower top-k accuracy for retrieval. Appendix A contains more details. ("???)
   -  

4. Similarity and loss
   - L2 Norm, dot product
   - log likelihood

5. Cross-dataset generalization
   - non-iid setting에서 얼마나 성능 하락이 있는지 확인
   - 다른말로, 여전히 추가적인 fine-tunning없이 다른 데이터 셋에 적용해도 일반화 되는지 확인
   - top-20 정확도에서 fine-tunned된 베스트 모델보다 3-5포인트 로스, BM25보다는 훨씬 좋은 성능을 보였다.

#### 5.3 Qualitative Analysis
일반적으로 DPR이 BM25 보다 성능이 좋지만, 검색된 결과는 질적으로 다르다.
BM25(용어기반 매칭 방법)은 키워드나 구절에 매우 민감한 반면, DPR은 어휘적 변화나 의미적 관계를 더 잘 포착한다.

#### 5.4 Run-time Efficiency
-----------------------
-----------------

### 6 Experiments: Question Answering
#### 6.1 End-to-end QA system
검색기 외에도 질문에 대한 답을 출력하는 판독기 신경망으로 구성.
구절 선택 모델은 질문과 구절의 cross-attention을 통해 재정렬 역할을 한다. cross-attention은 큰 구절에서는 분해 불가능한 특성 때문에 실현 가능하지 않지만, 그것은 듀얼 인코더 모델보다 능력이 좋다.(?)

작은 수의 retrived candidates 에서는 잘 작동하는 것이 확인되었다.?

#### 6.2 Result
SQuAD 데이터를 제외하고는 QA 결과가 더 좋게 나왔다.
NQ, TriviaQA와 같은 큰 데이터셋은 multiple datasets를 사용하여 훈련한 모델과 단독 데이터를 사용하여 훈련한 모델이 결과가 비슷했다.
상대적으로 작은 데이터셋(WQ, TREC)은 multiple datasets를 이용하여 훈련한 결과가 확실히 좋았다.
전반적으로 DPR-based 모델은 이전의 sota모델보다 성능이 좋았다.(5 중 4)

### 7 Related Work
---------
### 8 Conclusion
잠재적으로 Dense retrieval이 전통적인 sparse retrieval을 대체할 수 있을 것이라고 설명했다.
간단한 듀얼 인코더 접근은 놀랍게도 잘 작동하였지만, 우리는 dense retriever를 성공적으로 학습시키는데 중요한 요소가 있다는 것을 보여줬다(?)
복잡한 모델 구조나 유사도 함수가 추가적인 가치를 주는 것은 아니라는것을 보여줬다.





참고 : https://www.facebook.com/111809756917564/posts/276190540479484/