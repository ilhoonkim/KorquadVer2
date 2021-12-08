# **KorquadVer2**
MRC 스터디를 진행하다가 SQuAD 부터 Korquad 까지 스터디하게 되었고, 진행하면서 어려움을 겪은 부분이나 이해한 부분을 공유하고자 작성합니다.

## Korquad란?
Korquad 는 SQuAD 한국어 버전이라고 보면 됩니다.
SQuAD란 QA(Question-Answering) 데이터셋의 한 종류입니다.
wiki 에서 본문 데이터를 가져왔고 wiki의 본문에 질문, 그리고 본문 내에 포함하는 정답을 가지는 구조로 되어 있습니다. 
### **Korquad 데이터 구조(ver 1)**
![코쿼드데이터구조](https://user-images.githubusercontent.com/45644085/144961669-625e5302-4609-40ef-8e1e-071b90eaca10.JPG)
Korquad 학습 데이터 구조는 다음과 같습니다.
data 라는 list 안에 여러개의 title을 가진 paragraphs가 존재하고 해당 paragraphs는 context라는 내용(본문)을 가지고 있으며 qas를 여러개 가지고 잇습니다. qas는 id, question, answer를 가지고 있습니다.  

### **Korquad 학습 원리**
![image](https://user-images.githubusercontent.com/45644085/145139561-52c97934-9ea0-4b83-94c0-6e01c7073c41.png)
[이미지 출처 - https://amber-chaeeunk.tistory.com/104?category=986609]

간단하게 MRC 학습을 설명하자면, context(본문)과 question(질문)을 tokenizer를 통해 토큰화하여 input(X)으로 넣습니다. 그 때 해당 질문에 대한 answer(답변)의 본문에서 몇 번째 토큰인지 start_position과 end_position을 input(Y) 값으로 넣어줍니다. 해당 input(X)는 기반으로 사용되는 사전학습 언어모델을 통해 embedding이 될 것이고 해당 embedding - start/end position을 classifier로 학습하게 됩니다. 즉, 본문과 질문의 임베딩 값을 넣었을 때 답변의 위치를 찾도록 학습하는 것입니다.

### Korquad 데이터 구조(ver 2)
SQuAD가 1버전에서 2버전대로 변경되면서 데이터에도 변화가 생겼습니다.
기존에 SQuAD 1 버전에서는 본문 내에서 답변이 없을 경우에도 부정확한 답변을 주기도 하였습니다. 
이러한 점을 개선하고자 qas에서 answer가 context안에 없는 경우 is_impossible이라는 추가 변수를 true로 주어서 해당 문제까지 추가로 학습하도록 하였습니다.
해당 문제를 함께 학습함으로써 질문과 관련없는 본문에서는 답변할 수 없음을 인지하여 답변을 찾지 않을수 있게 되어 부정확한 답변을 주는 케이스를 방지한 것 같습니다.

#### **-Korquad의 추가 변경사항은?**
Korquad도 SQuAD와 마찬가지로 답변 불가능한 질문에 대한 학습을 추가하였습니다. 
해당 문제에 추가로 변경된 점이 있다면 기존의 512 토큰 길이(max_paragraph_length) 이내로 제한되었던 context를 위키 백과의 한 페이지 html을 사용하였다는 것입니다.
해당 변화로 인해 학습 이전에 긴 10000 token 이상의 html 원문을 학습가능한 형태인 512 토큰 이내의 paragraph로 분할해주는 작업이 필요하게 되었습니다. 
html 원문을 전처리를 통해 필요없는 tag를 제거하거나 다른 토큰으로 변환하는 작업이 필요하며, 해당 작업을 통해 테이블 형태의 데이터도 토큰으로 변환하여 들어오게 되어 
테이블 내에서도 기계독해가 가능하도록 하였습니다. 해당 전처리 코드는 github 등에 공개된 바가 거의 없고 JoungheeKim님이 korean-question-answer-system github에 korquad 처리를 위한 여러 전처리 코드(wiki_convert, preprocess 등) 을 올리셧으나 현재는 보이지 않는 상황입니다. 처음에 해당 코드를 활용하여 사용하였습니다. 해당 코드는 table을 |,/을 사용하여 의미론적으로 묶는 코드는 아니라 해당 부분은 변환이 필요하였습니다. 해당 부분은 추후에 추가로 설명드리겠습니다.

긴 html context를 여러개의 512 토큰 이내의 paragraphs로 나누고 각 parpagraphs에 기존의 html context에 붙어있던 qas의 질문을 함께 넣고 해당 본문에 답변이 있으면 is_impossible은 false가 되고 답변의 위치를 주게 됩니다. 반대로 나누어진 paragraphs에 답변이 없으면 is_impossible은 true가 되고 답변은 없게 됩니다.
이러한 korquad 2 버전의 변화로 인해, 짧은 기사 단위가 아니라 긴 문서도 html로 변환하여 기계독해를 시도해볼 수 있게 되었다는 점에서 의미가 있다고 봅니다.

### Korquad 2.1 학습 시 겪는 문제
개인이 Korquad 2 버전을 학습하려고 시도하다가 아마 대부분 컴퓨팅 문제로 포기하게 될 것입니다. 
아마 컴퓨팅이 1버전에 비해 약 10~20배는 더 필요해보입니다. json 파일을 불러와서 examples를 만드는 도중 broken pipe 혹은 다른 memory 관련 에러로 문제를 겪으실것입니다.
하물며 256G RAM 서버에서도 전체파일을 한번에 읽어오면 example까지는 만들어지나 feature로 변환하는 작업에서 터지는걸 확인했습니다.

그래서 저는 62G RAM , 1080TI 하나로도 학습할 수 있도록 하기 위하여 korquad 2.1 의 38개의 train.json 파일을 각각 1개씩 읽어 example 및 feature로 변환한 cached 파일로 변환하고 
각 cached 파일을 하나씩 읽어서 학습하는 코드를 작성하여 학습하고 있습니다.


