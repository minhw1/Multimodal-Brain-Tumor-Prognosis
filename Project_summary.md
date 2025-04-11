#  뇌종양 예후 예측
>환자의 MRI 영상 및 임상 정보를 기반으로, 지정된 시간 간격(ex: 6개월 후) 이후의 뇌종양 **부피Volume**를 수치적으로 정량 예측, **영역mask**을 예측하는 회귀 기반 모델을 개발한다.
<br><br>
## 흐름
+ 데이터 로딩 및 전처리(VIT&LLM) 후 정규화
  <br>-BraTS 데이터셋을 다운로드하고 MRI 이미지를 정규화
  <br>-환자의 메타데이터(나이, 성별, 병력 등)를 텍스트 형식으로 정리
+ Vision Transformer (ViT) 모델 구축
  <br>-MRI 영상에서 종양의 특징을 추출하는 모델
  <br>-사전 학습된 ViT 모델을 활용하거나, BraTS 데이터로 미세 조정(Fine-tuning)
+ LLM 기반 텍스트 인코더
  <br>-환자의 병력, 진료 기록 등을 벡터로 변환
  <br>-GPT 계열 모델이나 BERT 기반의 모델을 사용할 수 있음
+ 멀티모달 결합(Fusion Module)
  <br>-ViT의 영상 특징과 LLM의 임상 정보 특징을 결합
  <br>-Multimodal Fusion Module사용 가능
+ 회귀 모델 (부피 예측)=수치
  <br>-결합된 특징을 입력으로 받아 6개월 후의 종양 부피를 예측하는 회귀 모델 구현
  <br>-MSE(Mean Squared Error) 등의 손실 함수 사용
+ 세그멘테이션 모델 (Future Mask 예측)=형태
  <br>-결합된 특징을 입력으로 받아 6개월 후의 종양 마스크를 예측
  <br>-U-Net 또는 ViT 기반 세그멘테이션 모델 사용
<br><br>
## 구획화 모델
