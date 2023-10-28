# Spring周边

## Spring Security

### 授权数据模型

主体（用户）在访问资源时，需要根据其权限判断用户对哪些资源有哪些操作许可



数据模型为：

- 主体（用户id、账号、密码、...） 
- 角色（角色id、角色名称、...）
- 权限（权限id、权限标识、资源路径、...） 
- 主体和角色**多对多**关系（用户id、角色id、...）
- 角色和权限**多对多**关系（角色id、权限id、...） 



访问控制流程：主体申请查询信息→判断主体是否有查询该信息的权限→继续或阻止



### 基于session认证

#### 基本流程

1. 客户端发送用户认证请求
2. 服务器接收请求并成功认证，将用户相关数据存放在session中
3. 服务器将session_id存放到cookie中，返回客户端
4. 后续客户端携带session_id访问资源
5. 服务器的权限拦截器根据用户访问路径，判断用户权限，若校验通过，则放行



#### 在springmvc中的使用

1. 依赖导入：

   ```xml
   <dependency>
       <groupId>org.springframework.security</groupId>
       <artifactId>spring-security-web</artifactId>
       <version>5.1.4.RELEASE</version>
   </dependency>
   
   <dependency>
       <groupId>org.springframework.security</groupId>
       <artifactId>spring-security-config</artifactId>
       <version>5.1.4.RELEASE</version>
   </dependency>
   ```

2. 添加配置类：

   ```java
   @EnableWebSecurity
   public class WebSecurityConfig extends WebSecurityConfigurerAdapter {
       //配置用户信息服务
       @Bean
       public UserDetailsService userDetailsService() {
           InMemoryUserDetailsManager manager = new InMemoryUserDetailsManager();
           // 添加用户
           manager.createUser(
               User.withUsername("zhangsan").password("123").authorities("p1").build()
           );
           manager.createUser(
               User.withUsername("lisi").password("456").authorities("p2").build()
           );
           return manager;
       }
       
       @Bean
       public PasswordEncoder passwordEncoder() {
       	return NoOpPasswordEncoder.getInstance();
       }
       
       //配置安全拦截机制
       @Override
       protected void configure(HttpSecurity http) throws Exception {
           http
               .authorizeRequests()
               .antMatchers("/r/r1").hasAuthority("p1") // 设置需要的权限
               .antMatchers("/r/r2").hasAuthority("p2") // 设置需要的权限
               .antMatchers("/r/**").authenticated() // 设置需要权限校验的路径
               .anyRequest().permitAll() // 其他路径完全开放
               .and()
               .formLogin() // 支持from表单登录
               .successForwardUrl("/login-success"); // 认证成功后跳转/login-success
       }
   }
   
   ```

3. 加载配置类，在servlet配置类的getRootConfigClasses方法中添加配置类：

   ```java
   @Override
   protected Class<?>[] getRootConfigClasses() {
   	return new Class<?>[] { ApplicationConfig.class, WebSecurityConfig.class};
   }
   ```

4. 启动项目，通过/login路径进入spring security自带的登录页面并完成认证

5. 请求指定资源，会根据权限拦截



#### 在springboot中使用

1. 依赖导入：

   ```xml
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-security</artifactId>
   </dependency>
   ```

2. 配置类：无需@EnableWebSecurity

   ```java
   @Configuration
   public class WebSecurityConfig extends WebSecurityConfigurerAdapter {
   	// 内容同mvc
   }
   ```

   

### 配置类

> 配置类中某些方法需要返回json字符串，本篇中使用fastjson库作演示，方便展示需要返回的信息，实际使用中可用jackson将统一响应对象转换为json

#### 基础授权配置

configure(HttpSecurity http)方法中：

```java
http.authorizeRequests()
    .antMatchers("/r/r1").hasAuthority("p2")
    .antMatchers("/r/r2").hasAuthority("p2")
    .antMatchers("/r/**").authenticated() // 所有/r/**的请求必须认证通过
    .anyRequest().permitAll() // 其它所有请求无需认证
```

> 使用链式调用，按 .url匹配方法1.权限控制方法1.url匹配方法2.权限控制方法2...的形式调用

url匹配方法：

- `antMatchers(HttpMethod.*, String regx, ...)`：匹配URL
  - 第一个参数：可以不传，若传递，表示限定请求方法。例：`HttpMethod.GET`
  - 后续参数：传递ant表达式字符串，匹配路径
- `regexMatchers(HttpMethod.*, String regexPattern, ...)`：匹配URL
  - 第一个参数：同上
  - 后续参数：传递正则表达式字符串，匹配路径
- `anyRequest()`：匹配任何请求



访问控制方法：

- `permitAll()`：所有人都可访问
- `anonymous()`：允许匿名访问，与permitAll类似，但会触发filter
- `denyAll()`：不允许任何人访问
- `authenticated()`：需要被认证才能访问
- `rememberMe()`：允许通过remember-me登录的用户访问
- `access()`：SpringEl表达式结果为true时可以访问
- `fullyAuthenticated()`：用户完全认证可以访问（非remember-me下自动登录）
- `hasRole(String)`：参数表示角色，具有该角色可以访问
- `hasAnyRole(多个String)`：参数表示角色，具有其中任何一个角色可以访问
- `hasAuthority(String)`：参数表示权限，具有该权限可以访问
- `hasAnyAuthority(多个String)`：参数表示权限，具有其中任何一个权限可以访问
- `hasIpAddress(String)`：参数表示IP地址，如果用户IP和参数匹配，则可以访问



使用注解简化：

> 首先在配置类上添加注解来启用功能：`@EnableGlobalMethodSecurity(prePostEnabled = true, securedEnabled = true)`

- `@Secured`：方法注解，在方法执行前检查角色
  - 注解的值：仅支持字符串数组，值会传给AccessDecisionManager处理，字符串为角色名字符串
  - 默认角色：
    - `ROLE_ANONYMOUS`：未登录或匿名用户的角色
    - `ROLE_LOGIN`：登录后自动授予用户该角色
    - `ROLE_USER`：表示已登录的普通用户的角色
    - `ROLE_ADMIN`：表示具有管理权限的用户
- `@PreAuthorize`：方法注解，在方法执行前对传入表达式求值，决定是否允许访问
  - 注解的值：支持SpEL表达式
  - SpEL表达式：一个字符串，如 `"hasAuthority('xxx')"`、`"returnObject == 'abc'"`（返回字符串为abc时）
- `@PostAuthorize`：方法注解，在方法执行后对表达式求值，决定是否允许返回结果
  - 注解的值：支持SpEL表达式



全局忽略URL，在配置类中的configure(WebSecurity web)方法中：

```java
web.ignoring()
    .antMatchers("/index.html")
    .antMatchers("/*.ico")
    .antMatchers("/image/**");
```



#### 从数据库中获取授权信息

> 进行下列配置后，基础授权配置所设置的配置仍然生效

配置类中：

```java
@AllArgsConstructor
@Configuration
public class SecurityConfig extends WebSecurityConfigurerAdapter {  
    
    private final CustomSecurityMetadataSource customSecurityMetadataSource;
    
    private final CustomAccessDecisionManager customAccessDecisionManager;
    
    @Override
    protected void configure(HttpSecurity http) throws Exception {

        http.authorizeRequests()
            .withObjectPostProcessor(new ObjectPostProcessor<FilterSecurityInterceptor>() {
                @Override
                public <O extends FilterSecurityInterceptor> O postProcess(O object) {
                    object.setSecurityMetadataSource(customSecurityMetadataSource);
                    object.setAccessDecisionManager(customAccessDecisionManager);
                    return object;
                }
            })
            .anyRequest().authenticated(); // 所有请求都需验证
    }
}
```

> withObjectPostProcessor：后处理配置对象，修改配置未完全公开的对象的属性，上例中修改的是FilterSecurityInterceptor



SecurityMetadataSource类：根据要访问的资源，获取其所需的角色

```java
@Component
public class CustomSecurityMetadataSource implements FilterInvocationSecurityMetadataSource {

    @Override
    public Collection<ConfigAttribute> getAttributes(Object object) throws IllegalArgumentException {
		// 获取请求路径，并去除参数
        String url = ((FilterInvocation) object).getRequestUrl(); // 带参数的路径
        int index = url.indexOf('?');
        if (index > 0) {
            url = url.substring(0, index);
        }
        
        // ...

        // 返回该 URL 所需要的角色集合
        return SecurityConfig.createList("role1"); // 根据字符串创建ConfigAttribute集合，也可以传入字符串数组进行创建
        // return SecurityConfig.createList(); // 无需验证，不会进入AccessDecisionManager，直接放行
    }

    @Override
    public Collection<ConfigAttribute> getAllConfigAttributes() {
        // 返回所有的 ConfigAttribute 集合，会传递给 AccessDecisionManager 进行权限判断
        return new ArrayList<>(); // 返回空集合或null，表示不需要校验
    }

    @Override
    public boolean supports(Class<?> clazz) {
        // 返回 true 表示支持过滤器拦截请求
        return true;
    }
}
```



AccessDecisionManager类：判断用户是否有权访问该资源

```java
@Component
public class CustomAccessDecisionManager implements AccessDecisionManager {

    @Override
    public void decide(Authentication authentication, Object object,
            Collection<ConfigAttribute> configAttributes) throws AccessDeniedException,
            InsufficientAuthenticationException {
        // 进行权限校验，若校验不通过，需手动抛出异常
        // configAttributes为所需权限，即SecurityMetadataSource的getAttributes方法的返回值

        // 获取用户具有的角色集合
        Collection<? extends GrantedAuthority> authorities = auth.getAuthorities();
    }

    @Override
    public boolean supports(ConfigAttribute attribute) {
        // 返回true表示可以处理传入的 ConfigAttribute
        return true;
    }

    @Override
    public boolean supports(Class<?> clazz) {
        // 返回true表示可以处理传入的类型
        return true;
    }
}
```



#### 从数据库中获取用户信息

UserDetailsService类：用于查询用户信息，返回一个数据对象实现了UserDetails接口的对象

```java
@Service
public class CustomUserDetailsServiceImpl implements UserDetailsService {

    // 查询用户的服务类
    @Autowired
    private UserService userService;

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        // 查询用户
        User user = userService.findByUsername(username);
        
        if (user == null) {
            throw new UsernameNotFoundException("User not found");
        }
        
        UserDTO userDTO = new UserDTO();
        BeanUtils.copyProperties(user, userDTO);
        
        userDTO.setRoleList(...)
        
        return userDTO;
    }
}
```



实现了UserDetails接口的DTO对象：

```java
@Data
public class userDTO implements UserDetails {

    // 属性略

    // 获取用户的角色集合
    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        List<SimpleGrantedAuthority> authorities = new ArrayList<>();
        authorities.add(new SimpleGrantedAuthority("角色名1"));
        authorities.add(new SimpleGrantedAuthority("角色名2"));
        return authorities;
    }


    @Override
    public String getPassword() {
        // 获取密码
    }


    @Override
    public String getUsername() {
        // 获取账号
    }

    @Override
    public boolean isAccountNonExpired() {
        // 当前账号是否未过期
        return true;
    }

    @Override
    public boolean isAccountNonLocked() {
        // 当前账号是否未被锁定
        return locked == null || !locked;
    }

    @Override
    public boolean isCredentialsNonExpired() {
		// 当前账号证书（密码）是否未过期
        return true;
    }

    @Override
    public boolean isEnabled() {
        // 当前账号是否可用
        return deleted == null || !deleted;
    }
}

```



DaoAuthenticationProvider类：用于进行登录时的认证

```java
@Component
public class CustomAuthenticationProvider extends DaoAuthenticationProvider {

    public MyAuthenticationProvider(PasswordEncoder passwordEncoder,
                                    @Qualifier("myUserDetailsServiceImpl") UserDetailsService userDetailsService) {
        setPasswordEncoder(passwordEncoder);
        setUserDetailsService(userDetailsService);
        setHideUserNotFoundExceptions(false); // 区分用户不存在和密码错误异常
    }
    
    @Override
    public Authentication authenticate(Authentication auth) throws AuthenticationException {
        try {
            // 进行用户认证
            return super.authenticate(authentication);

        } catch (UsernameNotFoundException e) {
            // 用户不存在
        } catch (BadCredentialsException e) {
            // 密码错误
        } catch (AuthenticationException e) {
            // 其他异常，向上抛出
            throw e;
        }
    }
}
```



配置类中注入DaoAuthenticationProvider：

```java
@AllArgsConstructor
@Configuration
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    private CustomAuthenticationProvider customAuthenticationProvider;

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.authenticationProvider(customAuthenticationProvider);
    }

}
```

passwordEncoder的装配见[通用Bean处理](#通用Bean处理)



#### 自定义异常处理

configure(HttpSecurity http)方法中：

```java
http.exceptionHandling()
    .accessDeniedHandler(403异常处理器实例)
    .authenticationEntryPoint(未登录异常处理函数式接口)
```



403异常处理器：仅适用于已登录用户

```java
@Component
public class CustomAccessDeniedHandler implements AccessDeniedHandler {
    @Override
    public void handle(HttpServletRequest request, HttpServletResponse response, AccessDeniedException e) throws IOException, ServletException {
        
        response.setStatus(HttpStatus.FORBIDDEN.value());
        response.setHeader("Content-Type","application/json;charset=utf-8");

        JSONObject json = new JSONObject();
        json.put("code", 403);
        json.put("msg","权限不足，无法访问!");

        PrintWriter out = response.getWriter();
		out.write(json.toString());
        out.flush();
        out.close();
    }
}
```



未登录异常处理函数式接口：需要权限的资源被未登录用户访问时的抛出的异常，在不设置该接口时，默认会重定向到登录页面

```java
authenticationEntryPoint((request, response, authException) -> {
    response.setStatus(HttpStatus.UNAUTHORIZED.value());
    response.setHeader("Content-Type", "application/json;charset=utf-8");

    JSONObject json = new JSONObject();
    json.put("code", 401);
    json.put("msg", "请先登录");

    PrintWriter out = response.getWriter();
    out.write(json.toString());
    out.flush();
    out.close();
})
```



#### 用户登录

> **调用登录url必须以表单方式提交**

configure(HttpSecurity http)方法中：

```java
http.formLogin()
    .loginProcessingUrl("/login") // 登录url，方法为post
    .usernameParameter("username") // 自定义用户名字段名
    .passwordParameter("password") // 自定义密码字段名
    .successHandler(登录成功处理器实例) // 登录成功处理器
    .failureHandler(登录失败处理器实例) // 登录失败处理器
    .loginPage("/login").permitAll(); // 允许所有用户访问登录页面，可不指定该行
```



登录成功处理器：

```java
@Component
public class CustomAuthenticationSuccessHandler implements AuthenticationSuccessHandler {

    @Override
    public void onAuthenticationSuccess(HttpServletRequest request,HttpServletResponse response,Authentication authentication) throws IOException, ServletException {
        
        response.setStatus(HttpStatus.OK.value());
        response.setHeader("Content-Type","application/json;charset=utf-8");

        JSONObject json = new JSONObject();
        json.put("code", 200);
        json.put("message", "登录成功");
        json.put("data", authentication);

        PrintWriter out = response.getWriter();
        out.write(json.toString());
        out.flush();
        out.close();
    }
}
```



登录失败处理器：

```java
@Component
public class CustomAuthenticationFailureHandler implements AuthenticationFailureHandler {
 
    @Override
    public void onAuthenticationFailure(HttpServletRequest request, HttpServletResponse response, AuthenticationException e) throws IOException, ServletException {

        response.setStatus(HttpStatus.UNAUTHORIZED.value());
        response.setHeader("Content-Type","application/json;charset=utf-8");

        JSONObject json = new JSONObject();
        json.put("code", 401);
        json.put("message", "登录失败");
        json.put("error", e.getMessage());
 
        PrintWriter out = response.getWriter();
        out.write(json.toString());
        out.flush();
        out.close();
    }
}
```



#### 用户登出

configure(HttpSecurity http)方法中：

```java
http.logout()
    .logoutUrl("/logout") // 触发登出操作的url
    .logoutSuccessUrl("/login?logout") // 登出成功跳转的url
    .logoutSuccessHandler(成功登出处理器实例) // 成功登出处理器，有该项则忽略logoutSuccessUrl
    .addLogoutHandler(登出处理器实例) // 登出处理器
    .clearAuthentication(true) // 登出后清除认证状态
    .invalidateHttpSession(true) // 登出后销毁session
	.deleteCookies("JSESSIONID"); // 登出后删除cookie中的指定字段
```

> 登出处理器 会被调用来执行登出逻辑，然后再调用 成功登出处理器 来处理成功登出后的响应



成功登出处理器：

```java
@Component
public class CustomLogoutSuccessHandler implements LogoutSuccessHandler {
    @Override
    public void onLogoutSuccess(HttpServletRequest request, HttpServletResponse response, Authentication authentication) throws IOException {

        response.setStatus(HttpStatus.OK.value());
        response.setContentType("application/json;charset=utf-8");
        
        JSONObject json = new JSONObject();
        json.put("status", 200);
        json.put("msg","登出成功");

        PrintWriter out = response.getWriter();
        out.write(json.toString());
        out.flush();
        out.close();
    }
}
```



登出处理器：不得抛出异常

```java
@Component
public class CustomLogoutHandler implements LogoutHandler {
    @Override
    public void logout(HttpServletRequest request, HttpServletResponse response, Authentication authentication) {
       // 在这里执行自定义逻辑，比如清除缓存、记录日志等。
       System.out.println("用户" + authentication.getName() + "已经退出登录");
    }
}
```



#### session控制

configure(HttpSecurity http)方法中：

```java
http.sessionManagement()
    .sessionRegistry(sessionRegistry实例) // 自定义sessionRegistry
    .invalidSessionUrl("/session/invalid") // session失效后跳转路径
    .invalidSessionStrategy(函数式接口) // 自定义session失效处理，若指定该项，会使invalidSessionUrl失效
    .sessionCreationPolicy(SessionCreationPolicy.IF_REQUIRED) // session生成策略
    .maximumSessions(1) // 同一用户同时在线的最大session数（如在不同端登录该用户），设为-1表示无限制
    .maxSessionsPreventsLogin(false) // 达到最大session数后，是否保留已经登录的用户。为true，新用户无法登录；为 false，旧用户被踢出
    .expiredUrl("/xxx") // 用户被提出时重定向的url
    .expiredSessionStrategy(用户踢出处理器实例) // 用户踢出处理，会使expiredUrl失效
```



自定义session失效处理：需要一个函数式接口，通常在session超时时触发

```java
.invalidSessionStrategy((request, response) -> {
    response.setStatus(HttpStatus.UNAUTHORIZED.value());
    response.setHeader("Content-Type", "application/json;charset=utf-8");

    JSONObject json = new JSONObject();
    json.put("code", 401);
    json.put("msg", "登录信息过期");

    PrintWriter out = response.getWriter();
    out.write(json.toString());
    out.flush();
    out.close();
};
```



session生成策略，可为sessionCreationPolicy方法传入SessionCreationPolicy的枚举值：

- `ALWAYS`：若没有session则创建一个
- `NEVER`：springsecurity不创建session，会从应用中引用
- `IF_REQUIRED`：默认值，在需要时创建一个session
- `STATELESS`：不创建也不使用session



用户踢出处理器：

```java
@Component
public class MySessionExpiredStrategy implements SessionInformationExpiredStrategy {

    @Override
    public void onExpiredSessionDetected(SessionInformationExpiredEvent sessionInformationExpiredEvent) throws IOException {
        response.setStatus(HttpStatus.UNAUTHORIZED.value());
        response.setHeader("Content-Type", "application/json;charset=utf-8");

        JSONObject json = new JSONObject();
        json.put("code", 401);
        json.put("msg", "您的账号已在别的地方登录");

        PrintWriter out = response.getWriter();
        out.write(json.toString());
        out.flush();
        out.close();
    }
}
```



自定义sessionRegistry：

```java
public class MySessionRegistryImpl implements SessionRegistry {
	// ...
}
```



#### 自定义过滤器

configure(HttpSecurity http)方法中：

```java
http
    .addFilterBefore(过滤器实例, xx过滤器.class); // 在xx过滤器前添加自定义过滤器
	.addFilterAfter(过滤器实例, xx过滤器.class); // 在xx过滤器后添加自定义过滤器 
```



内置过滤器：过滤器顺序从上到下

| 别名                         | 类名称                                                | Namespace Element or Attribute                               |
| :--------------------------- | :---------------------------------------------------- | :----------------------------------------------------------- |
| CHANNEL_FILTER               | ChannelProcessingFilter                               | http/intercept-url[@requires](https://github.com/requires)-channel |
| SECURITY_CONTEXT_FILTER      | SecurityContextPersistenceFilter                      | http                                                         |
| CONCURRENT_SESSION_FILTER    | ConcurrentSessionFilter                               | session-management/concurrency-control                       |
| HEADERS_FILTER               | HeaderWriterFilter                                    | http/headers                                                 |
| CSRF_FILTER                  | CsrfFilter                                            | http/csrf                                                    |
| LOGOUT_FILTER                | LogoutFilter                                          | http/logout                                                  |
| X509_FILTER                  | X509AuthenticationFilter                              | http/x509                                                    |
| PRE_AUTH_FILTER              | AbstractPreAuthenticatedProcessingFilter( Subclasses) | N/A                                                          |
| CAS_FILTER                   | CasAuthenticationFilter                               | N/A                                                          |
| FORM_LOGIN_FILTER            | UsernamePasswordAuthenticationFilter                  | http/form-login                                              |
| BASIC_AUTH_FILTER            | BasicAuthenticationFilter                             | http/http-basic                                              |
| SERVLET_API_SUPPORT_FILTER   | SecurityContextHolderAwareRequestFilter               | http/@servlet-api-provision                                  |
| JAAS_API_SUPPORT_FILTER      | JaasApiIntegrationFilter                              | http/@jaas-api-provision                                     |
| REMEMBER_ME_FILTER           | RememberMeAuthenticationFilter                        | http/remember-me                                             |
| ANONYMOUS_FILTER             | AnonymousAuthenticationFilter                         | http/anonymous                                               |
| SESSION_MANAGEMENT_FILTER    | SessionManagementFilter                               | session-management                                           |
| EXCEPTION_TRANSLATION_FILTER | ExceptionTranslationFilter                            | http                                                         |
| FILTER_SECURITY_INTERCEPTOR  | FilterSecurityInterceptor                             | http                                                         |
| SWITCH_USER_FILTER           | SwitchUserFilter                                      | N/A                                                          |





自定义过滤器实例：

```java
@Component
public class ImageCodeFilter extends OncePerRequestFilter {
	@Override
    protected void doFilterInternal(HttpServletRequest request,
                                    HttpServletResponse response,
                                    FilterChain filterChain) throws ServletException, IOException {

        try {
            // 自定义验证
        } catch (XXException e) {
            authenticationFailureHandler.onAuthenticationFailure(request, response, e); // 认证失败，交由该handler处理，需要通过依赖注入获取其实例
            return;
        }

        filterChain.doFilter(request, response); // 验证通过，转到下一个过滤器
    }
}
```



#### 通用Bean处理

```java
@Configuration
public class SecurityBeanConfig {

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }

}
```



#### CSRF

configure(HttpSecurity http)方法中：

```java
http.csrf().disable(); // 关闭CSRF跨域保护
```



#### 匿名用户

configure(HttpSecurity http)方法中：

```java
http.anonymous()
    .principal("anonymousUser") // 设置匿名用户名
    .authorities("ROLE_ANONYMOUS"); // 设置匿名用户的角色
```



### token认证

1. 引入JWT依赖

   ```xml
   <dependency>
       <groupId>com.nimbusds</groupId>
       <artifactId>nimbus-jose-jwt</artifactId>
       <version>9.30.2</version>
   </dependency>
   ```

2. Token工具类

   ```java
   @Component
   public class TokenUtils {
   
       /**
        * 创建秘钥
        */
       private final byte[] SECRET = "6MNSobBABCDIO0fS6MNSobBRCHGIO0fS".getBytes();
   
       /**
        * 过期时间：毫秒数
        */
       @Value("${token.timeout}")
       private long EXPIRE_TIME;
   
       /**
        * 生成Token
        */
       public String buildJWT(String account) {
           try {
               // 创建一个32-byte的密匙
               MACSigner macSigner = new MACSigner(SECRET);
               // 建立payload 载体
               JWTClaimsSet claimsSet = new JWTClaimsSet.Builder()
                       .expirationTime(new Date(System.currentTimeMillis() + EXPIRE_TIME))
                       .claim("ACCOUNT", account)
                       .build();
   
               // 建立签名
               SignedJWT signedJWT = new SignedJWT(new JWSHeader(JWSAlgorithm.HS256), claimsSet);
               signedJWT.sign(macSigner);
   
               // 生成token
               return signedJWT.serialize();
   
           } catch (JOSEException e) {
               e.printStackTrace();
           }
           return null;
       }
   
       /**
        * 校验token
        */
       public String validToken(String token) throws ResultException {
           try {
               if (token == null) {
                   throw ResultException.of(10001, "Token 无效");
               }
               SignedJWT jwt = SignedJWT.parse(token);
               JWSVerifier verifier = new MACVerifier(SECRET);
               //校验是否有效
               if (!jwt.verify(verifier)) {
                   throw ResultException.of(10001, "Token 无效");
               }
   
               //校验超时
               Date expirationTime = jwt.getJWTClaimsSet().getExpirationTime();
               if (new Date().after(expirationTime)) {
                   throw ResultException.of(10002, "Token 已过期");
               }
   
               //获取载体中的数据
               Object account = jwt.getJWTClaimsSet().getClaim("ACCOUNT");
               //是否有openUid
               if (Objects.isNull(account)){
                   throw ResultException.of(10003, "账号为空");
               }
               return account.toString();
           } catch (ParseException | JOSEException e) {
               throw ResultException.of(10001, "Token 无效");
           }
       }
   }
   
   ```

3. Token认证拦截器

   ```java
   @AllArgsConstructor
   public class SandAuthenticationTokenFilter extends UsernamePasswordAuthenticationFilter {
   
       private Environment environment;
   
       private ObjectMapper objectMapper;
   
       private TokenUtils tokenUtils;
   
       private UserDetailsService userDetailsService;
   
       @Override
       public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
           String tokenHeader = environment.getProperty("token.header", "");
   
           HttpServletRequest httpRequest = (HttpServletRequest) request;
           HttpServletResponse httpResponse = (HttpServletResponse) response;
   
           String path = httpRequest.getRequestURI();
   
           if (path.matches(".*login$")) {
               chain.doFilter(request, response);
               return;
           }
   
           try {
               String authToken = httpRequest.getHeader(tokenHeader);
   
               if (authToken == null) {
                   errorHandler(httpResponse, ResultException.of(10000, "请先登陆"));
               }
   
               String username = tokenUtils.validToken(authToken);
   
               if (username != null) {
                   // 认证信息未写入时
                   if (SecurityContextHolder.getContext().getAuthentication() == null) {
                       UserDetails userDetails = userDetailsService.loadUserByUsername(username);
                       UsernamePasswordAuthenticationToken authentication = new UsernamePasswordAuthenticationToken(userDetails, null, userDetails.getAuthorities());
                       authentication.setDetails(new WebAuthenticationDetailsSource().buildDetails(httpRequest));
                       // 将权限写入本次会话
                       SecurityContextHolder.getContext().setAuthentication(authentication);
                   }
                   chain.doFilter(request, response);
               	return;
               }
   
           } catch (ResultException e) {
               // token认证失败的场景
               errorHandler(httpResponse, e);
               httpResponse.setStatus(HttpStatus.UNAUTHORIZED.value());
               httpResponse.setHeader("Content-Type", "application/json;charset=utf-8");
   
               PrintWriter out = httpResponse.getWriter();
               out.write(objectMapper.writeValueAsString(ResponseVO.error(e)));
               out.flush();
               out.close();
           }
       }
   
       private void errorHandler(HttpServletResponse httpResponse, ResultException e) throws IOException {
           httpResponse.setStatus(HttpStatus.UNAUTHORIZED.value());
           httpResponse.setHeader("Content-Type", "application/json;charset=utf-8");
   
           PrintWriter out = httpResponse.getWriter();
           out.write(objectMapper.writeValueAsString(ResponseVO.error(e)));
           out.flush();
           out.close();
       }
   }
   ```

4. 修改SpringSecurity的配置

   ```java
   http.sessionManagement()
   	.sessionCreationPolicy(SessionCreationPolicy.STATELESS); // 关闭session
   
   http.addFilterBefore(
   	new SandAuthenticationTokenFilter(environment, objectMapper, tokenUtils, userDetailsService),
   UsernamePasswordAuthenticationFilter.class
   ); // token验证
   ```

   > 注：过滤器不能交由spring控制，否则会使ignore中的要忽略的接口配置无效



### controller获取用户信息

#### 注入Principal获取登录时候的用户名

假设我们提交登录的时候用户名使用是用户名与密码，那么我们AuthenticationToken的Principal中存储的是用户名字段，在默认逻辑下，登录验证后的Authentication字段会与登录请求时候保持一致，那么Authentication中的Principal便是登录的用户名。下面一段Controller的代码便是通过Spring的注入机制获取上下文中的Principal对象。

```java
public UserModel findUser(Principal principal)  {
    UserModel userModel = userService.findOneByUsername(principal.getName());
    return userModel;
}
```

**注意：此处也可注入Authentication类，它是Principle的子类，属性更为详细**

#### 注入UserDetails获取数据中的UserDetails特有信息

这种场景通常是因为登录时候使用的的登录标识信息与我们需要使用的信息不一致，最常见的场景是，系统支持使用手机号和用户名进行登录，但是数据库中的查询逻辑我们只想支持用户表中的用户名。如果从Principal获取，我们无法判断存储的是手机号还是用户名。那么我们便可以通过注入UserDetails对象，来获取Authentication的details字段中UserDetails信息。

```java
public UserModel findUser(UserDetails userDetails)  {
    UserModel userModel = userService.findOneByUsername(userDetails.getUsername());
    return userModel;
}
```

#### 注入Authentication获取更多的信息

如果Principal和UserDetails中的用户身份信息都不足以满足当前业务使用从场景，比如你需要验证当时存储在details的一些自定义结构信息，那么我们可以通过在Controller层注入Authentication直接操作当前上下文的中Authentication对象。 当我们明白了如何在Controller中操作Authentication之后，我们为了进行偷懒也可以ControllerAdvice中对Authentication进行拦截转型，将明确类型的User实现类实例放置于上下中，以便Controller更容易的进行注入:

```java
@ControllerAdvice
public class CurrentUserAdvice {
    @ModelAttribute()
    public JwtUser currentUser(Authentication authentication) {
        JwtUser jwtUser = null;
        if(authentication!=null) {
             jwtUser = (JwtUser)authentication.getDetails();
        }
        return jwtUser;
    }
}
```

那么我们在Controller只需要通过@ModelAttribute去注入。

```less
    public UserModel findUser(@ModelAttribute() JwtUser JwtUser)  {
        UserModel userModel = userService.findOneById(JwtUser.getId());
        return userModel;
    }
```



作者：废柴大叔阿基拉
链接：https://juejin.cn/post/6844904142037581831
来源：稀土掘金
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

