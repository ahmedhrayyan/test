import FlowChart from '../assets/sso-flowchart.svg'

---
title: Single Sign-On
---

## Introduction
We have implemented Single Sign-On (SSO) for our application utilizing VisitSaudi as our OpenID Provider (OP). This document explains the flow of the SSO implementation and the API endpoints used.

First, you need to be familiar with the following:

- [OAuth 2.0](https://oauth.net/2/)
- [OpenID Connect (OIDC)](https://openid.net/connect/) Protocol. More specifically, the [Authorization Code Flow](https://openid.net/specs/openid-connect-core-1_0.html#CodeFlowAuth)


## Abstract
The application uses the Authorization Code Flow to authenticate users. Check out the below diagram to understand the flow:

<FlowChart style={{width: "100%", height: "auto"}} />

## API Documentation
In our flow, there is three applications responsible for the SSO:
* The OpenID Provider (OP): VisitSaudi
* The Resource Server: Our Backend API
* The Relying Party (RP): Our Next.js Frontend

### OP Documentation
You can find thorough documentation on how to integrate with **VisitSaudi** in the following document:
<object data={require("../assets/Saudi_Smart_ID_OIDC_Integration_Guide_V1.pdf").default} type="application/pdf" width="100%" height="500px">
  <p>Unable to display PDF file. <a href={require("../assets/Saudi_Smart_ID_OIDC_Integration_Guide_V1.pdf").default}>Download</a> instead.</p>
</object>

### Backend API Docs
The Resource Server is our backend API. You can find the API endpoints related to SSO [here](https://stg.api.dalila.tam.codes/swagger-ui/index.html#/o-auth-2-controller).
### RP (Frontend) API Docs
- `/login`: Constructs the authorization URL and redirects the user to the OP.
- `/login/sso`: Receives the authorization `code` from the OP (along with the `state`) as query parameter and exchanges it for an access token.
- `/logout`: Logs out the user from the application and the OP.

## Implementation
The following sections explain the implementation of the SSO flow in our application.

### Login
#### Flow:
1. The user clicks on the login button in the frontend.
2. The frontend redirects the user to the `/login` endpoint passing it `from` query parameter which presents the current pathname.
3. The `/login` endpoint constructs the authorization URL and redirects the user to the OP.
4. The user logs in and the OP redirects the user back to the `/login/sso` endpoint with the `code` and `state` query parameters.
5. The `/login/sso` endpoint exchanges the `code` for an access token and stores it in the session cookie.
6. The user is redirected back to the page they were on before the login (the value of state).
#### Design Choices:
* The `login` endpoint is passing the `from` query parameter to the OP as `state` query parameter to redirect the user back to the page they were on before the login. This is not ideal as the `state` query parameter is used for CSRF protection. We will change this in the future to use something like the one described [here](https://auth0.com/docs/protocols/oauth2/oauth-state).
* Upon successful exchange of the authorization code for an access token, we are storing the access token in the httpOnly cookie `sesssion` along with the `refresh_token` and `expires_in` and user internal id.
* We are extracting `expires_in` from the access token payload and we are storing it in the cookie to not decode the access token every time we need to check if the token is expired or not.
* We are storing the user internal id in the session cookie to be able to identify the user in the resource server as it is different from the `sub` claim in the access token.

### Refresh Token
#### Points to Note:
* The refresh token can be used only once to get a new access token. After that, the refresh token is invalidated and a new refresh token is issued.
* We cannot change cookies values in server components, We can only set cookies in server actions or route handlers (endpoints).
#### Flow:
1. Before making a request to the resource server, the frontend checks if the access token is expired or not.
2. If the access token is expired, the frontend sends a request a new access token using the refresh token.
3. Upon successful refresh, the frontend stores the new access token and the new refresh token in the session cookie.
4. The frontend makes the request to the resource server with the new access token.
#### Design Choices:
The check for access token expiration and update is done in two places:

**1. In `middleware.js`**

We cannot change the cookies values in server or client components In `Next.js` In addition there may be more than one request to the resource server in the same page that if token is expired will attempt to use the same `refresh_token` to request a new token which will fail.

So we are performing the check in the `middleware.js` file which is a middleware function that is executed before the page is rendered. This way we can check if the token is expired and refresh it before the page is rendered.

We also take this step further by checking if the token will expire in the next 15 minutes and if so we will refresh it. This is to avoid the case where the token expires while the user is on the page and the user tries to make a request to the resource server.

**2. In `appFetch`**

The middleware function is not enough as it run only on rendering components but it does not run on server actions and user may stay on the page for a long time and the token may expire. So we are also checking if the token is expired in the `appFetch` function which is a wrapper around the `fetch` function. This way we can check if the token is expired before making the request to the resource server for server actions (and we can update cookies in server actions).

This can lead to race conditions where multiple requests (server actions) are made to the resource server at the same time and all of them will check if the token is expired and try to refresh it. We are not handling this case as it is a rare case and it is not worth the complexity to handle it.

## Handling errors
The following sections explain how we handle errors in our SSO flow.
### 401 Unauthorized
#### Points to Note:
* The access and refresh token can be invalidated by the OP at any time.
* The refresh token can expire after a certain period of time.
* The resource server will return a `401 Unauthorized` status code if the access token is invalid or expired.

#### Flow:
1. The frontend makes a request to the resource server.
2. The resource server returns a `401 Unauthorized` status code.
3. The frontend will clear the session cookie and redirect the user to the `/session-expired` page to inform the user that the session has expired.
4. The user can click on the login button to login again.

#### Design Choices:
* We are redirecting the user to the `/session-expired` page instead of showing a popup window with the session expired message as we are not able to show a popup window in the server components.
* If the user closes the popup window which shows the session expired message, We are redirecting him to the homepage as a guest user.
* User can access the `/session-expired` anytime by typing the URL in the browser. We are not performing any kind of check as it is not necessary.

### Refresh Token Error
We are handling the refresh token error the same way we handle `401 Unauthorized` status code. We are redirecting the user to the `/session-expired` page and the user can click on the login button to login again.
