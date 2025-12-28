### Minimal SecurityConfig for hashing password

```java
package chatappjn.Config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.security.web.SecurityFilterChain;

@Configuration
public class SecurityConfig {

  @Bean
  public PasswordEncoder passwordEncoder() {
    return new BCryptPasswordEncoder();
  }

  @Bean
  public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
    http
        .csrf(csrf -> csrf.disable())
        .cors(cors -> {}) // enables CORS support
        .authorizeHttpRequests(auth -> auth
            .requestMatchers("/api/**").permitAll() // allow all REST endpoints
            .requestMatchers("/websocket/**").permitAll() // allow WS handshake
            .anyRequest().authenticated() // any other request requires auth
        );

    return http.build();
  }
}
```