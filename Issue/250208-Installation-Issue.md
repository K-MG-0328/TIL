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

### 추가 정리

#### 여러 JDK 버전을 병행 관리하는 패턴
macOS에서 여러 JDK를 깔끔하게 다루려면 다음 둘 중 하나가 일반적입니다.

**1. SDKMAN! 사용**
```bash
curl -s "https://get.sdkman.io" | bash
sdk install java 17-amzn
sdk install java 21-graal
sdk use java 17-amzn          # 현재 셸에서만 사용
sdk default java 21-graal     # 기본값 변경
```

**2. /usr/libexec/java_home + alias**
```bash
# .zshrc
alias java11='export JAVA_HOME=$(/usr/libexec/java_home -v 11)'
alias java17='export JAVA_HOME=$(/usr/libexec/java_home -v 17)'
```

`/usr/libexec/java_home`이 인식하지 않는 위치(예: Homebrew의 openjdk)에 있는 JDK는 다음 명령으로 시스템 JDK 디렉터리에 심볼릭 링크를 걸어 등록할 수 있습니다.
```bash
sudo ln -sfn /opt/homebrew/Cellar/openjdk@11/<버전>/libexec/openjdk.jdk \
    /Library/Java/JavaVirtualMachines/openjdk-11.jdk
```

#### "손상됨" 에러가 떴을 때 진단 체크리스트
1. `xattr -l <파일>`로 격리 속성 확인
2. `com.apple.quarantine` 있으면 `xattr -dr com.apple.quarantine <파일>` (`-r`로 디렉토리 재귀 제거)
3. 코드 서명 확인: `codesign -dv --verbose=4 <앱>`
4. Gatekeeper 정책: 시스템 설정 → 개인정보 보호 및 보안 → "확인되지 않은 개발자" 허용
5. 그래도 안 되면 보통 zip이 macOS Archive Utility로 풀리며 깨진 경우가 많아 `unzip <파일>`로 다시 풀기

#### xattr 사용 시 주의할 점
`xattr -d com.apple.quarantine`은 격리 태그를 제거해 Gatekeeper 검증을 우회합니다. **신뢰할 수 있는 소스**(공식 사이트, 회사 내부 배포)에서만 사용하세요. 다운로드한 무명 바이너리에 무차별로 적용하면 악성 코드 실행 위험이 있습니다.

#### nGrinder/Scouter Mac 설치 시 흔한 추가 이슈
- **Java 11 호환성**: nGrinder 3.5.x 컨트롤러는 Java 11 기반. Java 17로 실행하면 `--add-opens` 옵션 다수 필요.
  ```bash
  java --add-opens java.base/java.lang=ALL-UNNAMED \
       --add-opens java.base/java.util=ALL-UNNAMED \
       -jar ngrinder-controller-3.5.9-p1.war
  ```
- **포트 충돌**: 기본 8080이 이미 점유되어 있으면 `--port` 옵션 또는 `webapp.port` 설정 변경
- **Apple Silicon**: x86_64 전용 바이너리는 Rosetta로 실행되거나 실패. arm64 빌드 또는 `arch -x86_64` 접두로 우회

#### 일반적인 진단 체크리스트
```bash
# 1. JAVA_HOME과 PATH 확인
echo "JAVA_HOME=$JAVA_HOME"
echo "PATH=$PATH" | tr ':' '\n' | grep -i java
which java
java -version

# 2. 실제 어떤 java가 호출되는지
type -a java

# 3. 권한 문제
ls -la <실행파일>
```
