
## Title: Masked Autoencoders Are Scalable Vision Learners

Author: Kaiming He, Xinlei Chen, Saining Xie, Yanghao Li, Piotr Dollár, Ross Girshick

### 1. Prerequisite

- Transformer-based 모델은 주어진 데이터에서 패턴을 학습하는데 'effective'한 모델이다.
    - 데이터에 effective하다는 말은 데이터가 크면 클 수록 성능이 올라간다는 것이다.
    - 이는 데이터에 efficient한 CNN-based 모델과는 상반되는 얘기다.
    - 상기 특징을 갖는 이유는 두 모델간의 'Inductive Bias'에 의해서 나타난다.
    - Inductive Bias에 대한 자세한 내용은 생략하고, 간단히 말하면 주어진 데이터로부터 패턴을 추출하는 방식을 스스로/임의로 학습했느냐의 차이이다.
    - 따라서, Transformer-based 모델은 높은 성능을 위해서 Large-Scale 데이터에 대한 학습은 필수적이다.

- Large-Scale 데이터셋은 제작하는 것도 힘들고 경제적이지 않다.
    - Computer Vision에서 데이터셋 존재/제작 여부는 실질적인 연구 진행중 자주 부딪히는 문제점이다.
    - Transformer-based 모델이 큰 성공을 거두고, 다양한 분야에 application으로 뻗어나가는 동안 위에서 말한 `Large-Scale 데이터셋 부족`문제점이 더욱 도드라졌다.
        - 이를 해결하기 위해서는 Manual Labelling을 통해서 직접 제작하거나, 다른 방법이 필요하다.
    - 결론적으로 Large-Scale 데이터셋이 많으면 많을수록 Transformer-based 모델의 ‘Data Effectiveness’ 문제를 해결하는 동시에, 그 성능은 꾸준히 증가할 것이다.

### 2. Motivation

- Transoformer 모델의 근본적인 문제를 해결해보자 !
    - Kaiming He는 이를 해결하기 위한 접근법으로 `Self-Supervised Learning` 을 활용한다.
    - Self-Supervised Learning은 Unlabeled 데이터셋에 기반하여 좋은 특징을 추출할 수 있도록, Encoder를 사전에 학습시켜두는 방법론이다.
    - 즉, Transformer-based 모델의 성능을 향상시키기 위해서 Self-Supervised Learning을 이용해 Encoder를 사전에 잘 학습시켜두겠다는 것이다.
        - 해당 방법론은 Large-Scale 데이터셋 문제를 간접적으로 해결하는 것이라고 해석할 수 있다.

- 성공적인 NLP 모델 BERT에서는 Masking Pre-Training 방법론을 활용하고있다.
    - Transformer-based 모델을 위해서 pre-training하는 방법론은 NLP 분야에서는 활발히 이루어지는 트릭이였다.
    - 이를 Vision에 그대로 적용해보고자 했다고 논문에서 언급하고있다.
        - 본 연구 이전에도 몇 번의 시도는 있었으나 잘 안되었다고한다.
        - 기존 연구가 실패한 이유에 대해서 두개의 컨셉으로 정리한다.
        - `Information Dense`: NLP에서 사용하는 데이터인 sentence는 의미론적이고 정보가 밀집해있으나, Vision 데이터인 image는 spatial하게 불필요한 정보가 많고 semantic보다는 local하게 정보를 담고있다
        - `The role of decoder`: decoder의 역할이 다르다고 한다. NLP의 경우 단어 하나하나 rich information을 담고있으나, 이미지에서 pixel 하나하나는 trivial한 정보를 담고있다고 판단할 수 있기 때문이다.
            - 따라서, decoder 디자인이 중요하다고한다.

### 3. Method

- 논문에서 제안하는 방법론은 단순하 Reconstruction task로 간단하다.
    - Input을 Masking 처리하여 Transformer encoder가 Masking 되어있는 부분의 Pixel 정보를 채워넣게 한다.
    - 모델의 디자인은 다음과 같다. 
    
    <p align="center">
      <img src="https://user-images.githubusercontent.com/40862925/189523979-641d3990-1ecc-411d-8481-cd8d3c16cbd6.png" style="padding: 0;margin:0;">
    </p>
    
    - Encoder는 Masking 되어있지 않은 부분만 받고 정보를 encoding한다.
        - 이 Encoded Vector에 추론해야할 pixel 정보를 담아낸다.
        - 이 때, encoder가 활용하는 정보는 오로지 보이는 pixel 정보 뿐이다.
    - Decoder는 많은 정보가 encoded 된 vector를 통해서 전체 이미지를 구축한다.
    - 각 patch에서 mean/standard deviation을 구하고 해당 patch를 normalize한 값을 target으로 사용한다.
    - 손실함수의 경우 Masked patches에만 MSE를 활용하였다.
        - 모든 Pixel에 대해 구하는 것이 아니라 Masked region에만 계산 → 이유는 result-driven (정확도가 올라가서 그렇다고한다.)
    - Detail 1:  Mask를 75%까지 out 시키는 이유
        - Information을 '단순히' extra/inter-ploation해서 복구할 수 없는 수준까지 만들어야하기 때문이다.
    - Detail 2: Decoder의 input은 **positional embedding**이 사용됨
        - Mask tokens 이미지에서 어느 location에 위치하는지 information을 enforcing 하기 위함이다.
    

### 4. 구현
- Encoding
    - 모든 인풋 Patch에 대해서 Token을 생성한다.
      - Embedding한 후 Positional Encoding 시킨다.
    - Token 리스트를 Random하게 섞어준 후, List of Token에서 Last 75%를 없앤다.
- Decoding
    - Encoded Patch 리스트에 없앤 Token을 다시 붙여준다.
    - 전체 리스트를 원본 이미지처럼 Unshuffle 한다
    - Decoder에 Forward 해준다.
- Pixels vs. Tokens: Tokenization/Pixelization이 중요하지는 않다고한다.
