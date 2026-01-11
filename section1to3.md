# Section-3
# 11 What is Security?
![alt text](image-1.png)![alt text](image-2.png)![alt text](image-3.png)![alt text](image-4.png)![alt text](image-5.png)![alt text](image-6.png)![alt text](image-7.png)
# 12. The key Security Components?
![alt text](image-8.png)![alt text](image-9.png)![alt text](image-10.png)
# 13 Spring Security
![alt text](image-11.png)![alt text](image-12.png)
# 14. Spring Security in Action:
![alt text](image-13.png)![alt text](image-14.png)![alt text](image-15.png)![alt text](image-16.png)![alt text](image-17.png)
## HelloController.java
```java
package com.bharath.spirng.security;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HelloController {

	@GetMapping("/hello")
	public String hello() {
		
		return "Spring Security Rocks!";
	}
}
```
# 15. Resending Basic Authentication Details
![alt text](image-18.png)![alt text](image-19.png)
# 16. Config before and after 3.0
![alt text](image-20.png)![alt text](image-21.png)
# 17. Update 3.2 and higher
![alt text](image-22.png)![alt text](image-23.png)
#  18) Create Custom Security Configuration
![alt text](image-24.png)
### Config class
```java
package com.bharath.spirng.security;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.Customizer;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.web.SecurityFilterChain;

@Configuration
public class MySecurityConfig {

	//1. Create SecurityFilterChain Bean
	  //customize our security via HttpSecurity
	
	@Bean
	SecurityFilterChain filterChain(HttpSecurity httpSecurity) {
		
		//Need to configure HTTP Basic Authentication		
		httpSecurity.httpBasic(Customizer.withDefaults()); 
		
		//Customize this authentication for single (particular or few) request or for all request
		  // here all request should be authenticated.
		httpSecurity.authorizeHttpRequests(authorize -> authorize.anyRequest().authenticated());
		
		return httpSecurity.build();		
	}
}
```
# 19) Custom User Details Service
![alt text](image-25.png)
```java
package com.bharath.spirng.security;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.Customizer;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.core.userdetails.User;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.provisioning.InMemoryUserDetailsManager;
import org.springframework.security.web.SecurityFilterChain;

@Configuration
public class MySecurityConfig {

	
	//1. Expose another bean for UserDetailsService - This bean is automatically used in Security Chain		
	@Bean
	UserDetailsService userDetailsService() {
		
		//Currently we use InMemory DB
		  //This is implementation of UserDetailsService Interface.
		InMemoryUserDetailsManager userDetailsService = new InMemoryUserDetailsManager();
		
		// Here we configure user we want
		   //Every user have authority (role)
//		UserDetails user = User.withUsername("tom").password("cruise").authorities("read").build();
		
		//3
		UserDetails user = User.withUsername("tom")
				        .password(passwordEncoder().encode("cruise"))
				        .authorities("read").build();
		
		//Put this user in UserDeatailsManager
		userDetailsService.createUser(user);
		
		return userDetailsService;		
	}
	
	//2. Create BCryptPasswordEncoder for encoding the password
	@Bean
	BCryptPasswordEncoder passwordEncoder() {
		return new BCryptPasswordEncoder();
	}
	
	@Bean
	SecurityFilterChain filterChain(HttpSecurity httpSecurity) {
			
		httpSecurity.httpBasic(Customizer.withDefaults()); 
	
		httpSecurity.authorizeHttpRequests(authorize -> authorize.anyRequest().authenticated());
		
		return httpSecurity.build();		
	}
}
```
# 20 Test
![alt text](image-26.png)
# # 21) Create Custom Authentication Provider
![alt text](image-27.png)
### MyAuthenticationProvider
```java
package com.bharath.spirng.security;

import java.util.Arrays;

import org.jspecify.annotations.Nullable;
import org.springframework.security.authentication.AuthenticationProvider;
import org.springframework.security.authentication.BadCredentialsException;
import org.springframework.security.authentication.UsernamePasswordAuthenticationToken;
import org.springframework.security.core.Authentication;
import org.springframework.security.core.AuthenticationException;
import org.springframework.stereotype.Component;

@Component
public class MyAuthenticationProvider implements AuthenticationProvider {
	/*
	 * This AuthenticationProvider interface has 2 methods authenticate() method
	 * takes authentication Object and this authentication object has username and
	 * password. *
	 */

	@Override
	public @Nullable Authentication authenticate(Authentication authentication) throws AuthenticationException {

		// Get Principal i.e. username
		String userName = authentication.getName();

		// Get the password
		// authentication.getCredentials() return password but this is object u need to
		// converted to String.
		String password = authentication.getCredentials().toString();

		//Authentication Logic
		if("tom".equals(userName) && "cruise".equals(password)) {
			
			// This take username, password and also Optionally we pass grant or athority
			return new UsernamePasswordAuthenticationToken(userName, password, Arrays.asList());
		}else {
			throw new BadCredentialsException("Invalid Username or password");
		}
	}

	//We need to tell that we support username and password authentication token type.
	@Override
	public boolean supports(Class<?> authentication) {

		return authentication.equals(UsernamePasswordAuthenticationToken.class);
	}
	/*
	 * At runtime, the authentication manager passes this type within this authentication object		
	 *  to see if this provider supports this type of authentication 
     * and we are comparing it with username password authentication, token type.
If they both are same yes,
        this provider supports the username, and password authentication token type. 
        And you can use it.

	 */

}
```
# 22. Configure and Test
![alt text](image-28.png)![alt text](image-29.png)
# 23. Use Form Based Login
![alt text](image-30.png)![alt text](image-31.png)
### MySecurityConfig
```java
package com.bharath.spirng.security;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.Customizer;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.web.SecurityFilterChain;

@Configuration
public class MySecurityConfig {
	
	@Bean
	BCryptPasswordEncoder passwordEncoder() {
		return new BCryptPasswordEncoder();
	}
	
	@Bean
	SecurityFilterChain filterChain(HttpSecurity httpSecurity) {
			
//		httpSecurity.httpBasic(Customizer.withDefaults()); 
		httpSecurity.formLogin(Customizer.withDefaults()); //Use form login
	
		httpSecurity.authorizeHttpRequests(authorize -> authorize.anyRequest().authenticated());
		
		return httpSecurity.build();		
	}
}
```
# 24) Few More method
![alt text](image-32.png)![alt text](image-33.png)![alt text](image-34.png)
### HelloController
```java
package com.bharath.spirng.security;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HelloController {

	@GetMapping("/hello")
	public String hello() {
		
		return "Spring Security Rocks!";
	}
	
	
	@GetMapping("/bye")
	public String bye() {
		
		return "Get Lost!";
	}
}
```
## MySecurityConfig
```java
package com.bharath.spirng.security;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.Customizer;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.web.SecurityFilterChain;

@Configuration
public class MySecurityConfig {
	
	@Bean
	BCryptPasswordEncoder passwordEncoder() {
		return new BCryptPasswordEncoder();
	}
	
	@Bean
	SecurityFilterChain filterChain(HttpSecurity httpSecurity) {

		httpSecurity.formLogin(Customizer.withDefaults()); 
	
//		httpSecurity.authorizeHttpRequests(authorize -> authorize.anyRequest().authenticated());
		
		//Anybody who is authenticated can access hello endpoint.
//		httpSecurity.authorizeHttpRequests(authorize -> authorize.requestMatchers("/hello").authenticated());
		
		//more specific-- no other url hit by authenticate user
		httpSecurity.authorizeHttpRequests(authorize -> authorize.requestMatchers("/hello").authenticated().anyRequest().denyAll());
		
		return httpSecurity.build();		
	}
}
```
#  25) Create Custom Filter
![alt text](image-35.png)![alt text](image-36.png)![alt text](image-37.png)

### MySecurityFilter
```java
package com.bharath.spirng.security;

import java.io.IOException;

import jakarta.servlet.Filter;
import jakarta.servlet.FilterChain;
import jakarta.servlet.ServletException;
import jakarta.servlet.ServletRequest;
import jakarta.servlet.ServletResponse;

public class MySecurityFilter implements Filter {

	/*
	 * * Here attribute filterChain has complete chain of filter
	 */
	@Override
	public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
			throws IOException, ServletException {
		
		//Before invoking chain.doFilter(); if you want to do sth with request do it now.
		System.out.println("before"); 
		
		
		// This will send request and response to next filter; automatically
		chain.doFilter(request, response);
		
		System.out.println("after"); //if you want to do sth with response; do it here

	}

}
```

```java
package com.bharath.spirng.security;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.Customizer;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.web.SecurityFilterChain;
import org.springframework.security.web.authentication.www.BasicAuthenticationFilter;

@Configuration
public class MySecurityConfig {
	
	@Bean
	BCryptPasswordEncoder passwordEncoder() {
		return new BCryptPasswordEncoder();
	}
	
	@Bean
	SecurityFilterChain filterChain(HttpSecurity httpSecurity) {

//		httpSecurity.formLogin(Customizer.withDefaults()); 	
		httpSecurity.httpBasic(Customizer.withDefaults()); 	 //Change to basic authentication

//		httpSecurity.authorizeHttpRequests(authorize -> authorize.requestMatchers("/hello").authenticated().anyRequest().denyAll());
		httpSecurity.authorizeHttpRequests(authorize -> authorize.requestMatchers("/hello").authenticated());
		
		//I want to add filter before/after any other filer(here before )
		/*
		 * 1st arg - Filter iteself
		 * 2nd - before which filter u want to add.
		 */
		httpSecurity.addFilterBefore(new MySecurityFilter(), BasicAuthenticationFilter.class);
		
		return httpSecurity.build();		
	}
}
```
 # 26. Other filter classes.
![alt text](image-38.png)![alt text](image-39.png)


