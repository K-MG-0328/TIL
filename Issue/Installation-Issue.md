### nGrinder, Scouter 설치 시 이슈 사항 및 처리 방법들

#### Mac JDK 경로 
Mac의 기본 Java 경로 /Library/Java/JavaVirtualMachines/ 이외에는 /usr/libexec/java_home -V java_home -V 에 JDK를 찾지 못함  
java_home : mac에서 설치된 java 버전들을 관리하고 경로를 제공하는 유틸리티  

홈브루 오라클 jdk 경로 11.0.26  
/opt/homebrew/Cellar/openjdk@11  
아마존 jdk 경로  
/Users/gimmingyu/Library/Java/JavaVirtualMachines/corretto-17.0.14    
  
/usr/libexec/java_home -V 시 17만 표출  

java_home에 기본 경로 이외에 jdk 등록하는 방법  

JAVA_HOME을 직접 설정 .zprofile에서 수정해서 환경변수 영구적으로 적용   
export JAVA_HOME=/opt/homebrew/Cellar/openjdk@11/11.0.26/libexec/openjdk.jdk/Contents/Home  
export PATH=$JAVA_HOME/bin:$PATH  

#### Mac에서 다운로드한 파일이 "손상됨" 메시지를 표시할 때 
xattr로 파일 무결성 확인  
xattr -l scouter.client.app  
xattr macOS에서 파일의 확장 속성을 관리하는 명령어 -l 옵션을 사용하면 파일의 속성을 확인가능  
  
com.apple.quarantine: 0081;679a56d9;Chrome;  
com.apple.quarantine 속성은 macOS의 Gatekeeper(보안 시스템)가 부여하는 태그  
인터넷에서 다운로드된 파일은 자동으로 “격리(quarantine)” 상태가 됨  
  
xattr -d com.apple.quarantine ./scouter.client.app 로 해당 속성 제거  
