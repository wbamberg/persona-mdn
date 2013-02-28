If you implement a particular type of protection against CSRF (Cross-Site Request Forgery) login attacks, you will experience a problem when a user has multiple pages open on your site, and then tries to log in using one of them. This document explains what the problem is, and how to resolve it.

## CSRF and CSRF login attacks ##

In a normal CSRF attack, the user has already logged into the target website (for example, their bank). The user then loads a maliciously-constructed page on another site. This page sends an HTTP request to the target website: the request has some valuable payoff (for example, it transfers money to the attacker). The target website allows the request because it assumes that since the user is logged into the site, then the request was made by the user.

In a CSRF login attack, the user loads a maliciously-constructed page that logs the user into the target site **as the attacker**: the target site then sets a session cookie in the user's browser. The attacker is able to access the account later, and retrieve any information that the site collected about that user.

## Protection against CSRF: CSRF tokens derived from session IDs ##

A common protection against normal CSRF attacks is for websites to derive a token from the browser's session ID, and embed it in the pages they serve: then POST requests have to include the token, which is checked on the server. This means POST request can't be made from other domains, because they can't get access to the token used to authenticate them.

With CSRF **login** attacks, of course, the user isn't logged into the site already, so the website sets up an initial session as soon as the user visits the page. This is used to generate the CSRF token until the user logs in. Once the user logs in, a new session ID is generated, and the CSRF regenerated accordingly.

## The problem with Persona ##

The problem with Persona arises when the user is not yet logged in, and has loaded pages from the RP's site in two separate tabs, A and B. The pages contain the same CSRF token, derived from the initial session ID. Then the user logs in from tab A, and the website generates a new session ID, and thus a new CSRF token. But tab B still has the old page loaded, containing the old, now invalid, token.

Under normal circumstances this would not usually matter: as soon tab B reloads the page, or the user navigates to a new page, then the embedded CSRF token is updated. But when Persona has generated an assertion in response to a call to `navigator.id.request()`, Persona calls `onlogin` for every tab which has that website loaded, not just the single tab that requested the assertion. As soon as tab B's `onlogin` handler is called, it attempts to POST the assertion to the server using the old CSRF token - and the server raises a CSRF error.

## The solution ##

The solution is for the `onlogin` handler to send a GET request which fetches a fresh CSRF token and checks whether the user is already logged in. If the user is already logged in, then the handler just has to reload the page. Otherwise it uses the new CSRF token to POST the assertion to the server.
