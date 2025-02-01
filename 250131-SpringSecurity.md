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

