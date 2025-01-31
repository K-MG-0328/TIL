
## AuthenticationConfiguration은 언제 생성되는가?
별도로 @Bean 등록할 필요 없이 Spring Security가 내부적으로 설정합니다.
## AuthenticationConfiguration은 어디에서 설정할 수 있는가?
AuthenticationConfiguration 자체는 Spring Security 내부에서 관리되므로, 직접 설정할 필요는 없습니다.

## HttpSecurity가 관리하는 공유 객체
### AuthenticationManager	인증(Authentication) 관리 객체, 로그인 요청 시 사용자 검증을 수행	
http.getSharedObject(AuthenticationManager.class)  
### UserDetailsService	사용자 정보를 불러오는 서비스 (loadUserByUsername())
http.getSharedObject(UserDetailsService.class)  
### PasswordEncoder	비밀번호 암호화 객체 (BCryptPasswordEncoder) 
http.getSharedObject(PasswordEncoder.class)  


## AuthenticationFilter를 어떻게 생성하고 인증 처리 흐름은 어떻게 되는가?  
AuthenticationFilter는 요청을 가로채고 인증을 AuthenticationManager에 위임하므로  AuthenticationFilter에 AuthenticationManager를 생성한 후 필터에 주입해야합니다.  
이 후에 AuthenticationManager는 AuthenticationProvider에게 인증을 위임 후 실제 인증 처리를 진행합니다.  
  
AuthenticationFilter는 요청을 가로채고, AuthenticationManager에게 인증을 위임합니다.  
AuthenticationManager는 AuthenticationProvider를 통해 인증을 처리합니다.  
AuthenticationProvider는 내부적으로 사용자 정보(UserDetailsService)와 비밀번호 검증(PasswordEncoder)을 수행하여 인증합니다.  
인증 성공 시, 인증된 사용자 정보를 SecurityContext에 저장합니다.  

**AuthenticationFilter → AuthenticationManager → AuthenticationProvider → UserDetailsService + PasswordEncoder의 흐름으로 인증이 진행됩니다.**


## AuthenticationConfiguration와 HttpSecurity에서 AuthenticationManager, UserDetailsService, PasswordEncoder를 둘 다 관리하는데 어느 객체에서 해당 객체를 참조해야할까?
AuthenticationManager, UserDetailsService, PasswordEncoder를 일반적으로 사용하려면 AuthenticationConfiguration을 사용하고  
특정 필터 체인 내부에서만 필요할 경우 HttpSecurity.getSharedObject()를 사용할 수 있습니다.  

## AuthenticaitonProvider는 어디에서 관리하며 어떻게 등록할 수 있을까?
@Component를 사용하는 경우 자동으로 빈으로 등록하고 AuthenticationManager가 이를 감지하여 등록하도록 할 수 있고 SecurityConfig 내에서 수동으로 등록할 수도 있습니다. 그리고 AuthenticationManager에서 AuthenticationProvider들을 관리합니다.  
ex) .authenticationProvider(customAuthenticationProvider)  
