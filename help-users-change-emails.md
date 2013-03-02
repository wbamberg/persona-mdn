# Helping users change their email address #

Because in Persona usernames are email addresses, it's not obvious how you can let your users change their email address. We're working on improving the flow for this and at the moment it is a little awkward, but it is possible, following these steps:

1. Use an identifier that's independent of the user's email address as a primary key in your internal user table. This can then stay the same when the email address changes.
2. Allow users to add email addresses to their account. You'll need to verify the email addresses, which you can do either manually, by sending an email to the new address containing a verification link, or by using Persona itself. If you use Persona to add email addresses, read "Adding extra email addresses with Persona" below.
3. Let users sign in with any email in their account.
4. Let users delete email addresses.

With these features, users can change their email address like this:

1. log into their account using their existing email address
2. add the new email to their account
3. remove the old email from their account

## Adding extra email addresses with Persona ##

If you use Persona to add additional email addresses, then you need to be aware of a couple of things: make the context of the request clear, and update the value you pass into `loggedInUser` to ensure that the transaction isn't broken by Persona's session management.

### Clarify the context of the request ###

When you request a new assertion using with either the old `navigator.id.get()` API or the `navigator.id.request()` API, Persona expects that the user is trying to sign into a website, and the user interface it displays reflects that. If you are using Persona just to get a new verified email address, your site needs to make this clear to users, so they are not confused by the Persona dialog.

### Update loggedInUser ###

If you're using the `navigator.id.get()` API in the rest of your site, then you can just make a new `navigator.id.get()` call to get the extra email address.

But if you use `navigator.id.request()`, then you must also use `navigator.id.request()` to get the extra email address. If you do this then when you have verified the assertion inside your `onlogin` handler, you must update the `loggedInUser` argument to `navigator.id.watch()` with the new email address.

If you don't do this, then there will be a mismatch: Persona will think the logged in user is `new_emailaddress@example.org`, but your website will be telling it that the logged in user is `old_emailaddress@example.org`. In response, Persona will fire `onlogin` with an assertion for `new_emailaddress@example.org`, which your website will probably interpret as a new user signing up.

