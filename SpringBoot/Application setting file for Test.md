# 테스트용 설정 파일

#### 스프링부트에서 데이터베이스 설정, Hibernate 설정을 위해 resources하위에 application.yml 또는 application.properties를 생성한다.
#### 하지만 테스트하는 환경과 실제 애플리케이션을 실행하는 환경은 대게 다르다. 따라서, 테스트용 환경을 따로 만드는 것이 좋다.
#### 테스트는 격리된 환경에서 실행하고, 끝난 뒤에는 데이터를 초기화하는 것이 좋으므로, 메모리 DB를 사용하는 것이 가장 이상적이다.   
   
-----------------------------------------
![image](https://user-images.githubusercontent.com/60773356/119072884-3f189a80-ba27-11eb-8eee-5c1b930ee7dc.png)
* 운영 로직은 main 내부의 코드들이 우선권을 가지지만 테스트에서는 test 내부의 코드들이 우선권을 가짐.
* 즉, test/resources에 application.yml을 만들어놓으면 테스트를 할 때는 main의 application.yml이 무시되고 test의 application.yml이 실행.


![image](https://user-images.githubusercontent.com/60773356/119072943-5b1c3c00-ba27-11eb-9bda-edf61b5aa13d.png)
* 스프링 부트는 datasource 설정이 없으면, 기본적을 메모리 DB를 사용하고, driver-class도 현재 등록된 라이브러를 보고 찾아준다.
* 추가로 ddl-auto 도 create-drop 모드로 동작한다. (create을 하고 마지막에 초기화)
* 따라서 데이터소스나, JPA 관련된 별도의 추가 설정을 하지 않아도 된다.
* 테스트만을 위한 설정을 할 수 있기 때문에 엄청난 이점이 있다.
