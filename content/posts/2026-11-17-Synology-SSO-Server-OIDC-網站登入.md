+++
title = '【Synology】- 使用 SSO Server 與 OIDC 串接網站登入'
date = '2026-11-17T11:28:00+08:00'
slug = 'Synology-SSO-Server-OIDC-網站登入'
description = '記錄如何設定 Synology SSO Server 的 OIDC 服務，並以旅程網站為例說明登入流程與用戶端設定。'
categories = ['Synology', '後端技術']
tags = ['Synology', 'SSO', 'OIDC', '身分驗證']
keywords = ['Synology SSO Server', 'OIDC', 'OpenID Connect', 'PKCE', '網站登入']
image = '/image/2026-11-17-synology-oidc-general.png'
+++

## 前言

前幾個月在研究 Synology 的功能時，想到能不能把 NAS 上的帳號套用到自己開發的網站。以前接觸過 Microsoft Entra ID 管理使用者身分，這次則想試試使用 Synology 的 SSO Server，讓自己的應用程式透過 OIDC 完成登入。

這篇以自己的旅程網站為例，記錄 SSO Server 的設定與網站端登入流程。這裡是讓 **Synology SSO Server 擔任身分提供者**，不是把 Microsoft Entra ID 串進 Synology。

---

## 一、OIDC 是什麼？

OIDC（OpenID Connect）是建立在 OAuth 2.0 之上的身分驗證協定。網站可以把使用者導向身分提供者登入，成功後再驗證取得的身分資訊，不需要自行處理使用者的 Synology 密碼。

本次範例中的角色：

| 角色 | 本篇範例 | 職責 |
|------|----------|------|
| 身分提供者（Provider） | Synology SSO Server | 驗證帳號，提供 OIDC 授權與身分資訊 |
| 用戶端（Client） | 旅程網站 | 發起登入、驗證身分，並建立網站自己的登入狀態 |
| 使用者 | 網站訪客 | 在 Synology 登入頁輸入帳號密碼 |

網站收到使用者身分後，仍需要依自己的規則決定可以使用哪些功能；**登入成功不代表自動取得管理員權限**。

---

## 二、前置準備

1. 一台可使用 DSM 的 Synology NAS，並從「套件中心」安裝 `SSO Server`。
2. 準備可登入 SSO 服務的帳號，以及網站所需的使用者或群組。
3. 準備可供瀏覽器與網站後端連線的 SSO HTTPS 網址，並設定有效的 TLS 憑證。
4. 準備支援 OIDC 的網站，以及登入完成後要返回的 HTTPS 回呼網址。

本篇使用以下示意網址，請換成自己的實際設定：

| 用途 | 範例 |
|------|------|
| SSO Server 網域 | `https://sso.example.com` |
| 旅程網站網域 | `https://travel.example.com` |
| 網站回呼網址 | `https://travel.example.com/api/auth/oidc/callback` |

> DSM、SSO Server 版本與網路環境不同時，畫面和設定方式可能有所差異。若服務需從外網存取，請另外確認 DNS、反向代理、TLS 憑證與存取權限。

---

## 三、設定 Synology SSO Server

### 3.1 設定帳號類型與伺服器 URL

開啟 `SSO Server`，進入「一般設定」：

1. 依自己的帳號來源選擇「帳號類型」。畫面中使用的是「網域 / LDAP / 本地端」。
2. 設定 SSO 服務使用的「伺服器 URL」。
3. 按下「套用」儲存。

![Synology SSO Server 一般設定](/image/2026-11-17-synology-oidc-general.png)

這裡設定的是 SSO 服務網址；網站需要填入的 **Issuer** 則應以稍後 Discovery 文件中的 `issuer` 欄位為準，不要直接假設兩者一定相同。

### 3.2 啟用 OIDC 伺服器

1. 在左側選擇「服務」，切換到 `OIDC`。
2. 勾選「啟用 OIDC 伺服器」，按下「儲存」。
3. 複製畫面上的 `Well-Known URL`，後面用來確認 Discovery 文件。

![啟用 Synology OIDC 伺服器並取得 Well-Known URL](/image/2026-11-17-synology-oidc-service.png)

Well-Known URL 指向 OIDC Discovery 文件，其中包含 `issuer`、授權端點、Token 端點與公開金鑰位置。**Well-Known URL 是文件網址，Issuer 是身分提供者的識別網址，兩者不能直接當成同一個設定值。**

### 3.3 新增 OIDC 應用程式

1. 選擇左側「應用程式」，按下「新增」。
2. 在「選擇 SSO 協定」畫面選擇 `OIDC`，按下「下一步」。
3. 依精靈填寫應用程式名稱及網站的重新導向 URI，完成新增。

![在 SSO Server 新增 OIDC 應用程式](/image/2026-11-17-synology-oidc-application.png)

本篇以 `travel` 作為應用程式名稱，重新導向 URI 範例為：

```text
https://travel.example.com/api/auth/oidc/callback
```

新增後可在應用程式的編輯畫面查看應用程式 ID 與密鑰：

![OIDC 應用程式名稱、重新導向 URI、應用程式 ID 與密鑰](/image/2026-11-17-synology-oidc-client.png)

| SSO Server 欄位 | 網站端對應 | 說明 |
|----------------|------------|------|
| 重新導向 URI | Redirect URI | 登入完成後，瀏覽器返回網站的網址 |
| 應用程式 ID | Client ID | 識別這個網站用戶端 |
| 應用程式密鑰 | Client Secret | 供網站後端向 SSO Server 驗證用戶端 |

> 截圖中的網址已遮蔽，實際路徑請以你的網站為準。SSO Server 登記的 URI 必須與網站送出的 `redirect_uri` 完整一致，包括協定、網域、埠號及路徑。

---

## 四、設定網站用戶端

以下是旅程網站使用的環境變數範例：

```dotenv
TRIP_OIDC_ISSUER=https://sso.example.com/webman/sso
TRIP_OIDC_CLIENT_ID=replace-with-sso-client-id
TRIP_OIDC_CLIENT_SECRET=replace-with-sso-client-secret
TRIP_OIDC_REDIRECT_URI=https://travel.example.com/api/auth/oidc/callback
```

![旅程網站的 OIDC 環境變數範例](/image/2026-11-17-synology-oidc-env.png)

| 設定 | 要填入的內容 |
|------|--------------|
| `TRIP_OIDC_ISSUER` | Discovery 文件中的 `issuer`；上方路徑僅供示意 |
| `TRIP_OIDC_CLIENT_ID` | SSO Server 的應用程式 ID |
| `TRIP_OIDC_CLIENT_SECRET` | SSO Server 的應用程式密鑰 |
| `TRIP_OIDC_REDIRECT_URI` | 與 SSO Server 登記的重新導向 URI 相同 |

這些變數名稱是旅程網站自己的設定，不是 OIDC 統一規定；其他網站可能使用不同名稱。旅程網站透過 `os.Getenv` 讀取環境變數，不會自行載入 `.env`；需要由 Docker 或執行環境注入。

> Client Secret 只放在後端的安全設定中，不要寫進前端程式、公開文章或 Git。若密鑰外洩，應重新產生密鑰並同步更新網站設定。

### 4.1 確認 Discovery 文件

`Well-Known URL` 不是 Synology 專屬功能，而是 OIDC 用來提供 **Discovery（服務探索）文件**的標準機制。Synology SSO Server 啟用 OIDC 後，會在畫面上提供這個網址，讓網站知道去哪裡取得服務設定。

網站讀取這份 JSON 後，可以找到身分提供者的 `issuer`、登入用的授權端點、交換 Token 的端點，以及驗證簽章所需的公開金鑰位置，不必手動猜測各個網址。這份文件是服務中繼資料，不是 Client Secret，也不是使用者的 Token。

可以用瀏覽器開啟 SSO Server 畫面顯示的 Well-Known URL，確認回傳的 JSON。以下內容與端點路徑僅為示意，實際值請以自己的服務為準：

```json
{
  "issuer": "https://sso.example.com/webman/sso",
  "authorization_endpoint": "https://sso.example.com/example/authorize",
  "token_endpoint": "https://sso.example.com/example/token",
  "jwks_uri": "https://sso.example.com/example/jwks",
  "userinfo_endpoint": "https://sso.example.com/example/userinfo"
}
```

旅程網站會在設定的 Issuer 後加上 `/.well-known/openid-configuration` 取得文件，並確認回傳的 Issuer 與設定一致。此專案也要求使用的端點為 HTTPS，且與 Issuer 同來源；其他 OIDC 用戶端不一定有相同限制。

### 4.2 Issuer 是什麼？

`Issuer` 可以理解成「**發行者**」：它是用來識別哪一個身分提供者簽發 ID Token 的網址。在這篇範例中，發行者是 Synology SSO Server，不是旅程網站。

例如，Discovery 文件回傳：

```json
{
  "issuer": "https://sso.example.com/webman/sso"
}
```

網站就把這個值填入 `TRIP_OIDC_ISSUER`。登入後，後端會確認 ID Token 的 `iss` 欄位是否與預期的 Issuer 一致，並驗證簽章與其他必要欄位，避免接受來自錯誤身分提供者的 Token。

| 網址 | 用途 | 本篇範例 |
|------|------|----------|
| Issuer | 識別受信任的身分提供者 | `https://sso.example.com/webman/sso` |
| Well-Known URL | 取得 Discovery 設定文件 | `https://sso.example.com/webman/sso/.well-known/openid-configuration` |
| Redirect URI | 登入完成後返回網站的網址 | `https://travel.example.com/api/auth/oidc/callback` |

上面都是示意值；請開啟 SSO Server 畫面上的 Well-Known URL，使用文件中實際的 `issuer`，不要只填 NAS 網域或把整個 Well-Known URL 當成 Issuer。

---

## 五、網站登入流程

旅程網站使用 **Authorization Code Flow + PKCE**，由後端處理授權碼交換與身分驗證：

### 5.1 Authorization Code Flow + PKCE 是什麼？

**Authorization Code Flow（授權碼流程）** 是一種登入流程：使用者先前往身分提供者登入，成功後，身分提供者不直接把 Token 放在瀏覽器回呼網址中，而是先給網站一個短暫有效的 **授權碼（Authorization Code）**。網站後端再用授權碼向 Token 端點交換 Token。

**PKCE**（Proof Key for Code Exchange，唸作「pixy」）是授權碼流程的額外保護，用來降低授權碼被攔截後遭其他人拿去交換 Token 的風險。它的做法是：

1. 網站後端先產生隨機的 `code_verifier`，並以 SHA-256 計算對應的 `code_challenge`。
2. 導向 SSO 登入時，只把 `code_challenge` 送到授權端點；`code_verifier` 留在後端。
3. 使用者登入成功後，SSO 將授權碼送回網站。
4. 網站後端交換 Token 時，連同授權碼送出原本的 `code_verifier`。
5. SSO 檢查 `code_verifier` 是否對應一開始收到的 `code_challenge`，通過後才繼續處理 Token 請求。

簡單說，**授權碼是一次性的交換憑據，PKCE 則要求交換者證明自己持有發起登入時的驗證值**。在本範例中，網站後端還會使用 Client Secret 驗證自己的身分；PKCE 不會取代密鑰保護、HTTPS、Token 驗證或網站自身的權限判斷。

`state` 與 `nonce` 也各有用途：前者用來關聯登入交易，後者用來確認 ID Token 對應本次登入。實作時還需要把登入交易安全地綁定發起登入的瀏覽器，不能只依賴資料庫中存在 `state`。

### 5.2 完整流程

```text
使用者點擊網站登入
    ↓
網站後端讀取 OIDC Discovery 文件
    ↓
瀏覽器前往 Synology SSO 登入
    ↓
SSO 導回網站，附上 code 與 state
    ↓
網站後端確認登入交易並交換 Token
    ↓
驗證 ID Token，取得使用者身分與群組
    ↓
建立網站自己的 Session，返回首頁
```

### 5.3 發起登入

旅程網站的登入入口為 `/api/auth/oidc/start`。後端產生一次性的 `state`、`nonce` 與 PKCE 參數，再將瀏覽器導向 Discovery 文件中的授權端點。

| 參數 | 用途 |
|------|------|
| `state` | 確認回呼對應到先前發起的登入交易 |
| `nonce` | 確認 ID Token 對應到本次登入請求 |
| PKCE | 讓授權碼交換時必須搭配原始的 `code_verifier` |

此專案將登入交易存入資料庫，設定為十分鐘有效，並在回呼時取出刪除，避免重複使用。

### 5.4 回呼與交換 Token

使用者在 SSO 完成登入後，瀏覽器會帶著授權碼與 `state` 返回網站的回呼網址。後端確認登入交易有效，再使用授權碼、Client Secret 與 PKCE `code_verifier`，向 Discovery 文件指定的 Token 端點交換 Token。

這個交換由**網站後端直接送給 SSO Server**，不是讓前端取得 Client Secret。

### 5.5 驗證身分與建立 Session

取得 ID Token 後，旅程網站會透過 JWKS 公開金鑰驗證 RS256 簽章，並檢查 Issuer、Audience、必要時的 `azp`、有效期限、簽發時間與 `nonce`，以及必要的使用者識別資訊。

此專案會從 ID Token 取得群組；若沒有群組資訊，則嘗試使用 UserInfo 端點。接著依帳號與群組決定網站角色，建立網站自己的 Session Cookie。

> ID Token 用來驗證身分；網站 Session 用來維持後續登入狀態。`groups` 不是所有 OIDC 服務都保證提供的標準 Scope，換用其他 Provider 前要確認支援情況與權限規則。

---

## 六、Go + Gin：不使用資料庫的完整範例

以下使用 Go + Gin 建立獨立的 OIDC 登入範例。登入交易與 Session 暫存在程式記憶體中，不需要設定資料庫。

> 此範例適合單一程序測試：重啟後登入狀態會消失，也不適合多台伺服器共享 Session。正式環境需要依需求加入持久化或共享儲存、Session 撤銷、權限控管及其他安全防護。

### 6.1 建立專案

先安裝 Go，建立資料夾後在 PowerShell 執行：

```powershell
mkdir oidc-demo
cd oidc-demo
go mod init example.com/oidc-demo
go get github.com/gin-gonic/gin github.com/coreos/go-oidc/v3/oidc golang.org/x/oauth2
```

### 6.2 建立 `main.go`

將以下內容存成 `main.go`。本範例用 OIDC 套件處理 Discovery 與 ID Token 驗證，用記憶體保存登入交易與 Session：

```go
package main

import (
    "context"
    "crypto/subtle"
    "log"
    "net/http"
    "net/url"
    "os"
    "sync"
    "time"

    "github.com/coreos/go-oidc/v3/oidc"
    "github.com/gin-gonic/gin"
    "golang.org/x/oauth2"
)

type loginTransaction struct {
    state, nonce, verifier string
    expires time.Time
}

type loginSession struct {
    subject string
    expires time.Time
}

func main() {
    issuer := os.Getenv("TRIP_OIDC_ISSUER")
    clientID := os.Getenv("TRIP_OIDC_CLIENT_ID")
    clientSecret := os.Getenv("TRIP_OIDC_CLIENT_SECRET")
    redirectURI := os.Getenv("TRIP_OIDC_REDIRECT_URI")
    callbackURL, err := url.Parse(redirectURI)
    if issuer == "" || clientID == "" || clientSecret == "" ||
        err != nil || callbackURL.Scheme != "https" ||
        callbackURL.Host == "" || callbackURL.Path != "/api/auth/oidc/callback" {
        log.Fatal("請設定 OIDC 環境變數及 HTTPS 回呼網址")
    }

    ctx, cancel := context.WithTimeout(context.Background(), 10*time.Second)
    provider, err := oidc.NewProvider(ctx, issuer)
    cancel()
    if err != nil {
        log.Fatalf("無法讀取 OIDC Discovery：%v", err)
    }

    config := oauth2.Config{
        ClientID: clientID,
        ClientSecret: clientSecret,
        RedirectURL: redirectURI,
        Endpoint: provider.Endpoint(),
        Scopes: []string{oidc.ScopeOpenID},
    }
    config.Endpoint.AuthStyle = oauth2.AuthStyleInParams
    verifier := provider.Verifier(&oidc.Config{ClientID: clientID})

    var mu sync.Mutex
    transactions := make(map[string]loginTransaction)
    sessions := make(map[string]loginSession)
    router := gin.New()
    router.Use(gin.Recovery())
    if err := router.SetTrustedProxies(nil); err != nil {
        log.Fatal(err)
    }

    router.GET("/", func(c *gin.Context) {
        c.Data(http.StatusOK, "text/html; charset=utf-8",
            []byte(`<a href="/api/auth/oidc/start">使用 SSO 登入</a>`))
    })

    router.GET("/api/auth/oidc/start", func(c *gin.Context) {
        browserID := oauth2.GenerateVerifier()
        tx := loginTransaction{
            state: oauth2.GenerateVerifier(),
            nonce: oauth2.GenerateVerifier(),
            verifier: oauth2.GenerateVerifier(),
            expires: time.Now().Add(10 * time.Minute),
        }

        mu.Lock()
        for id, item := range transactions {
            if time.Now().After(item.expires) {
                delete(transactions, id)
            }
        }
        for id, item := range sessions {
            if time.Now().After(item.expires) {
                delete(sessions, id)
            }
        }
        transactions[browserID] = tx
        mu.Unlock()

        c.SetSameSite(http.SameSiteLaxMode)
        c.SetCookie("oidc_login", browserID, 600, "/", "", true, true)
        c.Redirect(http.StatusFound, config.AuthCodeURL(
            tx.state, oidc.Nonce(tx.nonce), oauth2.S256ChallengeOption(tx.verifier),
        ))
    })

    router.GET("/api/auth/oidc/callback", func(c *gin.Context) {
        browserID, cookieErr := c.Cookie("oidc_login")
        mu.Lock()
        tx, found := transactions[browserID]
        delete(transactions, browserID)
        mu.Unlock()
        c.SetSameSite(http.SameSiteLaxMode)
        c.SetCookie("oidc_login", "", -1, "/", "", true, true)

        if cookieErr != nil || !found || time.Now().After(tx.expires) ||
            subtle.ConstantTimeCompare([]byte(c.Query("state")), []byte(tx.state)) != 1 {
            c.String(http.StatusBadRequest, "登入交易無效或已過期，請重新登入")
            return
        }
        if c.Query("error") != "" || c.Query("code") == "" {
            c.String(http.StatusBadRequest, "SSO 登入未完成")
            return
        }

        ctx, cancel := context.WithTimeout(c.Request.Context(), 10*time.Second)
        defer cancel()
        token, err := config.Exchange(ctx, c.Query("code"), oauth2.VerifierOption(tx.verifier))
        if err != nil {
            log.Printf("OIDC Token 交換失敗：%v", err)
            c.String(http.StatusBadGateway, "無法交換 Token")
            return
        }
        rawIDToken, ok := token.Extra("id_token").(string)
        if !ok || rawIDToken == "" {
            c.String(http.StatusBadGateway, "SSO 未回傳 ID Token")
            return
        }
        idToken, err := verifier.Verify(ctx, rawIDToken)
        if err != nil || idToken.Nonce != tx.nonce || idToken.Subject == "" {
            c.String(http.StatusUnauthorized, "ID Token 驗證失敗")
            return
        }

        sessionID := oauth2.GenerateVerifier()
        mu.Lock()
        sessions[sessionID] = loginSession{idToken.Subject, time.Now().Add(time.Hour)}
        mu.Unlock()
        c.SetCookie("oidc_session", sessionID, 3600, "/", "", true, true)
        c.Header("Cache-Control", "no-store")
        c.Header("Referrer-Policy", "no-referrer")
        c.Redirect(http.StatusSeeOther, "/me")
    })

    router.GET("/me", func(c *gin.Context) {
        sessionID, err := c.Cookie("oidc_session")
        mu.Lock()
        session, found := sessions[sessionID]
        if found && time.Now().After(session.expires) {
            delete(sessions, sessionID)
            found = false
        }
        mu.Unlock()
        if err != nil || !found {
            c.String(http.StatusUnauthorized, "尚未登入")
            return
        }
        c.String(http.StatusOK, "登入成功，使用者識別碼：%s", session.subject)
    })

    if err := router.Run("127.0.0.1:8080"); err != nil {
        log.Fatal(err)
    }
}
```

此範例把隨機瀏覽器識別碼放在短效、`HttpOnly`、`Secure` 的 Cookie，讓回呼時可以找到該瀏覽器的登入交易；`state`、`nonce` 與 PKCE 驗證值留在伺服器記憶體。回呼取出交易後立即刪除，避免重複使用。

### 6.3 設定並啟動

先在 Synology SSO Server 新增 OIDC 應用程式，將重新導向 URI 設為你的網站公開 HTTPS 網址，例如 `https://travel.example.com/api/auth/oidc/callback`。再將網站的 HTTPS 反向代理指向執行 Go 程式的主機 `127.0.0.1:8080`；DNS 與 TLS 憑證也必須設定完成。

在 PowerShell 設定環境變數，將示意值換成自己的資料，再啟動程式：

```powershell
$env:TRIP_OIDC_ISSUER = "https://sso.example.com/webman/sso"
$env:TRIP_OIDC_CLIENT_ID = "你的應用程式ID"
$env:TRIP_OIDC_CLIENT_SECRET = "你的應用程式密鑰"
$env:TRIP_OIDC_REDIRECT_URI = "https://travel.example.com/api/auth/oidc/callback"
go run .
```

開啟 `https://travel.example.com`，點擊「使用 SSO 登入」。完成 Synology 登入後，應返回網站的 `/me`，顯示使用者識別碼。Cookie 設有 `Secure`，所以**不能直接用 `http://localhost:8080` 完成登入**。

Issuer 應以 Discovery 文件為準，Redirect URI 必須與 SSO Server 的登記值完全一致。此範例使用 Authorization Code + PKCE 和 `openid` Scope；實際能否登入仍取決於 SSO Server 版本、用戶端設定與 Provider 支援的功能。

> 旅程網站原本的實作不同：它使用資料庫儲存交易，且目前沒有將交易綁定發起登入的瀏覽器。正式使用前，應補上瀏覽器綁定以降低登入 CSRF 風險。

---

## 七、C# + ASP.NET Core：OIDC 登入範例

ASP.NET Core 可以使用官方的 OpenID Connect 驗證處理器處理 Discovery、授權碼交換、Token 驗證，以及登入流程中的 correlation 與 nonce。以下是獨立網站範例，並不是旅程網站現有的 Go 程式。

### 7.1 安裝套件與設定環境變數

在 ASP.NET Core Web 專案中安裝與使用的 .NET 主要版本相容的套件：

```console
dotnet add package Microsoft.AspNetCore.Authentication.OpenIdConnect
```

在執行環境注入以下變數，值從 Synology SSO Server 取得：

```dotenv
TRIP_OIDC_ISSUER=https://sso.example.com/webman/sso
TRIP_OIDC_CLIENT_ID=replace-with-sso-client-id
TRIP_OIDC_CLIENT_SECRET=replace-with-sso-client-secret
```

`TRIP_OIDC_ISSUER` 請以實際 Discovery 文件中的 `issuer` 為準；Client Secret 使用環境的安全密鑰管理機制，不要提交到 Git。

### 7.2 設定驗證與登入端點

以下可放在 Web 專案的 `Program.cs`。應用程式必須以 HTTPS 對外提供服務：

```csharp
using Microsoft.AspNetCore.Authentication;
using Microsoft.AspNetCore.Authentication.Cookies;
using Microsoft.AspNetCore.Authentication.OpenIdConnect;
using Microsoft.IdentityModel.Protocols.OpenIdConnect;

var builder = WebApplication.CreateBuilder(args);

var issuer = builder.Configuration["TRIP_OIDC_ISSUER"];
var clientId = builder.Configuration["TRIP_OIDC_CLIENT_ID"];
var clientSecret = builder.Configuration["TRIP_OIDC_CLIENT_SECRET"];

if (string.IsNullOrWhiteSpace(issuer) ||
    string.IsNullOrWhiteSpace(clientId) ||
    string.IsNullOrWhiteSpace(clientSecret))
{
    throw new InvalidOperationException("OIDC 設定不完整。");
}

builder.Services
    .AddAuthentication(options =>
    {
        options.DefaultScheme = CookieAuthenticationDefaults.AuthenticationScheme;
        options.DefaultChallengeScheme = OpenIdConnectDefaults.AuthenticationScheme;
    })
    .AddCookie(options =>
    {
        options.Cookie.HttpOnly = true;
        options.Cookie.SecurePolicy = CookieSecurePolicy.Always;
    })
    .AddOpenIdConnect(options =>
    {
        options.Authority = issuer;
        options.RequireHttpsMetadata = true;
        options.ClientId = clientId;
        options.ClientSecret = clientSecret;
        options.ResponseType = OpenIdConnectResponseType.Code;
        options.UsePkce = true;
        options.CallbackPath = "/api/auth/oidc/callback";
        options.Scope.Clear();
        options.Scope.Add("openid");
        options.SaveTokens = false;
    });

builder.Services.AddAuthorization();

var app = builder.Build();
app.UseAuthentication();
app.UseAuthorization();

app.MapGet("/", () =>
    Results.Content("<a href=\"/login\">使用 SSO 登入</a>", "text/html"));

app.MapGet("/login", () => Results.Challenge(
    new AuthenticationProperties { RedirectUri = "/me" },
    new[] { OpenIdConnectDefaults.AuthenticationScheme }));

app.MapGet("/me", (HttpContext context) =>
    Results.Text($"已登入：{context.User.Identity?.Name ?? "使用者"}"))
    .RequireAuthorization();

app.Run();
```

將 Synology 應用程式的重新導向 URI 設為網站公開網址加上 `CallbackPath`，例如 `https://travel.example.com/api/auth/oidc/callback`。若網站放在反向代理後方，必須正確設定受信任代理的轉送標頭，讓 ASP.NET Core 知道外部使用的是 HTTPS 與正確網域。

這個範例只要求 `openid`，驗證成功後建立 Cookie，沒有實作管理員角色對應。若要使用 `email` 或 `groups`，應先確認 Synology 版本、授權 Scope 與實際回傳的 Claims，再依網站需求明確設定；不要假設所有供應商都會回傳同樣的欄位。

> 此 C# 範例示範標準處理器的設定，尚需在自己的 Synology 與 .NET 環境實際驗證相容性。

---

## 結語

透過 Synology SSO Server，可以讓自己的網站把登入交給既有的身分服務處理。設定上最重要的是分清楚 **Issuer、Well-Known URL、Client ID、Client Secret 與 Redirect URI**，並讓網站後端確實驗證 ID Token。

這篇記錄的是旅程網站的實作方式；不同網站或 Provider 的端點、Scope、群組資訊與安全限制可能不同，串接前還是要以實際服務與應用程式的文件為準。

## 參考資料

- [Synology：使用 SSO Server 設定 OIDC SSO](https://kb.synology.com/en-sg/DSM/tutorial/set_up_oidc_for_dsm_in_sso_server)
- [Synology：SSO Server 應用程式設定](https://kb.synology.com/en-nz/PAS/help/SSOServer/sso_server_application_list?version=1_0)
- [旅程網站原始碼：JontCont/travel-site](https://github.com/JontCont/travel-site)
- [Microsoft：在 ASP.NET Core 設定 OpenID Connect](https://learn.microsoft.com/zh-tw/aspnet/core/security/authentication/configure-oidc-web-authentication)
- [OpenID Connect Core 1.0](https://openid.net/specs/openid-connect-core-1_0.html)
- [RFC 7636：Proof Key for Code Exchange](https://www.rfc-editor.org/rfc/rfc7636)
