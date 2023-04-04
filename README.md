# Focal-Loss-for-Dense-Object-Detection

2018년도 Object Detection의 Sota는 R-CNN계열의 2-Stage detector 계열

하지만 RetinaNet은 1-stage detector로 2-stage detector와 같은 성능을 가짐.

# Class Imbalance 

Object Detection Model은 기본적으로 이미지 내의 객체 영역을 찾아내고, IoU(intersection over Union) Threshold 즉 실제 바운딩 박스와 예측 바운딩 박스의

겹치는 넓이를 Threshold에 따라 Positive/Negative로 나눈 뒤 학습을 진행한다

여기서 배경과 객체간의 class Imbalance 문제가 발생한다. 이미지 내의 객체를 나타내는 부분 즉 Positive sample과 Negative sample(배경) 사이의

차이가 크기 때문이다. 배경으로 인하여 대부분의 샘플이 easy negative가 되고, 그 수가 월등히 많아 학습에 끼치는 영향력이 커져 모델 성능이 하락하는 문제가

발생한다. 즉 배경때문에 객체들이 잘 학습이 되지 않는 것이다.

# One Stage vs Two Stage

Two Stage의 Class Imblance 해결책은 두 가지 방법이 있습니다.

1.Region Proposal을 통해 Background Sample을 걸러줍니다.->추후 설명 예정

2.Sampling Heuristic 방법 적용->매우 많은 객체 후보들이 처리되어 학습 시간이 매우 증가함

두 가지 방법은 추후 Two stage Model인 RCNN 코드 리뷰와 함께 자세히 진행하겠습니다.

One Stage Detector는 새로운 방식의 loss인 focal loss를 통한 해결 방법을 제시하고 Two stage Model보다 성능 또한 개선하게 됩니다.

![image](https://user-images.githubusercontent.com/104436260/229714926-ab0504fe-4674-47f6-a4f8-1a6b46f02ecb.png)

기존 Cross Entropy Loss와 Focal Loss의 차이점은 수식에서 (1−p)γ를 추가로 multiply하는 것입니다.

Focal Loss는 Class imbalance 문제를 해결하기 위하여 easy sample-> 배경의 경우 weight를 down시키고 hard example-> 인식하기 어려운 객체에 대한

weight를 증가시켜 학습을 집중합니다. focal Loss의 효율성을 위하여 Model의 Backbone으로는 ResNet-101을 기본적으로 사용합니다.

ResNet을 통해 객체의 특징을 추출하고 이후의 별도의 모듈을 사용하여 객체의 위치와 클래스를 예측합니다.

즉, backbone(ResNet-101)에서는 Convolutional feature map을 추출합니다. 그리고 다음 Network로는 subnet network를 통해 object classification을 수행하고 두번째 subnet

에서는 boudning box regression을 수행합니다.

![image](https://user-images.githubusercontent.com/104436260/229718950-646ca4d8-43b9-4649-ab3b-86f915f6c056.png)

# Feature Pyramid Network Backbone

RetinaNet의 backbone network로 FPN을 적용한다. FPN의 구조는 위 그림에서(b)에 해당하는 부분이다.

RetinaNet은 P3에서 P7의 Pyramid Level을 사용한다.

# FPN이란

Feature Pyramid란 다른 크기의 객체를 탐지하기 위한 시스템의 기본 구성 요소라고 생각하면 될 듯 하다.

최근에는 이러한 Pyramid represntation즉 사용을 자제하고 있다고 한다. 연산과 메모리 용량을 많이 차지하기 때문이다.

Resnet-101도 충분히 깊은 모델인데 거기서 나온 특성값을 또 한번 Network에 돌려주니 메모리 문제가 생기긴 할 듯 하다.

하지만 성능 향상에는 큰 도움이 되기 때문에 간략하게 FPN에 대해서 알아보고 넘어가도록 한다.

![image](https://user-images.githubusercontent.com/104436260/229720661-a4e2fb9e-e21f-44c2-a1c6-dcb024380185.png)

1. FPN은 임의의 크기 이미지를 입력 받고 fully convolutional 방법으로 다양한 단계의 적합한 크기의 feature map을 출력한다

여기서 fully convolutional은 모든 레이어가 합성공으로만 이루어진 모델이라고 하는데 마지막에 Fully connected를 해주지 않고

모든 픽셀위치의 출력값을 생성한다는 점이 특징이다. Fully connected 대신 average Pooling을 통해 출력값을 얻는다.
