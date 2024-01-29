# **1. 기본 타임라인 설명**
### 1-1. **SMT의 단점:**
- 복잡도가 증가하고 여러 모델이 생성될 수 있음
- n-gram으로 인한 희소성 문제 발생
- 그래서 nmt 등장

### 1-2. **NMT란:**
- nmt는 최근 제안한 기계 번역에 대한 새롭 게 떠오르는 접근 방식
- NMT는 인코더와 디코더로 분리된 매우 작은 서브 컴포넌트 및 구성 요소로 이루어져 있음
- 인코더, 디코더는 구조로 분리되어 있음
- 인코더 신경망은 소스 문장을 읽고 고정 길이 벡터로 인코딩. 그런 다음 디코더는 인코딩된 벡터에서 번역을 출력
- conditional probability를 최대화하여 정확도 향상이 목표

### 1-3. **NMT(인코더, 디코더 방식)의 문제:**
- 고정 길이 벡터가 정보를 압축하는 데 정보 손실 위험이 크며, 문장이 길어질수록 성능이 저하됨
- 실제로 기본 인코더-디코더의 성능은 입력 문장의 길이 가 증가함에 따라 급격히 저하되는 것으로 나타남

### 1-4. **해결**
- 이 문제를 해결하기 위해 논문의 저자는 공동으로 정렬하고 변환하는 방법을 학습하는 인코더-디코더 모델의 확장을 도입
- 번역에서 단어를 생성할 때마다 원본 문장에서 가장 관련성이 높은 정보가 집중 된 위치 집합을 (소프트) 검색. 그런 다음 모델은 이러한 소스 위치 및 이전에 생성된 모든 대상 단 어와 관련된 컨텍스트 벡터를 기반으로 대상 단어를 예측


# **2. Background of NMT, RNN Encoder, Decoder**

### 2-1. NMT
- 번역을 할 때 조건부 확률이 가장 높은 y(결과)를 찾아야 함
- nmt에서는 매개변수화된 모델을 적용하여 병렬 훈련 코퍼스를 사용하여 문장 쌍의 조건부 확률을 최대화하는 문장을 찾아 생성
- 인코더-디코더 구조로 주로 이루어짐
- 입력 문장 x를 인코딩, 출력 문장 y로 디코딩
- (Cho et al., 2014a)와 (Sutskever et al., 2014)에서 RNN을 사용하여 가변 길이의 소스 문장을 고정 길이 벡터로 인코딩하고 이를 가변 길이의 대상 문장으로 디코딩


### 2-1-1. nmt의 의의
- 새로운 접근 방식이지만, 신경 기계 번역은 이미 훌륭한 결과를 보여주고 있음. Sutskever 등(2014)은 RNN(순환 신경망)과 장단기 메모리(LSTM) 유닛을 기반으로 한 신경 기계 번역이 전통적인 구문 기반 기계 번역 시스템과 유사한 성능을 달성한다고 보고
- 신경 구성 요소를 기존 번역 시스템에 추가하는 것, 예를 들어 구(phrase) 쌍을 점수화하기 위해 (Cho et al., 2014a)나 후보 번역을 다시 순위 지정하기 위해 (Sutskever et al., 2014) 같은 방식으로, 기존의 최첨단 성능 수준을 뛰어넘음


### 2-2. RNN encoder decoder


Encoder-Decoder 프레임워크에서 인코더는 입력 문장, 즉 벡터의 시퀀스 x = (x₁, ..., xₙ)를 벡터 c로 읽어들임. 가장 흔한 방법은 RNN을 사용하는 것이며, 이를 통해 다음과 같이 표현됨:

- hₜ = f(xₜ, hₜ₋₁)  (1) 인코더
- c = q({h₁, ..., hₙ})  고정 벡터

여기서 hₜ ∈ ℝⁿ은 시간 t에서의 hidden state이며, c는 hidden state의 시퀀스에서 생성된 벡터. f와 q는 어떤 비선형 함수를 나타냄. 예를 들어, Sutskever 등(2014)은 f로 LSTM을 사용하였으며, q는 q({h₁, ..., hₙ}) = hₙ와 같이 구현.



디코더는 주로 컨텍스트 벡터 c와 이전에 예측된 모든 단어 {y₁, ..., yₜ₀₋₁}을 고려하여 다음 단어 yₜ₀를 예측하기 위해 훈련됨. 다시 말하면, 디코더는 결합 확률을 순서대로 조건부로 분해하여 번역 y에 대한 확률을 정의:

```
p(y) = Πₜ₌₁ᵀ p(yₜ | {y₁, ..., yₜ₋₁}, c)   (2)
```

여기서 y는 번역을 나타내며, T는 번역 문장의 길이를 나타냄. 이것은 디코더가 각 시점에서 현재까지 예측된 단어들과 컨텍스트 벡터 c를 기반으로 다음 단어의 확률을 예측하는 방식을 나타냄.

RNN을 사용하면 각 조건부 확률은 다음과 같이 모델링:

```
p(yₜ | {y₁, ..., yₜ₋₁} , c) = g(yₜ₋₁, sₜ, c)   (3)   디코더
```

여기서 g는 비선형적이고 여러 층으로 구성될 수 있는 함수로, yₜ₋₁과 sₜ 및 c에 대한 확률을 출력함. 여기서 sₜ는 RNN의 hidden state. 다른 아키텍처, 예를 들어 RNN과 de-convolutional 신경망의 혼합이 가능하다는 것에 유의해야 함 (Kalchbrenner and Blunsom, 2013).



# **3.  LEARNING TO ALIGN AND TRANSLATE**
이 섹션에서는 신경 기계 번역을 위한 새로운 아키텍처를 제안. 이 새로운 아키텍처는 인코더로 양방향 RNN(Recurrent Neural Network)을 사용하며(3.2절), 디코딩 중에 번역을 수행하면서 소스 문장을 검색하는 디코더를 포함하고 있음(3.1절).


### 3-0. 용어
   - f: LSTM function
   - q: forward rnn
   - alpha: feedforward rnn
   - c: hidden-states(sequence of hidden-states, context vector)
콘텍스트 벡터, 맥락 벡터 최종적으로 디코더에 넘기기 전에 바이디렉션rnn이 구해준 값을 시퀀스로 이어줌
   - hj: encoder의 hidden state
   - i, j: decoder 및 encoder의 time step
   - Align: attention
   - annotation: hidden-state
   - e: 하이퍼볼릭 탄젠트를 사용하여 NN을 구할 때의 값 

### 3.1 DECODER: GENERAL DESCRIPTION
조건부 확률 정의
새로운 모델 아키텍처에서는 각 조건부 확률을 다음과 같이 정의:

p(yᵢ | y₁, ..., yᵢ₋₁, x) = g(yᵢ₋₁, sᵢ, cᵢ)   (4)   디코더에서 이렇게 계산됨

여기서 sᵢ는 시간 i에 대한 RNN의 히든 상태로, 다음과 같이 계산됨:

sᵢ = f(sᵢ₋₁, yᵢ₋₁, cᵢ)

기존의 인코더-디코더 접근 방식과는 달리 (Eq. (2) 참조), 이곳에서는 확률이 각 대상 단어 yᵢ에 대해 별도의 컨텍스트 벡터 cᵢ에 조건이 걸려 있음에 주목해야 함.

컨텍스트 벡터 cᵢ는 입력 문장을 인코더가 매핑한 주석(annotations) 시퀀스 (h₁, ..., hₜₓ)에 의존합. 각 주석 hᵢ는 입력 시퀀스의 i번째 단어 주변의 부분에 중점을 둔 정보를 포함. 다음 섹션에서는 주석이 어떻게 계산되는지에 대한 자세한 설명을 제공.

그런 다음 컨텍스트 벡터 cᵢ는 이러한 주석들 hᵢ의 가중 합으로 계산:
(벡터 (고정되지 않음))
![image](https://github.com/mandumonster/-NLPTeam/assets/113160870/82484496-66f6-47da-b5bf-45b20eade869)
  (5)

각 주석 hⱼ의 가중치 αᵢⱼ는 다음과 같이 계산:

(가중치))
![image](https://github.com/mandumonster/-NLPTeam/assets/113160870/4cd9c44e-0cd0-4aca-9e80-a2a55ec4bbeb)
   (6)

여기서 
(에너지)
![image](https://github.com/mandumonster/-NLPTeam/assets/113160870/7fb82d42-a796-4774-b7b8-7f1a6594ec9e)


는 정렬 모델로, j 위치 주변의 입력과 i 위치에서의 출력이 얼마나 잘 일치하는지를 점수로 표현. 이 점수는 RNN의 히든 상태 sᵢ₋₁ (아래 사진에서 yᵢ를 생성하기 직전에)와 입력 문장의 j번째 주석 hⱼ에 기반.


![image](https://github.com/mandumonster/-NLPTeam/assets/113160870/98db9a9d-5d3c-472c-8a2b-ba69343d8512)


정렬(어텐션 매커니즘) 모델 α를 피드포워드 신경망으로 매개변수화함. 이 신경망은 제안된 시스템의 다른 구성 요소들과 함께 공동으로 훈련됨.

전통적인 기계 번역과는 달리, 여기서는 attention이 숨겨진 변수로 간주되지 않음. 대신에 정렬 모델은 직접적으로 소프트 정렬(soft alignment)을 계산. 이는 비용 함수의 그래디언트를 역전파할 수 있게 하며, 이 그라디언트는 정렬 모델과 전체 번역 모델을 함께 학습하는 데 사용될 수 있음.

(수식이 도저히 나오지 않아서 이미지로 대체)
![image](https://github.com/mandumonster/-NLPTeam/assets/113160870/d769ecb2-d7a4-4cf9-bc5f-41cc0070cc80)



### 3.2 ENCODER: BIDIRECTIONAL RNN FOR ANNOTATING SEQUENCES

일반적인 RNN은 Eq.hₜ = f(xₜ, hₜ₋₁)  (1)에 설명된 대로 입력 시퀀스 x를 첫 번째 기호 x₁부터 마지막 기호 xₙ(Xt_x)까지 순서대로 읽음. 그러나 제안된 구조에서는 각 단어의 주석이 이전 단어뿐만 아니라 다음 단어들도 요약하길 원함. 따라서 우리는 최근에 음성 인식에서 성공적으로 사용된 양방향 RNN (BiRNN, Schuster and Paliwal, 1997)을 사용하려고 함

양방향 RNN은 정방향(forward) 및 역방향(backward) RNN으로 구성됨. 정방향 RNN →f는 입력 시퀀스를 순서대로 읽고 (x₁에서 xₙ까지), 정방향 숨겨진 상태의 시퀀스 −→h₁, · · · , −→hₙ을 계산함. 역방향 RNN ←−f는 역순으로 시퀀스를 읽고 (xₙ에서 x₁까지), 역방향 숨겨진 상태의 시퀀스 ←−h₁, · · · , ←−hₙ을 생성.

각 단어 xⱼ의 주석은 정방향 숨겨진 상태 −→hⱼ와 역방향 숨겨진 상태 ←−hⱼ를 연결하여 얻음. 즉, hⱼ = [−→hⱼ; ←−hⱼ]임. 이렇게 함으로써 주석 hⱼ는 이전 단어 및 다음 단어의 요약을 포함. RNN이 최근 입력을 더 잘 나타내는 경향이 있기 때문에 주석 hⱼ는 주변의 xⱼ 주변 단어에 중점을 둠. 이 주석 시퀀스는 나중에 컨텍스트 벡터를 계산하는 데 디코더 및 정렬 모델에서 사용됨 (Eqs. (5)–(6)).



# **4. 실험**

### 4-1. 데이터셋
target task: english-french translation 
ACL WMT ’14의 영어 프랑스어 코퍼스
- 348M개로 단어 제한
- 가장 많이 사용되는 단어 30,000개 사용

rnn encoder-decoder

- up to 30 word sentences
- up to 50 words sentence
- 1000 hidden units of encoder when forward +1000 hidden units of encoder when backward
- 1000 hidden units of decoder 

​
optimizer

- adadelta(앱실론: 10의 -6승, rho=0.95), SGD
- 미니 배치: 80문장

기존 RNN과 RNN search라는 논문에서 제안한 모델을 통해 결과 비교
![image](https://github.com/mandumonster/-NLPTeam/assets/113160870/a0cc27ed-1198-4507-b660-e1d856010c1c)


# **5. 결과**
### 5-1. 정량적 결과
![image](https://github.com/mandumonster/-NLPTeam/assets/113160870/112fa723-ec53-4526-a552-938cd888860a)
표에는 BLEU 점수로 측정된 번역 성능이 나열되어 있음.
RNNsearch 가 모든 경우 더 뛰어난 성능을 보임. 또한 RNNsearch의 성능이 기존 구절 기반 번역 시스템 (Moses)과 동등하다는 성능을 제공.

이는 Moses가 RNNsearch 및 RNNencdec를 훈련시키는 데 사용한 병렬 코퍼스에 추가로 별도의 단일 언어 말뭉치 (418M 단어)를 사용하는 반면에 이루어진 중요한 성취임.

### 5.2 질적 분석
### 5.2.1 정렬
![image](https://github.com/mandumonster/-NLPTeam/assets/113160870/27295da4-a4c6-4d22-9ee4-cc45c852713d)
- RNN search50에서 찾은 4개의 샘플 정렬
- x축: 영어
- y축: 프랑스어
- 각 픽셀은 가중치(w)를 의미(hidden state weight \[ \alpha_{ij} = \frac{ \exp(e_{ij}) }{ \sum_{k=1}^{T_x} \exp(e_{ik}) } \]
을 시각화) (0: 검정, y: 흰색)

- a에서 모델은 [European Economic Area]를 [zone economique européenne]로 올바르게 번역
- RNNsearch는 [zone]을 [Area]에 정확하게 정렬하고, 두 단어 ([European] 및 [Economic])를 건너뛰고, 그런 다음 한 번에 한 단어씩 되돌아가 전체 구를 완성

### 긴 문장 번역
문장 앞 부분은 정보의 누락 없이 어느정도 좋은 번역 결과를 제공했지만, 문장의 뒤로 갈수록 정보의 누락이 있음







