# KorquadVer2
MRC 스터디를 진행하다가 SQuAD 부터 Korquad 까지 스터디하게 되었고, 진행하면서 어려움을 겪은 부분이나 이해한 부분을 공유하고자 작성합니다.

## Korquad란?
Korquad 는 SQuAD 한국어 버전이라고 보면 됩니다.
SQuAD란 QA(Question-Answering) 데이터셋의 한 종류입니다.
wiki 에서 본문 데이터를 가져왔고 wiki의 본문에 질문, 그리고 본문 내에 포함하는 정답을 가지는 구조로 되어 있습니다. 
### Korquad 데이터 구조(ver 1)
![코쿼드데이터구조](https://user-images.githubusercontent.com/45644085/144961669-625e5302-4609-40ef-8e1e-071b90eaca10.JPG)
Korquad 학습 데이터 구조는 다음과 같습니다.
data 라는 list 안에 여러개의 title을 가진 paragraphs가 존재하고 해당 paragraphs는 context라는 내용(본문)을 가지고 있으며 qas를 여러개 가지고 잇습니다. qas는 id, question, answer를 가지고 있습니다.  

### Korquad 학습 원리
![image](https://user-images.githubusercontent.com/45644085/145139561-52c97934-9ea0-4b83-94c0-6e01c7073c41.png)
[이미지 출처 - https://amber-chaeeunk.tistory.com/104?category=986609]

간단하게 MRC 학습을 설명하자면, context(본문)과 question(질문)을 tokenizer를 통해 토큰화하여 input(X)으로 넣습니다. 그 때 해당 질문에 대한 answer(답변)의 본문에서 몇 번째 토큰인지 start_position과 end_position을 input(Y) 값으로 넣어줍니다. 해당 input(X)는 기반으로 사용되는 사전학습 언어모델을 통해 embedding이 될 것이고 해당 embedding - start/end position을 classifier로 학습하게 됩니다. 즉, 본문과 질문의 임베딩 값을 넣었을 때 답변의 위치를 찾도록 학습하는 것입니다.

