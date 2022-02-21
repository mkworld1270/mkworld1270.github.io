---
layout: post
title:  "키클락 클라이언트 개발"
---

# 키클락 클라이언트 개발

[TOC]

## 개요

키클락 인증 방식을 사용하는 C# 클라이언트 샘플을 만들어봄

코드 샘플은 하단의 4가지 항목에 대하여 제공  

- 로그인 (보안 토큰 생성)

- 보안 토큰 사용하여 서버 접근 방법
- 보안 토큰 갱신 방법
- 토큰 디코드 방법



## 키클락 토큰 인증 과정

1. Token 발급 요청
   - REQ : 사용자 로그인 
   - RES  : 사용자 정보 확인 후 Token 발급 (AccessToken + RefreshToken)
2. Data 요청
   - REQ : 요청 Data ( + AccessToken)
   - RES  
     1. AccessToken 유효 => 요청 Data 전송
     2. AccessToken 만료 => AccessToken 만료 신호 전송
3. Token 갱신 요청
   - REQ : Token 갱신 요청 (AccessToken + RefreshToken)
   - RES : Token 확인 후 새로운 Token 발급



## 클라이언트 개발 

### 로그인 

##### EndPoint

| HTTP 메소드 | URL                                                          | 설명           |
| ----------- | ------------------------------------------------------------ | -------------- |
| POST        | {키클락 서버 url}/auth/realms/{realm}/protocol/openid-connect/token | 보안 토큰 생성 |

##### Body 

Content-Type : x-www-form-urlencoded

| 항목          | 설명               |
| ------------- | ------------------ |
| client-id     | client id          |
| username      | 유저명             |
| password      | 유저 비밀번호      |
| grant_type    | grant 유형         |
| client_secret | client 시크릿 코드 |

##### C# 클라이언트 샘플 코드

```c#
var tokenUrl = $"{_keyCloakBaseUrl}/auth/realms/{_realm}/protocol/openid-connect/token";            
var requestMessage = new HttpRequestMessage(HttpMethod.Post, tokenUrl)
{
    Content = new FormUrlEncodedContent(new List<KeyValuePair<string, string>>()
    {
        new KeyValuePair<string, string>("username", username),
        new KeyValuePair<string, string>("password", password),
        new KeyValuePair<string, string>("grant_type", grant_type),
        new KeyValuePair<string, string>("client_id", client_id),
        new KeyValuePair<string, string>("client_secret", client_secret)
    })
};

var httpResponseMessage = await _httpInvoker.SendAsync(requestMessage, CancellationToken.None);
// fail response
if (!httpResponseMessage.IsSuccessStatusCode)
{
    return null;
}

var responseJson = await httpResponseMessage.Content.ReadAsStringAsync();
```

##### 응답 결과

```json
{
    "access_token": "eyJhbGciOiJSUzI1NiIsInR5cCIgOiAiSldUIiwia2lkIiA6ICJNLXIwV0pzZmNVVFBiWGtxWWZMRWZGY29UX1pDeGE0cHpXUl96c0tuMFMwIn0.eyJleHAiOjE2NDU3NjUwODcsImlhdCI6MTY0NDkwMTA4NywianRpIjoiNWI2NDQ3MGItY2M3MS00Y2IwLWI3NWEtM2M2MjRiN2Q2YWJmIiwiaXNzIjoiaHR0cDovLzE5Mi4xNjguMTIxLjE2Mjo4MDgxL2F1dGgvcmVhbG1zL1NJVCIsImF1ZCI6ImFjY291bnQiLCJzdWIiOiJlNDdjNjcyOS05YTc1LTRmMGMtODM2Zi05MzE2NjM0NDk5OTAiLCJ0eXAiOiJCZWFyZXIiLCJhenAiOiJ3YXR0LWFwcCIsInNlc3Npb25fc3RhdGUiOiIxMTQ3YzRhOC01YTFlLTRjNGQtYTY2OS0zZDQ3YTI3MDYzOTkiLCJhY3IiOiIxIiwicmVhbG1fYWNjZXNzIjp7InJvbGVzIjpbIm9mZmxpbmVfYWNjZXNzIiwidW1hX2F1dGhvcml6YXRpb24iXX0sInJlc291cmNlX2FjY2VzcyI6eyJ3YXR0LWFwcCI6eyJyb2xlcyI6WyJ3YXR0X2FkbWluIiwid2F0dF91c2VyIiwid2F0dF9jaGllZl9hZG1pbiIsIndhdHRfZGV2Il19LCJhY2NvdW50Ijp7InJvbGVzIjpbIm1hbmFnZS1hY2NvdW50IiwibWFuYWdlLWFjY291bnQtbGlua3MiLCJ2aWV3LXByb2ZpbGUiXX19LCJzY29wZSI6ImVtYWlsIHByb2ZpbGUiLCJlbWFpbF92ZXJpZmllZCI6ZmFsc2UsIm5hbWUiOiLqtozrr7zqtJEiLCJwcmVmZXJyZWRfdXNlcm5hbWUiOiJtaW5rd2FuZyIsImRlcHQiOlsi6rCc67Cc7IS87YSwID4g7ISg7ZaJ7Jew6rWs7YyAIl0sInNpZCI6Ijc5OCJ9.ExkhUIB0K2AwZ7r5lxigvcTNyxUnlqDYK5Llz-c0Sqdd_Etz9LALUpE2EGZpLGtC6olAb-wl2W-eIy5EQcAtl5zX2DKV28STdeX5c9UMEoWpRc_xeD5p4voy__dToyWXTONfExzzn2ICIgbQY-hmkbOqQSkqWgGGEICel2BFEWGzHQPc_0EQk1y3rHV_ePd6t2oirZKFnuiaAtFFXdF4yQLIvTD22Fua8wa0U_AD4LcAloO2PxL_naHKtRf-xqedgJ4Q1Jc-P7OKOSLsuCL466iAyeTw94QXTtrTV-lhvM8z5XAZFdX2vhfTrqGQ6tUmb2k3IHIRkw5MowWt7Xpp7w",
    "expires_in": 864000,
    "refresh_expires_in": 1728000,
    "refresh_token": "eyJhbGciOiJIUzI1NiIsInR5cCIgOiAiSldUIiwia2lkIiA6ICIwOWE0M2M4MC0wZGQ2LTQyOGItODBkZC0xMjA2NWMzZDVkZDIifQ.eyJleHAiOjE2NDY2MjkwODcsImlhdCI6MTY0NDkwMTA4NywianRpIjoiYmExYWI0NGUtMWM2YS00OTBlLTllZmEtNWM4Yjg2M2M0ZGU4IiwiaXNzIjoiaHR0cDovLzE5Mi4xNjguMTIxLjE2Mjo4MDgxL2F1dGgvcmVhbG1zL1NJVCIsImF1ZCI6Imh0dHA6Ly8xOTIuMTY4LjEyMS4xNjI6ODA4MS9hdXRoL3JlYWxtcy9TSVQiLCJzdWIiOiJlNDdjNjcyOS05YTc1LTRmMGMtODM2Zi05MzE2NjM0NDk5OTAiLCJ0eXAiOiJSZWZyZXNoIiwiYXpwIjoid2F0dC1hcHAiLCJzZXNzaW9uX3N0YXRlIjoiMTE0N2M0YTgtNWExZS00YzRkLWE2NjktM2Q0N2EyNzA2Mzk5Iiwic2NvcGUiOiJlbWFpbCBwcm9maWxlIn0.83f5g0SyXuHXyTVr7OyJX0NGT9cn57t1Alrp5OWuQVc",
    "token_type": "Bearer",
    "not-before-policy": 0,
    "session_state": "1147c4a8-5a1e-4c4d-a669-3d47a2706399",
    "scope": "email profile"
}
```



### 토큰 갱신 

##### EndPoint

| HTTP 메소드 | URL                                                          | 설명           |
| ----------- | ------------------------------------------------------------ | -------------- |
| POST        | {키클락 서버 url}/auth/realms/{realm}/protocol/openid-connect/token | 보안 토큰 생성 |

##### Body 

Content-Type : x-www-form-urlencoded

| 항목          | 설명               |
| ------------- | ------------------ |
| client-id     | client 아이디      |
| client_secret | client 시크릿 코드 |
| grant_type    | "refresh_token"    |
| refresh_token | 리프레시 토큰      |

##### C# 클라이언트 샘플 코드

```c#
var tokenUrl = $"{_keyCloakBaseUrl}/auth/realms/{_realm}/protocol/openid-connect/token";            
var requestMessage = new HttpRequestMessage(HttpMethod.Post, tokenUrl)
{
    Content = new FormUrlEncodedContent(new List<KeyValuePair<string, string>>()
    {
        
        new KeyValuePair<string, string>("client_id", client_id),
        new KeyValuePair<string, string>("client_secret", client_secret),
        new KeyValuePair<string, string>("grant_type", "refresh_token"),
        new KeyValuePair<string, string>("refresh_token", refresh_token),
    })
};

var httpResponseMessage = await _httpInvoker.SendAsync(requestMessage, CancellationToken.None);
// fail response
if (!httpResponseMessage.IsSuccessStatusCode)
{
    return null;
}

var responseJson = await httpResponseMessage.Content.ReadAsStringAsync();
```

##### 응답결과

결과는 로그인 응답결과와 같음



### 토큰 디코딩 방법

로그인 / 토큰 갱신을 통해 얻은 AccessToken의 데이터를 사용하기 위해서 토큰 디코딩이 필요하다.

JWT 라이브러리를 사용하여 토큰을 읽어 필요한 내용을 사용하면 된다.  

##### 디코딩 방법

```c#
private JwtSecurityToken ReadToken(string signedAndEncodedToken)
{
    var tokenHandler = new JwtSecurityTokenHandler();
    SecurityToken validatedToken;
    validatedToken = tokenHandler.ReadToken(signedAndEncodedToken);
    return validatedToken as JwtSecurityToken;
}
```

##### 디코딩 AccessToken 내용

```c#
  "iat": 1620951551,
  "jti": "e9d2b8ad-53eb-4795-889c-4b2451850e2b",
  "iss": "{키클락 서버 url}/auth/realms/{realm}",
  "aud": [
    "login-app",
    "account"
  ],
  "sub": "4e6ecda2-9e29-4bae-9bfe-e0f3701b6059",
  "typ": "Bearer",
  "azp": "login-app",
  "session_state": "c28cab3f-38bf-4941-83f4-061a9016680e",
  "acr": "1",
  "realm_access": {
    "roles": [
      "offline_access",
      "uma_authorization"
    ]
  },
  "resource_access": {
  },
  "scope": "email profile",
  "email_verified": false,
  "preferred_username": "mk",
  "sid": "1"
}
```

### 