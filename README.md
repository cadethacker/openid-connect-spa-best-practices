# OpenID Connect Best Practices for SPA with Trusted Backend

*__Soooooo this is awkward.... but I've been giving out bad advice about where and how to store OAuth2 tokens on the client side__* hahaha  

So one thing I have learned in this nearly 2 year journey through OAuth2/OpenID Connect is that it is _artisanal_   and there are TONS of opinions, and sometimes it is very difficult to find good advice. Also I generally ignore any pages from prior to 2017/2018.  When you do a google search, the first 4 links will say different things.   So as of Oct 2019, here is the world as I know it.  If you are doing something different, it is doesn't mean it is bad, there are 1,000 ways todo all of this. 

### New to this whole OAuth2/OpenID Connect thingy?  

Check the Further Reading at the bottom.  There are tons of good resources for begininers, but this is a journey not a simple library you pull down and plug and run.  Security is a daily effort, not 1 feature in the backlog. Become a student of this, I promise it will pay off. 

If you are just getting started, here is an outstanding video services from Oracle, focus on videos 4-10. 

[Oracle Learning Library - Cloud Standards - Security](https://www.youtube.com/playlist?list=PLKCk3OyNwIzuD_jxWu-JddooM2yjX5q99)

### Terms

Lexicon is important, and one of the ways I see so many OpenID Connect/OAuth2 articles go sideways is they don't define the terms and so who exactly is the "client" gets squishy.  So in an effort to remove "squish" here is what I believe:

* Client -  The browser.  I guess this could also be a Mobile Browser shell around the web app (PhoneGap/Cordova), but I'll let you figure that out. 

* Resource Service - Your trusted backend.  I'm making an assumption that your front end only (mostly) talks to your backend, and your backend in turn makes downstream requests on its behalf. 

* access_token - a token representing a user. It is intended to be used by a downstream (backend) service.  It can be opaque or a jwt.  I've seen both.  You should treat it opaque, but if it is a jwt, make sure to check the expiration time. 
* id_token - by standard, it is a jwt. 


### Best Practices

*I'd love feedback so please let me know if you feel differently about any of these items* 

When building a SPA that has a trusted backend. (which most of us are). 

1. __Which Grant Type should I use?__ Use the Auth Code Grant and not the  Implicit Grant.  If you are using the Implicit Grant, that is fine, but make sure you follow all best practices.  It is easy to get wrong. Which is part of the problem.  


2. *How to get the token from the server to the browser?* If using the Auth Code Grant, then you need to return the tokens to the browser from the backend.  Use Set Cookie for that, not fragment. (again this is not a hard line, just what i'm reading as best practice, and easiest)  

* The cookie should have the full path of your domain of your app. 
* The access_token should have the HttpOnly (suggestion)
* The id_token should be available to your javascript code.(so don’t set HttpOnly)
* Neither should have a date, so that they close when the browser closes. 
* You can be fancy and delete the cookie when the tab closes, up to you. 

3. Once your javascript code kicks in, you can pull the token from the cookie and move it to memory, then delete the cookie, up to you. If you are going to pull the tokens immediately and delete them, then don’t worry about setting the HttpOnly flag. 

4. Verify (Always) and Validate (sometimes) the tokens when it makes sense. 

Use the id_token for verifying information about the human logged into your app, not the access_token. 



### Further Reading

Here are some great links I have currated:

https://tools.ietf.org/html/rfc6749

https://openid.net/connect/

https://auth0.com/docs/security/store-tokens#single-page-apps

https://blog.dareboost.com/en/2019/03/secure-cookies-secure-httponly-flags/

https://medium.com/oauth-2/why-you-should-stop-using-the-oauth-implicit-grant-2436ced1c926
