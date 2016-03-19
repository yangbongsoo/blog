# Java8 in Action
**Stream processing** : stream이란 한번에 한 개씩 만들어지는 연속적인 데이터 항목들의 모임을 말한다. <br>
**동작 파라미터화** : 메서드를 다른 메서드의 인수로 넘겨주는 기능을 제공한다. <br>
**병렬성과 공유 가변 데이터** : 다른 코드와 동시에 실행하더라도 안전하게 실행할 수 있는 코드를 만드려면 공유된 가변 데이터에 접근하지 말아야 한다. 이런 함수를 pure 함수, stateless 함수라 불느다. <br>

###자바 함수
자바8에서는 함수를 새로운 값의 형식으로 추가했다.(즉, 함수 자체가 값) <br>

**메서드 레퍼런스**<br>
ex) 디렉토리에서 모든 숨겨진 파일을 필터링하는 문제에서 우선 주어진 파일이 숨겨져 있는지 체크하는 기능
```
File[] hiddenFiles = new File(“.”).listFiles(new FileFilter(){
	public boolean accept(File file){
		return file.isHidden(); //숨겨진 파일 필터링. 
	}
}
```
위의 코드를 보면 자바8 전까지는 File 클래스에 이미 isHidden이라는 메서드가 있는데 FileFilter로 감싼 다음에 FileFilter를 인스턴스화해야 했다. 
```
File[] hiddenFiles = new File(“.”).listFiles(File::isHidden);
```
**자바8의 메서드 레퍼런스 ::** (이 메서드를 값으로 사용하라는 의미)를 이용해서 listFiles에 직접 전달할 수 있다. 기존에 객체 레퍼런스(new로 객체 레퍼런스를 생성함)를 이용해서 객체를 이리저리 주고받았던 것처럼 자바 8에서는 File::isHidden을 이용해서 메서드 레퍼런스를 만들어 전달할 수 있게 되었다. <br>