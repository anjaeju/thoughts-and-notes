## Title: Masked Autoencoders As Spatiotemporal Learners

Author: Christoph Feichtenhofer, Haoqi Fan, Yanghao Li, Kaiming He

---

### 1. Prerequisite

- Image-Level에서 수행하던 MAE pre-training을 Video 데이터 및 모델에도 진행하기 위해 확장한 방법론이다.
    - Facebook Research는 지금 video에 집중하고있다.
        - Video DL Tool (Pytorchvideo, SlowFask)과 Kaiming He를 필두로 Video DL 모델 개발을 하는 것으로 유추가 가능하다
    - 비디오 기반의 DL은 실용적인 면에서 `속도`가 중요하고 이미지 수준의 연구보다도 한 차원 높은 larger-scale 데이터셋과 시간이 필요하다

- Transformer-based Masked Autoencoder를 이용해 SSL을 적용하는 이유
    - Transformer의 경우 global attention을 기준으로 데이터에서 feature를 학습하기 때문에 MAE기반의 SSL과 어울린다.
        - Local Patch를 통해서 Global information을 추측할 수 있다면 Global Feature에 대한 이해가 높다고 판단할 수 있다.
        - CNN은 local feature를 기준으로 representation learning하기 때문에 MAE-based SSL을 사용하기에 적절하지 않다고 언급.

### 2. Research Question

- Image에 대한 MAE연구를 성공적으로 끝냈으니, video에도 scalable하게 적용해보자
    - Challenge 1. Higher redundancy in Video data than Image

### 3. Method

- 논문에서 제안하는 방법론은 시간축 Time을 늘려준 후 Reconstruction task를 진행하는 것이다.
    - Input을 masking 할 때 Frame까지 반영하여, Transformer-based encoder가 Masking 되어있는 비디오 전체를 복구한다.
    
    <p align="center">
      <img src="https://user-images.githubusercontent.com/40862925/189530429-696214bf-5730-4a34-a779-3fcb286eeaac.png" style="padding: 0;margin:0;">
    </p>

- Video에 대한 MAE 모델은 다음의 구성요소를 포함한다.
    - Patch Embedding, Masking, Autoencoding
    - 1) Patch embedding
        - 비디오 클립을 ***non-overlapping*** regular grid patches로 분리한다.
        - Patch는 flattened되고 linear projection을 통해서 embedding된다.
        - Embedded Token에 Positional embedding 처리를 해준다.
    - 2) Masking (Input 만들기)
        - Randomly 하게 Masking을 진행한다.
            - 이때 random masking 하는 과정을 본 논문에서는 agnostic 이라고 정의한다.
            - Advantages of agnostic masking
            
            <p align="center">
            <img src="https://user-images.githubusercontent.com/40862925/189530443-b8be1f61-309f-4fec-b846-052f111e2d3d.png" style="padding: 0;margin:0;">
            </p>
                
        - 본 논문에서는 90%의 Masking ratio 사용한다.
            - 비디오를 사용하기 때문에 시간축에 대한 정보를 반영해 더 많이 Masking 해야한다고 한다.
            - 즉, 시간축을 활용하면 이미지 보다 좀 더 쉽게 비디오를 복구할 수 있기 때문에 이를 방지하는 것.
            - Encoder는 global information을 예측하는 데 어렵게 배워야 더 좋은 pre-trained weight를 갖을 수 있다.
    - 3) Autoencoding
        - Encoder에 들어갈 때는, Only visible set of embedded patches를 forwarding

### 3. Interesting

- Controlling # of Seens
    - 현재 He는 각 sample당 '# of seens'을 맞춰주려고하고있다.
- About Data Augmentation
    - video에는 자연적으로 많은 'Temporal' augmentation을 포함한다고 주장한다.
        - View point: 뷰 포인트를 변화시키는 것도 model robustness에 도움이 된다.
        - Motion
        - Deformation
        - Occlusion
    - 비디오 데이터에 대해서 'Spatial' augmentation을 진행하니 크게 도움이 되지 않는 것을 증명하였다.

- 논문에서는 SSL을 통해서 useful knowledge를 배웠다고 언급
    - Despite minimal inductive biases, our method achieves strong empirical results, suggesting that useful knowledge can be learned from data.
    - SSL을 통해서 pre-trained 된 weight가 준비되어졌고, 이 Pre-trained weights는 Task-specific한 information 뿐만 아니라 다른 우리가 모르는 useful knowledge도 추출하게된다. 결과적으로 dowstream task에서 성능이 향상되면서, Task-Specific한 information에 도움이 되는 knowledge를 준비할 수 있었다는 것이 증명된다.

- Our work is done independently and concurrently with [66] on a related method.
    - 너무 비슷한 시기에 비슷한 방법론이 논문화되어서, 명확히 표기하고 비교하지는 않았지만, VideoMAE 라는 논문이있다.
    - 본 논문도 MAE 방법론을 Video에 적용시킨 것이며 어느 논문이 우수하다고 판단할 수 없지만 접근성 면에서 VideoMAE는 code를 포함하고있어 참고해보면 좋을 것 같다.
