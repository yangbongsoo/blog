# 모든 객체의 공통 메서드
Object에 정의된 비-final 메서드(equals, hashCode, toString, clone, finalize)의 명시적인 일반 규약들에 대해서 알아보자.(종료자 finalize는 제외) 추가로 Object의 메서드는 아니지만 특성이 비슷한 Comparable.compareTo도 알아보자.
###규칙8 : equals를 재정의할 때는 일반 규약을 따르라

