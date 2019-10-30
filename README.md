# OpenID Connect Best Practices for SPA with Trusted Backend

*Soooooo this is awkward.... but I've been giving out bad advice about where and how to store OAuth2 tokens on the client side* hahaha  

So one thing I have learned in this nearly 2 year journey through OAuth2/OpenID Connect is that it is _artisanal_   and there are TONS of opinions, and sometimes it is very difficult to find good advice.  When you do a google search, the first 4 links will say different things.   So as of Oct 2019, here is the world as I know it.  If you are doing something different, it is doesn't mean it is bad, there are 1,000 ways todo all of this. 

### Terms

Lexicon is important, and one of the ways I see so many OpenID Connect/OAuth2 articles go sideways is they don't define the terms and so who exactly is the "client" gets squishy.  So in an effort to remove "squish" here is what I believe:

* Client -  The browser.  I guess this could also be a Mobile Browser shell around the web app (PhoneGap/Cordova), but I'll let you figure that out. 

* Resource Service - Your trusted backend.  I'm making an assumption that your front end only (mostly) talks to your backend, and your backend in turn makes downstream requests on its behalf. 


### Best Practices



*Please let me know if this lines up with what you all are seeing in your teams* 

When building a SPA that has a trusted backend. (which most of us are). 

1. *Which Grant To Use?* Use the Auth Code Grant and not the  Implicit Grant.  If you are using the Implicit Grant, that is fine, but make sure you follow all of the best practices below.  


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

https://auth0.com/docs/security/store-tokens#single-page-apps

https://blog.dareboost.com/en/2019/03/secure-cookies-secure-httponly-flags/
