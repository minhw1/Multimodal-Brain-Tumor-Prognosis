#  뇌종양 예후 예측
환자의 MRI 영상 및 임상 정보를 기반으로, 지정된 시간 간격(ex: 6개월 후) 이후의 뇌종양 **부피Volume**를 수치적으로 정량 예측, **영역mask**을 예측하는 회귀 기반 모델을 개발한다.
<br><br>

## 흐름
+ 데이터 로딩 및 전처리 후 정규화
  <br>-BraTS 데이터셋(2021)을 다운로드하고 MRI 이미지를 정규화
  ~~<br>-환자의 메타데이터(나이, 성별, 병력 등)를 텍스트 형식으로 정리~~
+ U-Net 모델 구축
  <br>-MRI 영상에서 종양의 위치를 추출하는 모델
+ ~~LLM 기반 텍스트 인코더
  <br>-환자의 병력, 진료 기록 등을 벡터로 변환~~
+ ~~멀티모달 결합
  <br>-만약 텍스트 정보도 활용한다면 영상 특징과 LLM의 임상 정보 특징을 결합해야함~~
---
+  미래 종양 부피 예측 모델 구축
  <br>-결합된 특징을 입력으로 받아 N개월 후의 종양 부피를 예측하는 회귀 모델 구현
+  미래 종양 위치 예측 모델 구축
  <br>-결합된 특징을 입력으로 받아 N개월 후의 종양 마스크를 예측
  <br>-U-Net 또는 ViT 또는 GAN 기반 세그멘테이션 모델 사용
<br><br>

## 사용 데이터
BraTS2021
>+ 의료 영상 자료
>+ 3D 뇌 MRI 스캔을 기반으로 한 볼륨 데이터
>+ 한 환자의 MRI는 여러 슬라이스로 구성된 3차원 영상이며, 보통 다음과 같은 4가지 시퀀스가 제공된다.
> ![Image](https://github.com/user-attachments/assets/3b1877c0-d128-48cd-b9f3-8f667014f5e3)
>>1. ~~T1-weighted (T1)~~: 기본적인 해부학 정보제공 주로 사용안함
>>2. T1 with contrast enhancement (T1ce) : 종양의 활성 부위 잘 보임
>>3. T2-weighted (T2) : 종양 주위 부종 표현에 유리
>>4. FLAIR (Fluid Attenuated Inversion Recovery) : 종양 영역 더 잘보이게 함함
<br><br>

## 주요 기술
### 1. 전처리
모델에 데이터를 넣기 전에 데이터를 깨끗하게 정리하고 준비하는 과정
+ 정규화 ( 크기, 밝기 등 )
>  보통 픽셀 값을 0~1 사이 혹은 Z-score 정규화로 변경  [ 차이점 ](#정규화-비교) <br>
> ![Image](https://github.com/user-attachments/assets/2ea431be-e9ac-4792-8f74-32e3c3332a55) <br>
> **x = (x-μ)/σ (x>0)**<br>
>  위의 식에서 x>0은 MRI 영상의 뇌종양 영역이고, μ는 데이터 전체의 평균치, σ 는 데이터의 표준 편차를 나타낸다.
+ 배경 제거 ( 데이터 상태에 따라 다름 )

### 2. 데이터 증강 기법
기존에 있던 데이터를 인위적으로 회전, 이동, 확대 등을 통하여 기존의 데이터를 확장시켜 학습에 충분한 데이터를 확보한다.
![Image](https://github.com/user-attachments/assets/3bbb0489-1671-4fcb-b303-fff603008556)

### 3. 종양 구획화 모델(segmentation)
U-Net 활용
이미지를 작게 줄여가면서 중요한 특징을 뽑고, 다시 복원하면서 세밀한 위치 정보를 복구
>CNN 기반 구조 <br>
![image](https://github.com/user-attachments/assets/15435108-96f6-47be-8bf4-9d15cf4d9378)

https://www.kaggle.com/code/domingao/iou-82-brain-tumor-segmentation
 에서 제공되는 구획화 모델 코드 및 기법을 활용할 예정<br> 
 <a name="구획화모델코드"></a>
<br>  [구획화 모델 코드.pdf](BraTS21_visualization.pdf)

~~4. 텍스트 인코더
진료 기록, 병력 등의 텍스트를 벡터로 변환
LLM 기반 ( bert 보다 넓은 개념, 일반적으로 bert로도 충분 )~~

### 5. 미래 종양 부피 예측
수치 <br>
회귀 모델,CNN + MLP 활용

### 6. 미래 종양 위치 예측
형태 <br>
CNN+LSTM 활용

---
## 사용 툴
PyCharm <br>
VS Code

## 사용 데이터셋
1. Brain Tumor Progression Dataset (andrewmvd/brain-tumor-progression) <br>
+ 형식: 2D 슬라이스 (3D MRI를 2D로 분할) <br>
+ 특징: 환자별 종양 진행 과정을 2D 이미지로 기록. 종양 마스크 포함. <br>

2. BraTS 2021 Task 1 Dataset (dschettler8845/brats-2021-task1) <br>
+ 형식: 3D 볼륨 데이터 (NIfTI 포맷) <br>
+ 특징: 멀티시퀀스 MRI(T1, T1c, T2, FLAIR)와 정밀 종양 세그멘테이션 마스크 포함. <br>
**2D로 변환 필요!**
 [ 변환 코드 ](#구획화모델코드)
---
## 정규화 비교
| 항목           | 0~1 스케일링                | Z-score 정규화                      |
|----------------|------------------------------|-------------------------------------|
| 기준           | 최솟값, 최댓값  (0~1)             | 평균(0), 표준편차(1)                      |
| 아웃라이어 민감도 | 민감함                       | 덜 민감함                           |
| 주요 사용처     | CNN, 기본 이미지 전처리      | ViT, 통계 기반 모델, 분석 전처리   |
| 정렬 효과       | 스케일 맞춤                  | 중심 위치 및 분산 정렬             |
