# Spring周边

## Spring Security

### 授权数据模型

主体（用户）在访问资源时，需要根据其权限判断用户对哪些资源有哪些操作许可



数据模型为：

- 主体（用户id、账号、密码、...） 
- 角色（角色id、角色名称、...）
- 权限（权限id、权限标识、权限名称、...） 
- 资源（资源路径）
- 主体和角色**多对多**关系（用户id、角色id、...）
- 角色和权限**多对多**关系（角色id、权限id、...） 
- 权限和资源**多对一**关系（权限id、资源id）



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

- `antMatchers(HttpMethod.*, String regx)`：匹配URL
  - 第一个参数：可以不传，若传递，表示限定请求方法。例：`HttpMethod.GET`
  - 第二个参数：传递ant表达式字符串，匹配路径
- `regexMatchers(HttpMethod.*, String regexPattern)`：匹配URL
  - 第一个参数：同上
  - 第二个参数：传递正则表达式字符串，匹配路径
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



#### 从数据库中获取授权信息

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
            });
    }
}
```

> withObjectPostProcessor：后处理配置对象，修改配置未完全公开的对象的属性，上例中修改的是FilterSecurityInterceptor



SecurityMetadataSource类：根据要访问的资源，获取其所需的权限

```java
public class CustomSecurityMetadataSource implements FilterInvocationSecurityMetadataSource {

    @Override
    public Collection<ConfigAttribute> getAttributes(Object object) throws IllegalArgumentException {

        String url = ((FilterInvocation) object).getRequestUrl(); // 获取请求的 URL
        
        // ...

        // 返回该 URL 所需要的权限集合
        return SecurityConfig.createList("role1"); // 根据字符串创建ConfigAttribute集合
        // return SecurityConfig.createList("ROLE_ANONYMOUS") // 无需登录
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
public class CustomAccessDecisionManager implements AccessDecisionManager {

    @Override
    public void decide(Authentication authentication, Object object,
            Collection<ConfigAttribute> configAttributes) throws AccessDeniedException,
            InsufficientAuthenticationException {
        // 进行权限校验，若校验不通过，需手动抛出异常
        // configAttributes为所需权限

        // 获取用户角色集合
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



DaoAuthenticationProvider类：用于进行具体认证

```java
@Component
public class CustomAuthenticationProvider extends DaoAuthenticationProvider {

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
    
    @Qualifier("customUserDetailsServiceImpl")
    private UserDetailsService userDetailsService;
    
    private PasswordEncoder passwordEncoder;

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        customAuthenticationProvider.setPasswordEncoder(passwordEncoder);
        customAuthenticationProvider.setUserDetailsService(userDetailsService);
        auth.authenticationProvider(customAuthenticationProvider);
    }

}
```

passwordEncoder的装配见[通用Bean处理](#通用Bean处理)



#### 自定义异常处理

configure(HttpSecurity http)方法中：

```java
http.exceptionHandling()
    .accessDeniedHandler(异常处理器实例) // 403异常处理器
```



403异常处理器：

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



#### 用户登录

configure(HttpSecurity http)方法中：

```java
http.formLogin()
    .loginProcessingUrl("/login") // 登录url，方法为post
    .usernameParameter("username") // 自定义用户名字段名
    .passwordParameter("password") // 自定义密码字段名
    .successHandler(登录成功处理器实例) // 登录成功处理器
    .failureHandler(登录失败处理器实例) // 登录失败处理器
    .permitAll(); // 允许所有用户访问登录页面
```



登录成功处理器：

```java
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
    .invalidateHttpSession(true); // 登出后销毁session
```

> 登出处理器 会被调用来执行登出逻辑，然后再调用 成功登出处理器 来处理成功登出后的响应



成功登出处理器：

```java
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
public class CustomLogoutHandler implements LogoutHandler {
    @Override
    public void logout(HttpServletRequest request, HttpServletResponse response, Authentication authentication) {
       // 在这里执行自定义逻辑，比如清除缓存、记录日志等。
       System.out.println("用户" + authentication.getName() + "已经退出登录");
    }
}
```



#### session控制



#### 验证码



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



#### 1





配置类configure方法的http参数的常用方法：

```java
// 添加过滤器到用户名密码校验前（一般为验证码校验）
http.addFilterBefore(filter,UsernamePasswordAuthenticationFilter.class);

// 授权配置，笼统的配置应放在后面
http.authorizeRequests()
    // antMatchers：传递ant表达式，用于匹配路径
    // regexMatchers：传递正则字符串，用于匹配路径
    // hasAuthority
    .antMatchers("/r/r1").hasAuthority("p2")
    .antMatchers("/r/r2").hasAuthority("p2")
    .antMatchers("/r/**").authenticated() // 所有/r/**的请求必须认证通过
    // anyRequest：表示所有请求
    // permitAll：所有人都可访问
    .anyRequest().permitAll() // 其它所有请求无需认证
    
    .withObjectPostProcessor(new ObjectPostProcessor<FilterSecurityInterceptor>() {
        @Override
        public <O extends FilterSecurityInterceptor> O postProcess(O object) {
            object.setSecurityMetadataSource(cmmpFilterInvocationSecurityMetadataSource);
            object.setAccessDecisionManager(cmmpAccessDecisionManager);
            return object;
        }
    });



// 未登录时返回json，给前端判断
// session失效时返回json，给前端判断
http.sessionManagement()
    /*
    控制session生成策略，可选枚举值有：
      ALWAYS		- 若没有session则创建一个
      NEVER			- springsecurity不创建session，会从应用中引用
      IF_REQUIRED	- 默认值，在需要时创建一个session
      STATELESS		- 不创建也不使用session
     */
    .sessionCreationPolicy(SessionCreationPolicy.IF_REQUIRED)
    .maximumSessions(-1)
    .sessionRegistry(sessionRegistry);

// session超时的处理
http.sessionManagement()
    .invalidSessionStrategy(
    ((request, response) ->
     this.errorResponse(
         request,
         response,
         "登录信息过期",
         "登录信息过期，请重新登录。")));

// 未登录时的处理
http.exceptionHandling()
    .authenticationEntryPoint(
    ((request, response, authException) ->
     this.errorResponse(
         request,
         response,
         "未登录",
         "您未登录，请先登录。")));
```





