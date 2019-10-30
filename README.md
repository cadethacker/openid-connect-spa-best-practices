# OpenID Connect Best Practices for SPA with Trusted Backend

*__Soooooo this is awkward.... but I've been giving out bad advice about where and how to store OAuth2/OpenID tokens on the client side__* hahaha  

So one thing I have learned in a nearly 2 year journey of using OAuth2/OpenID Connect at enterprise scale is that

> .....it is _artisanal_   

When searching google, there are TONS of opinions. It is very difficult to find solid, consistent advice. Also I generally ignore any pages from prior to 2017/2018.  When searching, the first 4 links will say 7 different things.  :D So as of Oct 2019, here is the world as I know it.  If you are doing something different, it is doesn't mean it is bad, there are 1,000 ways todo all of this, but here is my currated knowlege.  Hopefully it helps somebody.  If so, give this a star above :D 

### New to this whole OAuth2/OpenID Connect thingy?  

Check the Further Reading at the bottom.  There are tons of good resources for begininers, but this is a journey and not a simple library you pull down and plug and run.  Security is a daily effort, not 1 feature in the backlog. Become a student of this, I promise it will pay off. 

If you are just getting started with OAuth2 and OpenID Connect, here is an outstanding video services from Oracle, focus on videos 4-10. 

[Oracle Learning Library - Cloud Standards - Security](https://www.youtube.com/playlist?list=PLKCk3OyNwIzuD_jxWu-JddooM2yjX5q99)

### Terms

Lexicon is important, and one of the ways I see so many OpenID Connect/OAuth2 articles go sideways is they don't define the terms and so who exactly is the "client" gets squishy.  So in an effort to remove "squish" here is what I believe:

* Client -  The browser. Period.

* Resource Service - Your trusted backend.  I'm making an assumption that your front end only (mostly) talks to your backend, and your backend in turn makes downstream requests on its behalf. 

* access_token - a token representing a user authorization.  It can be opaque or a jwt.  I've seen both.  The access_token is used for authorizing a call to a resource service. 

* id_token - by standard, it is a jwt. (Yeah!) This is a jwt that should contain all of the user's public data: first name, last name, scopes, email, etc. 

* auth_code - a 1 time use token to retrieve the access_token and the id_token. 

* refresh_token - a long time use token that you store on the server to get a new access token on demand.  


### Best Practices

*I'd love feedback so please let me know if you feel differently about any of these items* 

When building a SPA that has a trusted backend. (which most of us are). 

1. __Which Grant Type should I use?__ Use the Auth Code Grant and not the  Implicit Grant. Ask for the openid scope so you get an id_token.  If you are using the Implicit Grant, you should make plans to remediate.  See Further Reading below. The Implicit Grant is easy to get wrong. Which is part of the problem.  

2. __Should I use Sessions?__ This is a key questions and requires careful thought. Regardless of using sessions or not, the id_token goes to the browser. So this really impacts the access_token and refresh_token. 

Option 1: Use Session - If you do this, the id_token goes to the browser, and the id_token, access_token and refresh_token go into the session. 

PRO: This is probably the most secure stance for the access_token and refresh_token. There are many good sessions libraries that have been hardened and battle tested. 

CON: If you scale the backend beyond 1 instance, you have to take a hard look at proper session backing cache like redis or a database. This adds operational complexity to your app stack. 

Option 2: Do not use Session - If you do this, the id_token and access_token go to the browser, and the refresh_token gets dropped and ignored. This should *NEVER* to to the browser, ever ever ever. :D 

PRO: This is simpler app setup and doesn't require a persistant session backing database. 

CON: You loose your refresh_token and have have to have proper security posture for holding the access_token in the browser. 

2. __Post Auth Code flow... How to get the tokens from the server to the browser?__ If using the Auth Code Grant, then you need to return the tokens from the trusted backend to the browser. 

If using sessions, you need to return the id_token.  If not using sessions, then you need to return both id_token and access_token.  

The easiest way is to use Set-Cookie for that, not fragment. Don't get attached to the cookie, once it gets to the browser, you could pull the value from the cookie and delete the cookie, more on that in a moment. 

To be safe: 

* The cookie should have the __FULL PATH__ of your application domain. 
* Neither should have a date, so that they expire when the browser closes. 

3. __Where do I store the token once my javascript code reactivates after the auth flow finishes?__

Once your javascript code kicks back in post auth flow, the safest option is to pull the token(s) from the cookie(s) and move it to the application memory, then delete the cookie(s). 

*DO NOT* store the tokens in Web (Local or Session) Storage.  Those should be treated as open cache and not security.  Those storage options are open to *all* javascript loaded in your app.  Go count the number of external javascript libraries you import.  Do you trust all of them?  Probably not. 

PRO: By moving the id_token (and maybe access_token as well) into memory, are become a hard target for XSS or CSER attacks because the token is not where attackers expect it.  

4. __What do I do with the tokens once I have them?__ 

1. Use a good client side JS library to open the JWT.  This https://github.com/auth0/jwt-decode good one.  It will open the JWT for you, and does very light validation. 

2. Use the id_token for verifying information about the human logged into your app, not the access_token. You can verify if it is modifed, so great for maininting authorizations inside the browser without making a round trip. 

My personal opinion (and could easily be argued different ways) is that OpenID Connect Level 1 is to set the id_token and access_token for a work day (8 hours) and throw away the refresh token.  But if you set the id_token/access_token for shorter period of time (say 15 minutes) then you will need to store the refresh token (which should be set for 8 hours) on the server session so your user can refresh their token on demand.  That is a OpenID Connect Level 2 topic.  Start with Level 1 and work up to Level 2. 

5. __How do I make authorized calls to my backend?__ 

First point, once the id_token gets to your browser, it should stay there. This is for your javasript code, and should not be passed around. If you want to validate it, then write a quick endpoint on your backend and pass it back to validate it, but other than that call, the id_token stays put. 

For the access_token you have two options:

* If you left the access_token as a cookie, or even left it with the HttpOnly flag then it will just go along with all backend calls and you can check for that cookie on the backend call. 

* If you left the access_token as a cookie or you moved the access_token to memory, you can pull the value before any `fetch` call and put it as a header on the request: `Authorization: Bearer my.awesome.jwt`.  This is a good practice and most services expect this format. 


### Further Reading

Here are some great links I have currated:

[1] https://tools.ietf.org/html/rfc6749

[2] https://openid.net/connect/

[3] https://medium.com/@robert.broeckelmann/when-to-use-which-oauth2-grants-and-oidc-flows-ec6a5c00d864

[4] https://auth0.com/docs/security/store-tokens#single-page-apps

[5] https://blog.dareboost.com/en/2019/03/secure-cookies-secure-httponly-flags/

https://medium.com/oauth-2/why-you-should-stop-using-the-oauth-implicit-grant-2436ced1c926
