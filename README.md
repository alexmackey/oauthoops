# OAuth Oops (draft)

OAuth has a steep learning curve and there are many opportunities to make a mistake.

This guidance is a work in progress list contains a list of common OAuth misconfigurations and bad practices.

Many of these issues myself and colleagues have seen in the wild and some we’ve made the mistake ourselves – learn from these mistakes and build secure applications :)

## OAuth Checks

| Item    | Description |
| -------- | ------- |
| Is user sign-up enabled?  | Most identity providers have an option to enable/disable user sign-up. This should be disabled if not wanted otherwise any user could sign-up to your application. |
| Are older less secure flows enabled e.g. implicit flow | If specific OAuth flows are not required they should be disabled to reduce attack surface and ensure most secure approach used |
| Is the correct flow being used?  | OAuth offers several flows or approaches for different purposes. Some should never be used in certain situations such as Client Credentials flow for SPA applications as this will expose the secret. |
| For applications using Client Credentials flow are they sharing Client id and Secret? | Each instance should have unique client credentials client and secret so can be revoked/ rotated if needed |
| Are there any third party resources (scripts/images) referenced on OAuth flow pages and leaking credentials? | If third party resources referenced on OAuth flow pages may leak details. Use Referrer-Policy: no-referrer or meta referrer to prevent this |
| Does the application have a restrictive set of redirect urls? | Most Identity providers will enforce setting redirect urls. Redirect urls should be as restrictive as possible to prevent attacker constructing a URL that will redirect to a malicious endpoint to harvest details. |
| Is the redirect url verification safe? | Ensure valid redirect url checks match exactly and cannot be circumvented by methods such as path transversal etc |
| Is the application vulnerable to open redirect? | If application contains open redirect issue and this page is contained in valid redirect urls then an attacker could use this to redirect credentials to their server |
| Does redirect url allow use of localhost? | Sometimes for development purposes redirect urls set to localhost may be treated differently and allowed. This would allow an attacker to register a domain such as localhost.simpleisbest.co.uk to harvest request details. [Paypal (2016) suffered from this issue](https://www.bleepingcomputer.com/news/security/paypal-removes-magic-word-from-oauth-authentication-procedure/) |
| Does the application use state parameter and validate it to prevent CSRF | The state param should contain an unguessable value linked to the users session to prevent CSRF attacks. If requests dont contain state value then attacker could initiate OAuth flow themselves and trick user into completing it and performing an action. |
| Does the application request excessive permissions? | Application should request minimal permissions required to prevent abuse |

## Desktop/Mobile Applications Only