IntelliJ에서 External Libraries에 있는 외부 클래스 파일 디버깅 방법에 대해서 

먼저 Java Decompiler plugin을 다운받는다</br>
![](decompiledplugin.PNG)

그리고  Decompiled .class file ... 여기서 Download Source를 누른다. </br>
![](classfile이 디버깅되나.PNG)


이제 디버깅을 할 수 있게 된다. </br>
![](okok.PNG)

그런데 External Libraries에 있는 모든 클래스 파일을 java파일로 디컴파일 가능할 줄 알았는데 그건 또 아니다. 어떤건 눌러도 변환되지 않는다. (그 이유 아직 모르겠음) </br>

그리고 이걸 한 후로부터 디버깅하는거 에 대해 좀 꼬이게 됐다. (이유 분석중)</br>

새로운 프로젝트 열어서 download source 했는데 안되네 ... 일관적이지 않은 이유가 뭘까 <br>
드디어 답을 알게 되었다. 다운로드하면 메이븐 레파지토리에 자바 파일이 있어서 다운로드가 되는거고 
디컴파일해서 아무런 변화가 없는 경우는 소스코드를 못찾은 경우 ㅋㅋ 
---


![](xss 원리.PNG)

encodingFilter를 lucy-xss-filter설정보다 먼저안하거나 설정을 아예 안하게됐을때 문제

