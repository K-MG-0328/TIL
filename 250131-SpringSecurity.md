## 인증 처리
### AuthenticationConfiguration은 언제 생성되는가?
별도로 @Bean 등록할 필요 없이 Spring Security가 내부적으로 설정합니다.
### AuthenticationConfiguration은 어디에서 설정할 수 있는가?
AuthenticationConfiguration 자체는 Spring Security 내부에서 관리되므로, 직접 설정할 필요는 없습니다.

### HttpSecurity가 관리하는 공유 객체
#### AuthenticationManager ::	인증(Authentication) 관리 객체, 로그인 요청 시 사용자 검증을 수행	
http.getSharedObject(AuthenticationManager.class)  
#### UserDetailsService	:: 사용자 정보를 불러오는 서비스 (loadUserByUsername())
http.getSharedObject(UserDetailsService.class)  
#### PasswordEncoder ::	비밀번호 암호화 객체 (BCryptPasswordEncoder) 
http.getSharedObject(PasswordEncoder.class)  


### AuthenticationFilter를 어떻게 생성하고 인증 처리 흐름은 어떻게 되는가?   
AuthenticationFilter는 요청을 가로채고, AuthenticationManager에게 인증을 위임합니다. AuthenticationManager는 AuthenticationProvider를 통해 인증을 처리합니다.  AuthenticationProvider는 내부적으로 사용자 정보(UserDetailsService)와 비밀번호 검증(PasswordEncoder)을 수행하여 인증합니다.  인증 성공 시, 인증된 사용자 정보를 SecurityContext에 저장합니다.  

**AuthenticationFilter → AuthenticationManager → AuthenticationProvider → UserDetailsService + PasswordEncoder의 흐름으로 인증이 진행됩니다.**

### AuthenticationConfiguration와 HttpSecurity에서 AuthenticationManager, UserDetailsService, PasswordEncoder를 둘 다 관리하는데 어느 객체에서 해당 객체를 참조해야할까?
AuthenticationManager, UserDetailsService, PasswordEncoder를 일반적으로 사용하려면 AuthenticationConfiguration을 사용하고 특정 필터 체인 내부에서만 필요할 경우 HttpSecurity.getSharedObject()를 사용할 수 있습니다.  

### AuthenticaitonProvider는 어디에서 관리하며 어떻게 등록할 수 있을까?
@Component를 사용하는 경우 자동으로 빈으로 등록하고 AuthenticationManager가 이를 감지하여 등록하도록 할 수 있고 SecurityConfig 내에서 수동으로 등록할 수도 있습니다. 그리고 AuthenticationManager에서 AuthenticationProvider들을 관리합니다.  
ex) .authenticationProvider(customAuthenticationProvider)  

## 인증 후 처리

### AuthenticationFilter에서 인증처리가 성공하면 따로 인증 영속성 처리를 해줘야하는가?
기본적으로는 Spring Security가 자동으로 인증 정보를 SecurityContextHolder 및 세션에 저장하므로, 별도의 인증 영속성 처리가 필요하지 않습니다.
하지만 세션 정책이나 커스텀 필터 동작 방식에 따라 추가적인 처리가 필요할 수도 있습니다.  

### CustomAuthenticationFilter를 만들게되면 스프링 시큐리티 6버전에서는 인증 유지를 자동으로 안해주고 저장하는 로직을 인증 필터 내에서 구현해야하는가?
Spring Security 6에서는 CustomAuthenticationFilter를 만들면 인증 정보를 자동으로 유지하지 않으며 SecurityContext를 저장하는 로직을 구현해야 합니다.
Spring Security 5에서는 SecurityContextPersistenceFilter가 자동으로 SecurityContext를 유지했지만, Spring Security 6에서는 requireExplicitSave(true)가 기본값으로 설정되어 자동 저장이 되지 않습니다.

### CustomAuthenticationFilter에서 인증 영속성 처리를 위한 로직을 구현해야할 때 어떻게 구현할 수 있는가?
CustomAuthenticationFilter의 successfulAuthentication에서 아래의 처리들을 해줘야한다.

        ✅ SecurityContext 생성 및 저장
        SecurityContext securityContext = SecurityContextHolder.createEmptyContext();
        securityContext.setAuthentication(authResult);
        SecurityContextHolder.setContext(securityContext);

        ✅ SecurityContextRepository를 사용하여 인증 정보 수동 저장
        securityContextRepository.saveContext(securityContext, request, response);

        ✅ 세션에 SecurityContext 저장 (중복 코드 제거 가능)
        HttpSession session = request.getSession();
        session.setAttribute(HttpSessionSecurityContextRepository.SPRING_SECURITY_CONTEXT_KEY, securityContext);

        ✅ 성공 핸들러 실행 (추가적인 로직을 처리)
        this.getSuccessHandler().onAuthenticationSuccess(request, response, authResult);


## OAuth2.0

OAuth 2.0 Cllient  
OAuth 2.0 ResourceServer  
OAuth 2.0 Authorization Server  
  
### OAuth 2.0
OAuth 2.0은 사용자의 자격 증명을 직접 공유하지 않고도, 제3자 애플리케이션(Client)이 특정 자원(Resource Server)에 안전하게 접근할 수 있도록 하는 인증 및 권한 부여 프로토콜  

### OAuth 2.0 Client
사용자의 인증을 요청하고, Access Token을 받아 API 요청을 수행하는 애플리케이션  
  
### OAuth 2.0 Resource Server
보호된 리소스를 제공하는 서버 (Access Token을 사용하여 접근 가능)  

### OAuth 2.0 Authorization Server
사용자를 인증하고 Access Token을 발급하는 서버  

### OAuth 2.0의 기본 흐름
OAuth Client → Authorization Server (로그인 요청)
OAuth Client ← Authorization Server (Access Token 발급)
OAuth Client → Resource Server (Access Token과 함께 요청)
Resource Server → OAuth Client (보호된 리소스 반환)

### OIDC란
클라이언트의 서비스를 사용자가 이용하기 위해서  
        •	사용자 정보를 필요로 하지 않고 단순히 접근 권한만 필요한 경우 → OAuth 2.0  
        •	사용자 정보(식별, 프로필 정보 등)가 필요한 경우 → OIDC (OAuth 2.0 + ID 토큰)  
        
### AccessToken에도 사용자정보가 담겨서 오는데 AccessToken으로 인증처리를 해도 되지 않을까?
AccessToken에 사용자의 일부 정보가 포함될 수도 있지만, 서버 측 인증 로직을 그 AccessToken에만 의존해서 구현하는 것은 일반적으로 권장되지 않습니다.  
Spring Security OAuth2 (또는 OIDC) 표준 방식을 따르면, 이미 **인증된 사용자 정보(Principal)**가 SecurityContext에 안전하게 관리되므로, 그대로 쓰면 됩니다.  
추가 정보가 필요하다면 OAuth2SuccessHandler나 UserService 등에서 DB 조회 후 세션에 저장해서 쓰는 식으로 확장하시면 됩니다.  
AccessToken 보다는 ID Token + UserInfo Endpoint를 통해 검증된 사용자 정보를 이용하는 편이 훨씬 간단하고 안전합니다.  

### AccessToken을 왜 사용하면 안될까?
AccessToken 내용이 항상 사용자 정보를 ‘충분히’ 담고 있지 않을 수 있습니다. 또한 공급자마다 토큰 형식, 키 회전(Key Rotation), 클레임 규칙이 달라서 유지보수 부담이 큼

### 세션에서 사용자 정보를 가져오는 방법 
1.	OAuth2 인증이 완료되면,
        •	Spring Security가 Authentication 객체를 만들어서 SecurityContext에 저장합니다.  	
        •	컨트롤러에서는 @AuthenticationPrincipal, 세션에 저장되어있는 SecurityContext(Authentication) 매개변수 등을 통해 사용자 정보(Principal, OAuth2User 등)에 접근할 수 있습니다.
3.	추가적으로 세션에 저장해야 할 정보가 있다면,
        •	OAuth2AuthenticationSuccessHandler(혹은 직접 정의한 CustomOAuth2SuccessHandler) 같은 인증 성공 후 실행되는 핸들러에서
        •	**인자로 전달되는 Authentication**에서 사용자 정보를 꺼내서, 필요한 추가 데이터를 세션(HttpSession)에 담아주면 됩니다.
