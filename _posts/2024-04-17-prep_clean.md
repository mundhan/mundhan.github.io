---
layout: single
title: "데이터 정제"
---

* FCE
  * text: 원본 문장
  * edits: 교정 시작 오프셋, 교정 끝 오프셋, 교정 결과, 오류 유형
    * 오프셋은 문자 단위의 char index인데, 이를 errant 툴킷을 통해 토큰 단위의 token index로 바꾸어주어야 한다. 
<pre>
  <code>
{"text": "13th June 2000\n\nDear Ms Helen Ryan\n\nCompetition Organiser\n\nI have just recieved the letter, which lets me know that I have won the first prize. I am proud of winning it and would like to say how thankful I am. I am therefore writing to give you my further information.\n\nFirst of all, I am a student and would like to travel in July. Because I intend to take  examination in Septenber. So I can afford to go on holiday only in July. Secondly, I would prefer to stay at log cabins rather than tents. It will be my first time to experience to stay there and I am interested in trying new things which I have not tried.\n\nThe next consideration is the activities. Actually, I used to play basketball for 6 years and am quite confident of it. And nowadays, I have been practicing tennis as  school activity. Therefore I would like to choose basketball and tennis.\n\nFinally, I am considering about what kind of clothes I should wear and how much I need. I would be most grateful if you could give me further advicable information.\n\nI look forward to hearing from you.\n\nYours sincerely,", "age": "26-30", "q": "1", "script-s": "27", "edits": [[0, [[71, 79, "received", "S"], [195, 203, "grateful", "RJ"], [333, 340, "This is because", "M"], [358, 358, "an", "MD"], [374, 383, "September", "S"], [465, 467, "in", "RT"], [523, 536, "experiencing", "FV"], [537, 544, "staying", "FV"], [730, 732, "at", "RT"], [741, 749, "recently", "RY"], [763, 773, "practising", "SA"], [784, 784, "a", "MD"], [873, 884, "wondering", "RV"], [1001, 1010, null, "DJ"]]]], "l1": "ja", "id": "TR27*0100*2000*01", "answer-s": "4.3"}
  </code>
</pre>

* 이 때, bea-19 task에서 제공한 gold reference의 경우(아래 예시) 토큰마다 공백으로 구분되어 있다.
* 이로 인해, gold reference를 그대로 나의 모델로 교정하면 실제로 원본에는 구두점 오류가 없었지만, 교정 내용에 구두점 교정이 불필요하게 포함된다.

<pre>
  <code>
S I have just recieved the letter , which lets me know that I have won the first prize .
A 3 4|||R:SPELL|||received|||REQUIRED|||-NONE-|||0
  </code>
</pre>

<pre>
  <code>
S I have just recieved the letter , which lets me know that I have won the first prize .
A 3 4|||R:SPELL|||received|||REQUIRED|||-NONE-|||0
A 6 8|||R:ORTH|||letter,|||REQUIRED|||-NONE-|||0
  </code>
</pre>

- - - 

그래서 토큰 단위로 공백 분할되지 않도록, 원본 문장으로부터 문장을 가져오려고 한다. 
이때, 문장 분할 기준을 어떻게 설정해야 할 지 고민.
* 문장 분할 기준이 맞지 않으면, 문장 개수의 차이 등으로 결과 비교가 어려움

* 교정 전 후 문장 개수 차이가 나서 확인해 보니 아래와 같은 문장이 교정 결과에서 삭제된다.
'''
Yours Sincerely,
'''
  * 때문에, 인덱스를 설정하고 해당 인덱스끼리의 비교가 필요할 것으로 보인다.

* g4.py: 원본 입력문장(B_dev_input_문장개수)으로 교정 문장 추출
* errant_parallel: 원본 입력문장(B_dev_input_문장개수)과 교정문장(g4_모델_레벨_문장개수) 비교 -> m2 파일
  '''
  errant_parallel -orig [원본입력문장] -cor [교정문장] -out [저장할파일명]
  '''
* errant_compare: 원본 m2 파일과 교정 m2 파일 비교
  '''
  errant_compare -hyp [교정m2파일명] -ref [원본m2파일명]
  '''
