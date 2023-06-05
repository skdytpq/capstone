## Keypaper 

- ContrastVAE : 트랜스포머 구조의 VAE!
- <p align = "center">
  <img width = "500" src = "https://github.com/skdytpq/capstone/exp/contrast1?raw=True">
  <br>
  그림 
</p>
인코더 디코더 구조에서 Transformer를 사용할 경우 Latent Space 의 표현이 제대로 되지 않음.

### Posterior Collapse

 $D_{KL}[q_{\phi}(z\vert x ) \vert \vert p(z)]$: posterior term

해당 Loss term 이 transformer 구조를 VAE 에 사용할 시 0으로 수렴하는 현상이 발생한다. 

- $p(z)$ 에 대한 정규분포 조건이 코드상으로 표현되지 않음
- Hidden Layer 에서 무거운 모델을 사용했기 때문에 해당 정보만으로도 예측이 가능해짐 (과거 데이터로 부터 generation 가능)

### Data augmentation

위 문제를 해결하고 잠재 벡터에 대한 표현을 풍부하게 학습하기 위해 두 개의 VAE 구조에서 Contrast Learning 을 진행.

해당 과정에서 한쪽 VAE 에 Data augmentation 진행 : 과거 참조를 과하게 하는 현상을 방지하기 위해

또한 모델 FFN 안에서 Variational dropout을 통해 히든 레이어에 대해서도 augmentation 진행

- Slime4rec : 1D kernel Module 

- <p align = "center">
  <img width = "500" src = "https://github.com/skdytpq/capstone/exp/contrast2?raw=True">
  <br>
  그림 
</p>
인코더 구조에서 임베딩 레이어를 거친 후 Dynamic, Static Module 을 각 통과하여 시퀀스 데이터의 Local 정보를 학습

Dynamic Slide 의 경우 filter size 가 유동적, Static slide 의 경우 정적으로 학습 진행. 

푸리에 변환을 거친 이산화 Feature 에 각 Kernel을 통해 1D CNN 과 비슷한 학습 후 Inverse 푸리에를 통해 복호화.

이후 FFN 을 통과하여 확률 분포 학습.

Contrastlearning 을 수행하기 위해 각 모듈에 다른 dropout 을 적용함

## Idea

- Transformer 가 Session-based 에서 잘 안 쓰이는 이유 : 데이터의 session 길이가 길지 않기 때문에 Transformer 의 global represntation 이 불리함.

  - 따라서 Sequential 모델을 사용할 경우 GRU , BiLSTM 모델 기반 Seq2Seq 구조가 주로 사용됨

  따라서 Transformer의 local representation 학습 과정을 도와주는 모듈을 추가하여 Transformer 모듈에서는 전체 정보만을 학습하게 함

  ContrastVAE 의 구조에서 한 쪽은 Transformer, 다른 한 쪽은 Slime4Rec 구조로 설계

  두 모듈의 contrastlearning 을 통해 Latent Space 는 Transformer에서 전체 구조에 대해 학습하며 Slime4rec 모듈에서 Local 정보를 학습

- Augmentation

  - Data augmentation 의 경우 설계한 각 두 모듈에 대해 다르게 진행하여 Ablation Study를 진행하여 성능을 비교함
  - Dropout 구성의 변화 : 아직까지 구체화 된 아이디어 없음
  - 다른 논문들에서 사용한 augmentation 방법 적용 예정

- Sampling 

  - VAE 의 poseterior collapse 문제를 해결하기 위해 sampling 방법에서 아이디어 보충 예정

### 진행상황

- Sequential dataset인 Beauty dataset으로 ContrastVAE 실험 세팅 완료
- Idea1 에서 제시한 모듈을 붙이는 부분 실험 세팅 중
  - 모듈 두 개를 붙이는 것까지 완료를 했으나 성능이 제대로 나오지 않음
  - Slime4rec의 임베딩 레이어 구성을 ContrastVAE 의 임베딩으로 통일하였기 때문에 해당 임베딩 과정 변경 예정
  - Slime4rec 논문에서 제시한 Dropout Contrastlearning 구현 예정
- 데이터 셋에서 augmentation Pipeline 구축 예정
  - 다양한 조건을 통한 Ablation Study 진행 예정
- Z_sampling 의 디리클레 분포 사용
  - 좋은 성능을 
