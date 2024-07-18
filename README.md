# ClickBaitNews
**ClickBaitNews Classification And Generate News Title**\
**낚시성 기사 탐지 및 새로운 제목 생성**\
팀원: 김효민, 이예진, 정태영\
프로젝트 기간 : 23.5.4~6.12**

**분석 데이터 : AI Hub 데이터 셋 이용** https://www.aihub.or.kr/aihubdata/data/view.do?currMenu=115&topMenu=100&dataSetSn=71338 

**Train data: 233,166개**\
**Valid data: 58,292개**\
**Test data: 36,422개**


# 프로젝트 선정 이유
![image](https://github.com/user-attachments/assets/3c959eb4-bf07-4bf8-bf4e-05005c78e6ec)
![image](https://github.com/user-attachments/assets/752e22ab-6497-4fcd-a074-4fc53e3f4d88)

위 이미지처럼 많은 인터넷 뉴스 기사의 제목이 클릭을 유도하는 낚시성 기사인 것을 알 수 있음.\
뿐만 아니라, 앞으로 바뀔 기사 노출 알고리즘이 적용되면 더 많은 낚시성 기사가 생성될 우려가 있음.\
또, 기사를 클릭할 때마다 요금을 부과하는 요금제가 적용될 경우 이러한 낚시성 기사는 소비자에게 피해를 줄 수 있음.\
뿐만 아니라, 바쁜 현대 삶속에서 기사의 전문을 읽지 않고 제목만 읽고 트렌드를 파악하는 사람의 수가 늘어나고 있음.

**따라서, 낚시성 제목의 기사를 모델을 통해 분류하고 추후 피해 방지를 목적으로 적절한 제목을 생성해주는 알고리즘을 개발하려고 함.**\
**낚시성 기사가 증가함에 따라 잘못된 정보에 쉽게 노출이 되고 고착화 되면 사회적인 문제로도 커질 수 있음.**\
**또한 잘못된 기사를 체크하고, 수작업으로 제목을 생성하는 과정에서 많은 시간과 비용이 들어가므로 비즈니스적으로도 가치가 있음.**

# 전체 프로젝트 파이프라인
![image](https://github.com/user-attachments/assets/49f5ccb3-5f06-413a-9a56-514053f7bde3)

뉴스의 제목과 본문을 분리하여 낚시성 기사인지 아닌지 판단하는 Clickbait Classification 모델에 넣어 낚시성 기사인지 아닌지를 먼저 파악함.\
만약 낚시성 기사라고 판단이 된다면 본문의 내용을 바탕으로 제목을 생성하는 New Title Generation 모델에 넣어 새로운 제목을 생성.\
생성된 제목과 실제 참 제목과의 유사도를 통해 새로운 제목을 평가함.

# 낚시성 기사 분류 모델 예측 과정
![image](https://github.com/user-attachments/assets/a38dd7d9-5ad6-4d49-9ab9-a58f6db685f2)
낚시성 기사를 분류하는데는 Sentence-BERT를 fine-tuning 후 이용함.\
Sentence A 에는 전처리된 제목을 B에는 전처리된 본문을 넣어 각각 임베딩을 진행.\
제목이 본문의 내용을 잘 담고 있다면 각 벡터간 유사도가 높을거라 판단.

모델 세부내용
1. Data : 낚시성/ 비낚시성 제목, 본문, Label 
2. PreTrained BERT : klue/roberta-base
3. pooling :mean pooling
4. loss function : ContrastiveLoss, Online ContrastiveLoss

# 낚시성 기사 분류 모델 성능 평가
![image](https://github.com/user-attachments/assets/61f89cc0-4268-4a4e-9aaa-64f22db125c4)
![image](https://github.com/user-attachments/assets/a52298aa-9b11-4993-8881-a7e6d75b4796)
OnlineContrastive Loss의 성능이 비슷하지만 조금 더 좋아 해당 경우의 성능 그래프 및 지표임.\
**유사도 계산방식을 어떤식으로 하더라도 해당 모델을 통한 분류 결과 Accuracy, F1 등 다양한 평가지표에서 정확도가 높은 것을 알 수 있음.**

# 제목 생성 모델 생성 과정
![image](https://github.com/user-attachments/assets/9908fa5b-cafd-410a-b945-c472d6968971)
본문의 내용을 잘 담고 있고 요약하는 것이 제목을 대체할 수 있다고 생각.\
단, 제목의 경우 2개의 KOBART 모델을 fine-tuning 후 이용해 제목을 생성함
- 본문 전체의 내용을 1개의 모델로 요약할 경우 뉴스 기사의 제목인 헤드라인 형태가 아닌 줄글 형태로 나오는 것을 고려함.
- 따라서, 1차 모델로 본문의 내용을 요약하고 요약한 핵심내용을 바탕으로 2차 모델을 이용해 제목을 생성함.
- 분류와 마찬가지로 실제 참 제목과 생성한 새로운 제목간의 유사도를 바탕으로 평가함.

모델 세부 내용
1. Data :비낚시성 기사의 제목, 기사본문 요약문 
2. PreTrained KOBART : gogamza/kobart-base-v2
3. evaluation : 코사인 유사도, 맨하탄 유사도, ROUGE-1, ROUGE-L

# 생성 모델 성능평가 
![image](https://github.com/user-attachments/assets/b3dfc456-b0b9-4df7-b2ab-2c7efbb65384)
해당 낚시성 데이터의 경우 제목과 본문의 내용이 아예 다른 경우임.\
따라서 낚시성 제목의 경우 유사도나 ROUGE score가 아예 관계 없는 0에 근접함을 알 수 있음.\
반면 생성한 제목의 경우 높은 수치는 아니지만 조금더 score점수가 좋게 나온것을 알 수 있음.

# 향후 과제
1. 분류모델의 학습 데이터 확장 : 해당 프로젝트에서는 가장 시급하다고 생각되는 문제인 제목과 본문의 불일치 내용만이 아닌 다른 경우도 학습해야 함.\
-- 낚시성 제목의 경우 본문과의 불일치 외에도 자극적인 내용, 과장된 제목등이 포함 됨. 해당 내용도 구성을 하여 범용적인 모델을 만들어야 함.

2. 모델 경량화 : 실제 서비스에 만약 활용한다면 제목 생성 시간이 중요해짐. 이에 따라 모델 경량화가 필요.

3. 생성 모델의 평가 지표 : 한글은 순서보다 조사가 중요하기에 ROUGE-U, RDASS 등 조금 더 한글의 특성을 반영하는 평가지표를 추가로 고려해야 함. 


