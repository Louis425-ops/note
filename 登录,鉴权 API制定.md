# 登录, 鉴权 API制定

## 登录鉴权系统

**基础登录流程及其过程中可能出现的问题**

用户在客户端输入username和password, 客户端将凭据发送给服务器, 服务器查询数据库进行验证, 返回验证结果

**主要问题**

#### 1. 安全问题

**密码明文传输**：直接传递密码容易被中途抓包

**中间人攻击**：即使密码加密，攻击者也可直接重放请求

我们可以使用使用HTTPS协议确保传输安全

#### 2. 

每次操作都需要传递密码，体验极差, 密码在客户端缓存存在安全风险

这个可以通过分离Login和Authentication过程

#### 3. 数据库安全问题

- 直接存储明文密码风险极高

**加盐Hash**

**密码安全存储示例：**

```go
import "golang.org/x/crypto/bcrypt"

func HashPassword(password string) (string, error) {
    hashedPassword, err := bcrypt.GenerateFromPassword([]byte(password), bcrypt.DefaultCost)
    if err != nil {
        return "", err
    }
    return string(hashedPassword), nil
}

func CheckPassword(hashedPassword, password string) bool {
    err := bcrypt.CompareHashAndPassword([]byte(hashedPassword), []byte(password))
    return err == nil
}
```

Cookie是存储在客户端本地的一段文本信息，按域名划分，大小为KB级别，以键值对形式展示。当浏览器访问网站时，会自动将对应域名下的Cookie与HTTP请求一同发送

### Cookie机制

**1. 生命周期管理**

- **Permanent Cookie**: 通过`ExpiredAt`和`MaxAge`控制过期时间
  - `ExpiredAt`: 绝对时间形式
  - `MaxAge`: 设置时间+offset形式（优先级更高）
  - `MaxAge`设为负数可立即删除Cookie
- **Session Cookie**: 未设置过期时间，在"session"结束后删除

**2. 访问限制**

```http
HTTP/2.0 200 OK
Content-Type: text/html
Set-Cookie: yummy_cookie=choco
Set-Cookie: tasty_cookie=strawberry
// JS访问Cookie
document.cookie="username=John Doe; expires=Thu, 18 Dec 2043 12:00:00 GMT";
```

**重要安全属性：**

- `HttpOnly`: 限制前端JS访问Cookie
- `Secure`: 仅允许HTTPS传输Cookie

**3. 作用范围**

- `Domain`: 限制Cookie所属域名，默认匹配子域名
- `Path`: 限制Cookie所属路径，默认为`/`

### Session机制

#### Session的必要性

- Cookie大小有限，无法存储大量信息
- 服务端需要存储用户状态信息（购物车、表单等）
- 服务器需要确认请求者身份

#### Session工作流程

1. 服务器创建Session，生成SessionID
2. SessionID通过Cookie发送给浏览器
3. 浏览器后续请求携带SessionID
4. 服务器通过SessionID识别用户并查询Session信息

**Session基于Cookie实现**

#### Session代码实现示例

```go
// 使用第三方库 github.com/gorilla/sessions
var store = sessions.NewCookieStore([]byte("Your-Secret-Key"))

func main() {
    store.Options.MaxAge = 30 // 设置过期时间
    
    // 登录处理
    r.GET("/login", func(c *gin.Context) {
        session, _ := store.Get(c.Request, "Your-Session-Name")
        session.Values["authenticated"] = true
        session.Values["username"] = "Username from frontend"
        session.Save(c.Request, c.Writer)
        c.String(http.StatusOK, "Login success")
    })
    
    // 验证访问
    r.GET("/home", func(c *gin.Context) {
        session, _ := store.Get(c.Request, "Your-Session-Name")
        if session.Values["authenticated"] == true {
            c.String(http.StatusOK, "Welcome "+session.Values["username"].(string))
        } else {
            c.String(http.StatusUnauthorized, "Please login first")
        }
    })
}
```

### 从Cookie到token

#### CSRF攻击

攻击者通过诱骗用户向目标网站发送请求，而浏览器自动携带Cookie，服务器误认为是合法请求。

#### JWT

```go
import "github.com/golang-jwt/jwt/v5"

const TokenExpireDuration = time.Second * 10
var CustomSecret = []byte("your-secret")

type CustomClaims struct {
    UserID   int64  `json:"user_id"`
    Username string `json:"username"`
    jwt.RegisteredClaims
}

// 生成Token
func GenToken(UserID int64, Username string) (string, error) {
    claims := CustomClaims{
        UserID,
        Username,
        jwt.RegisteredClaims{
            ExpiresAt: jwt.NewNumericDate(time.Now().Add(TokenExpireDuration)),
            Issuer:    "lh",
        },
    }
    token := jwt.NewWithClaims(jwt.SigningMethodHS256, claims)
    return token.SignedString(CustomSecret)
}

// 解析Token
func ParseToken(tokenString string) (*CustomClaims, error) {
    var claims = new(CustomClaims)
    token, err := jwt.ParseWithClaims(tokenString, claims, func(token *jwt.Token) (interface{}, error) {
        return CustomSecret, nil
    })
    
    if err != nil {
        return nil, err
    }
    
    if token.Valid {
        return claims, nil
    }
    return nil, errors.New("invalid token")
}
```

#### Token存储与传递

**localstorge：**

```javascript
localStorage.setItem('TokenName', token); // 保存
localStorage.getItem('TokenName');        // 读取
```

**HTTP传递：**

```javascript
const token = localStorage.getItem('TokenName');
const response = await fetch("http://localhost:8081/checkToken", {
    method: "POST",
    headers: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${token}`
    }
});
```

**Access JWT + Refresh JWT：**

- Access JWT: 短期（30分钟）
- Refresh JWT: 长期（12小时）
- Access过期时使用Refresh获取新Token

### OAuth 2.0

第三方应用（如DeepSeek）通过OAuth获得用户授权，访问鉴权方（如微信）的用户资源

```go
var oauthConfig = oauth2.Config{
    ClientID:     "your-client-id",
    ClientSecret: "your-client-secret",
    RedirectURL:  "your-redirect-uri",
    Scopes:       []string{"scope1", "scope2"},
    Endpoint: oauth2.Endpoint{
        AuthURL:  "auth-endpoint-url",
        TokenURL: "token-endpoint-url",
    },
}

func handleLogin(w http.ResponseWriter, r *http.Request) {
    url := oauthConfig.AuthCodeURL(oauthStateString)
    http.Redirect(w, r, url, http.StatusTemporaryRedirect)
}

func handleCallback(w http.ResponseWriter, r *http.Request) {
    state := r.FormValue("state")
    if state != oauthStateString {
        // CSRF防护
        return
    }
    
    code := r.FormValue("code")
    token, err := oauthConfig.Exchange(context.Background(), code)
    // 使用token调用API
}
```

------

## 二、API

API（Application Programming Interface）是前端和后端进行通信和交换数据的规范，确定前后端交互的路径、方法和参数

### 统一Response格式

```javascript
// 统一response格式
{
    "code": 0,        // 0表示成功，其他表示失败
    "msg": "success", // 返回信息
    "data": any       // 返回数据
}

// 获取评论示例
{
    "code": 0,
    "msg": "success",
    "data": {
        "total": 121,
        "comments": [
            {
                "id": 1,
                "name": "用户名",
                "content": "评论内容"
            }
        ]
    }
}
```

#### URL设计

```
https://api.abc.com/PROG/user/get [GET]
```

#### HTTP方法

### GET请求传参方式

#### 1. 查询字符串

```
/api/user?id=123&name=xyz
```

#### 2. URL参数

```
/user/getById/abc123
```

#### 3. 参数数组

```
/api/user?filter[]=admin&filter[]=active
```

#### 4. 参数对象

```
/api/user?filter={"role":"admin", "status":"active"}
```

### POST请求传参方式

#### JSON数据

```javascript
async function sendJsonData(url, data) {
    try {
        const response = await fetch(url, {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json'
            },
            body: JSON.stringify(data)
        });
        
        if (!response.ok) {
            throw new Error(`HTTP error! Status: ${response.status}`);
        }
        
        const result = await response.json();
        return result;
    } catch (error) {
        console.error('Error:', error);
    }
}
```

其他略过了

### 状态码设计

#### HTTP状态码

- **1xx**: 信息状态码
- **2xx**: 成功状态码
- **3xx**: 重定向状态码
- **4xx**: 客户端错误
- **5xx**: 服务器错误

#### 业务状态码（JSON ErrCode）

```javascript
// 登录失败
{
    "errcode": 10001,
    "errmsg": "password incorrect",
    "data": null
}

// 登录成功
{
    "errcode": 0,
    "errmsg": "",
    "data": {
        "id": "user123",
        "name": "用户名",
        "phone": "手机号"
    }
}
```

### 特殊数据类型处理

- **时间**: RFC3339格式或Unix时间戳
- **二进制数据**: Base64编码后作为字符串
- **大容量资源**: 通过单独接口上传，返回URL或key