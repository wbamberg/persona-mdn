# Calling logout() on a failed login #

After your `onlogin` hander gets called with an assertion, if for any reason can't use the assertion to log the user in, you must call `navigator.id.logout()`.

If you don't, then the next time you call `navigator.id.watch()` Persona will immediately call your `onlogin` handler again, with the same assertion. Typically this will lead to an endless loop of failed login attempts:

1. the user clicks "Sign In"
2. the user interacts with the Persona interface, and Persona generates an assertion
3. Persona delivers the assertion to the page's `onlogin` handler
4. the `onlogin` handler rejects the assertion, and redirects the user to the login page
5. the login page loads, calls `navigator.id.watch()`, and we go back to (3)

The reason is that Persona tries to remember which email you want to use to log into a particular site. Once the user has tried to log into your site as bob@gmail.com, Persona remembers that this is the address they want to use with your site. Then when the next page load calls `navigator.id.watch()` with a `loggedInUser` of "null", Persona compares that with its value of "bob@gmail.com", and sends the assertion again.

To make Persona forget the association between your site and the email address, call `navigator.id.logout()` if you don't want to log the user in with that assertion. This might be because the assertion does not validate, or because you don't want to use the given email address.

A common scenario where this is a problem is when an RP wants to allow users to sign **in** with Persona, but does not want to let them sign **up** with Persona, preferring some custom registration system for new users. In this case, when you get an assertion, you'll check that the email address it contains is for one of your existing users, and reject the login attempt if it is not. If you do reject this assertion, you must call `navigator.id.logout()`.
