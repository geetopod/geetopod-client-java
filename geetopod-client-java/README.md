![geetoPod - Identiy Solutions](https://github.com/geetopod/geetopod/raw/master/resources/images/geetopod-banner-96.png)

# Java Client

## How to use

### 1. Implement session handler

```
package com.geetopod.ceg;

import org.springframework.web.context.request.RequestContextHolder;
import org.springframework.web.context.request.ServletRequestAttributes;

import javax.servlet.http.HttpSession;

public class SpringBootSession extends com.ssopod.clients.SSOSession {
    public String getValue(String key) {
        ServletRequestAttributes attr = (ServletRequestAttributes) RequestContextHolder.currentRequestAttributes();
        HttpSession session = attr.getRequest().getSession();
        Object valueO = session.getAttribute(key);
        if (valueO == null) {
            return "";
        } else {
            return valueO + "";
        }
    }

    public void setValue(String key, String value) {
        ServletRequestAttributes attr = (ServletRequestAttributes) RequestContextHolder.currentRequestAttributes();
        HttpSession session = attr.getRequest().getSession();
        session.setAttribute(key, value);
    }
}
```

### 2. Set session handler

```
com.geetopod.clients.Services.instance().session(new SpringBootSession());
```

### 3. Set gateway url

```
String gatewayUrl = "http://gw.ssopod.com";
com.geetopod.clients.Services.instance().gatewayUrl(gatewayUrl);
```

### 4. Get services object

```
com.geetopod.clients.Services services = com.geetopod.clients.Services.instance();
```

### 5. Get client object

```
com.geetopod.clients.SSOClient client = com.geetopod.clients.Clients.instance().getClient();
```

### 6. Login (no OTP, no SSO)

```
com.geetopod.clients.SSOClient client = com.geetopod.clients.Clients.instance().getClient();

String company = "<company>";
boolean hasSSO = false;
boolean hasOTP = false;
String username = "<username>";
String password = "<password>";

LoginRequest loginRequest = new LoginRequest();
loginRequest.company = company;
loginRequest.hasSSO = hasSSO;
loginRequest.hasOTP = hasOTP;
loginRequest.username = username;
loginRequest.password = password;
LoginResponse loginResponse = client.login(loginRequest);
if (loginResponse.isError) {
    throw new AuthenticationServiceException("[" + loginResponse.errorCode + "] " + loginResponse.errorMessage, null);
} else {
    String token = loginResponse.token;
    long expiresIn = loginResponse.expiresIn;
    String refreshToken = loginResponse.refreshToken;
    long refreshExpiresIn = loginResponse.refreshExpiresIn;
}
```

### 7. Logout

```
boolean clearAllSessions = false;

com.geetopod.clients.SSOClient client = com.geetopod.clients.Clients.instance().getClient();

client.logout(clearAllSessions);
```

### 8. Login (has OTP, no SSO)

```
com.geetopod.clients.SSOClient client = com.geetopod.clients.Clients.instance().getClient();

String company = "<company>";
boolean hasSSO = false;
boolean hasOTP = true;
String username = "<username>";
String password = "<password>";

LoginRequest loginRequest = new LoginRequest();
loginRequest.company = company;
loginRequest.hasSSO = hasSSO;
loginRequest.hasOTP = hasOTP;
loginRequest.username = username;
loginRequest.password = password;
LoginResponse loginResponse = client.login(loginRequest);
if (loginResponse.isError) {
    throw new AuthenticationServiceException("[" + loginResponse.errorCode + "] " + loginResponse.errorMessage, null);
} else if (loginResponse.token.length() > 0) {
    String token = loginResponse.token;
    long expiresIn = loginResponse.expiresIn;
    String refreshToken = loginResponse.refreshToken;
    long refreshExpiresIn = loginResponse.refreshExpiresIn;
} else {
    String OTPToken = "<OTP token>";
    
    LoginOTPRequest loginOTPRequest = new LoginOTPRequest();
    loginOTPRequest.company = company;
    loginOTPRequest.phone = loginResponse.phone;
    loginOTPRequest.token = OTPToken;
    LoginOTPResponse loginOTPResponse = client.loginOTP(loginOTPRequest);
    if (loginOTPResponse.isError) {
        throw new AuthenticationServiceException("[" + loginOTPResponse.errorCode + "] " + loginOTPResponse.errorMessage, null);
    } else {
        String token = loginOTPResponse.token;
        long expiresIn = loginOTPResponse.expiresIn;
        String refreshToken = loginOTPResponse.refreshToken;
        long refreshExpiresIn = loginOTPResponse.refreshExpiresIn;
    }
}
```

### 9. Login (no OTP, has SSO)

```
@GetMapping("/login")
public String getLogin() {
    com.geetopod.clients.SSOClient client = com.geetopod.clients.Clients.instance().getClient();

    String company = "<company>";
    boolean hasSSO = false;
    boolean hasOTP = false;
    String username = "<username>";
    String password = "<password>";
    String webUrl = "http://localhost:8080";

    LoginRequest loginRequest = new LoginRequest();
    loginRequest.company = company;
    loginRequest.hasSSO = hasSSO;
    loginRequest.hasOTP = hasOTP;
    loginRequest.username = username;
    loginRequest.password = password;
    LoginResponse loginResponse = client.login(loginRequest);
    if (loginResponse.isError) {
        throw new AuthenticationServiceException("[" + loginResponse.errorCode + "] " + loginResponse.errorMessage, null);
    } else {
        String token = loginResponse.token;
        long expiresIn = loginResponse.expiresIn;
        String refreshToken = loginResponse.refreshToken;
        long refreshExpiresIn = loginResponse.refreshExpiresIn;
        String ssoToken = loginResponse.ssoToken;
        
        PutSSOTokenUrlRequest putSSOTokenUrlRequest = new PutSSOTokenUrlRequest();
        putSSOTokenUrlRequest.ssoToken = ssoToken;
        putSSOTokenUrlRequest.company = company;
        putSSOTokenUrlRequest.redirectUrl = webUrl + "/postlogin";
        PutSSOTokenUrlResponse putSSOTokenUrlResponse = com.geetopod.clients.Services.instance().putSSOTokenUrl(putSSOTokenUrlRequest);
        String goUrl = "/";
        if (!putSSOTokenUrlResponse.isError) {
            goUrl = putSSOTokenUrlResponse.url;
        }
        return "redirect:" + goUrl;
    }
}

```

### 10. Single sign on

```
@GetMapping("/postloginsso")
public String getPostLoginSSO(@RequestParam(name = "sso_token") String ssoToken) {
    com.geetopod.clients.SSOClient client = com.geetopod.clients.Clients.instance().getClient();

    String company = "<company>";

    ValidateSSOTokenRequest requestV = new ValidateSSOTokenRequest();
    requestV.company = company;
    requestV.ssoToken = ssoToken;
    ValidateSSOTokenResponse responseV = client.loginSSO(requestV);
    if (responseV.isError) {
        throw new AuthenticationServiceException("[" + responseV.errorCode + "] " + responseV.errorMessage, null);
    } else {
        return "redirect:/";
    }
}

@GetMapping("/loginsso")
public String getLoginSSO() {
    String company = "<company>";
    String webUrl = "http://localhost:8080";

    GetSSOTokenUrlRequest getSSOTokenUrlRequest = new GetSSOTokenUrlRequest();
    getSSOTokenUrlRequest.redirectUrl = webUrl + "/postloginsso";
    getSSOTokenUrlRequest.company = company;
    GetSSOTokenUrlResponse getSSOTokenUrlResponse = com.geetopod.clients.Services.instance().getSSOTokenUrl(getSSOTokenUrlRequest);
    if (!getSSOTokenUrlResponse.isError) {
        String goUrl = getSSOTokenUrlResponse.url;
        return "redirect:" + goUrl;
    } else {
        String message = getSSOTokenUrlResponse.errorMessage;
        return "redirect:/";
    }
}

```

### 11. Login with Github

```
@GetMapping("/social/login/github")
public String getSocialLoginGitHubUrl() {
    String webUrl = "http://ceg-springboot.geetopod.com";
    
    SocialLoginUrlRequest requestV = new SocialLoginUrlRequest();
    requestV.company = "ccsidd";
    requestV.provider = "github";
    requestV.redirectUrl = webUrl + "/social/callback/github";
    SocialLoginUrlResponse responseV = com.geetopod.clients.Services.instance().socialLoginUrl(requestV);
    String goUrl = responseV.url;
    
    return "redirect:" + goUrl;
}

@GetMapping("/social/callback/github")
public String getSocialLoginGitHubCallback(@RequestParam(name = "verifiedToken") String verifiedToken) {
    String webUrl = "http://ceg-springboot.geetopod.com";

    SocialLoginProcessRequest requestV = new SocialLoginProcessRequest();
    requestV.company = "ccsidd";
    requestV.verifiedToken = verifiedToken;
    requestV.hasSSO = true;
    SocialLoginProcessResponse responseV = com.geetopod.clients.Clients.instance().getClient(null).loginSocial(requestV);

    PutSSOTokenUrlRequest requestU = new PutSSOTokenUrlRequest();
    requestU.ssoToken = responseV.ssoToken;
    requestU.company = "ccsidd";
    requestU.redirectUrl = webUrl + "/postlogin";
    PutSSOTokenUrlResponse responseU = com.geetopod.clients.Services.instance().putSSOTokenUrl(requestU);
    String goUrl = responseU.url;

    return "redirect:" + goUrl;
}
```
