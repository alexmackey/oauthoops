# :scream_cat: OAuth Oops (draft) :scream_cat:

OAuth has a steep learning curve and there are many opportunities to make a mistake.

This guidance is a work in progress and contains a list of common OAuth related mistakes and bad practices.

This guide focuses on OAuth v2.1​ and friends. These friends have their own distinct standards and include OpenID, JWT (JSON Web Token or RFC 7519), JWS (JSON Web Signatures)​ and JOSE (JavaScript Object Signing and Encryption). Issues impacting these are included as they are often found as part of OAuth implementations. 

Many of these mistakes myself and colleagues have seen in the wild and some we’ve made ourselves – let's learn from these issues and build secure applications :smile:

## Support

Thanks to my employer [Kodez](https://www.kodez.com.au) for support in putting this together and please reach out to Kodez for assistance with Identity, Development or AppSec.

## Contribution

This article is in draft status and looking for feedback - PR's welcome.

## OAuth Checks

| Item    | Description |
| -------- | ------- |
| Is user sign-up enabled?  | Most identity providers have an option to enable/disable user sign-up. This should be disabled if not wanted otherwise any user could sign-up to your application |
| Are older less secure flows enabled e.g. implicit flow? | If specific OAuth flows are not required they should be disabled to reduce attack surface and ensure most secure approach used |
| Is the correct flow being used?  | OAuth offers several flows or approaches for different purposes. Some should never be used in certain situations such as Client Credentials flow for SPA applications as this flow requires a client secret that would be exposed |
| For applications using Client Credentials flow are they sharing Client id and Secret? | Each instance should have unique client credentials client and secret so can be revoked/ rotated if needed |
| Are there any third party resources (scripts/images) referenced on OAuth flow pages and leaking credentials? | If third party resources referenced on OAuth flow pages may leak details. Use Referrer-Policy: no-referrer or meta referrer to prevent this |
| Does the application have a restrictive set of redirect urls? | Most Identity providers will enforce setting allowed redirect urls. Redirect urls should be as restrictive as possible to prevent attacker constructing a URL that will redirect to a malicious endpoint that harvests details |
| Is the redirect url verification safe? | Ensure allowed redirect url checkscannot be circumvented by methods such as path transversal etc |
| Is the application vulnerable to open redirect? | If application contains open redirect issue and this page is contained in valid redirect urls then an attacker could use this to redirect credentials to their server |
| Does redirect url allow use of localhost? | Sometimes for development purposes redirect urls set to localhost may be treated differently and allowed. This would allow an attacker to register a domain such as localhost.simpleisbest.co.uk to harvest request details. [Paypal (2016) suffered from this issue](https://www.bleepingcomputer.com/news/security/paypal-removes-magic-word-from-oauth-authentication-procedure/) |
| Does the application use state parameter and validate it to prevent CSRF? | The state param should contain an unguessable value linked to the users session to prevent CSRF attacks. If requests dont contain state value then attacker could initiate OAuth flow themselves and trick user into completing it and performing an action |
| Does the application request excessive permissions? | Application should request minimal permissions required to prevent potential abuse |


## Desktop or Mobile Applications Checks

| Item | Description |
| -------- | ------- |
| Is the native or mobile application using Client Credentials Flow or have embedded secrets? | Client Credentials Flow should not be used for mobile applications as client secret could be extracted from application. This issue occurred with [Twitter Mobile app, 2010](https://arstechnica.com/information-technology/2010/09/twitter-a-case-study-on-how-to-do-oauth-wrong/2/) |
| Does the application use the devices browser for login? | Mobile applications should use Authorization Code Flow with PKCE with the devices browser. Providing the login screen in the application could allow application to grab username and password. Using the devices browser allows the user to benefit from browser security controls and extensions as per [rfc8252](https://datatracker.ietf.org/doc/html/rfc8252) |


## User Account Handling Checks

| Item | Description |
| -------- | ------- |
| Is email/mobile ownership verified in application? | Without validating ownership of email/mobile number attacker may be able to take over accounts by creating a valid login and then updating email or mobile to victims details |
| Can you chage email address or mobile be changed without verification? | Without validating ownership of email/mobile number attacker may be able to take over accounts by creating a valid login and then updating email or mobile to victims details |
| Can you sign up for legitimate user address without verification and then have it linked to social account? | Some applications may allow users to link account to social login e.g. Facebook and could allow account take over via user registering valid account and then linking to Social account |
| Is email being used to link identity provider record and application record? | Email is a bad choice of fields to link between identity provider/application as email can be re-assigned, names change etc. Use a long alpha numeric non-predictable ID |

## Federated Identity Checks

| Item | Description |
| -------- | ------- |
| Does the federated identity provider(s) validate email/mobile ownership? | Without validation of email/mobile ownership there is potential for account takeover by creating user in external identity provider |
| Does application validate the identity provider user is from? | Application should validate which identity provider has issued token for a user to prevent malicious external identity provider masquerading as a different identity providers user. Additionally, watch out for ability to specify multiple email addresses on user account that could be abused |


## Token/JWT checks

| Item | Description |
| -------- | ------- |
| Are framework/library methods used to validate token? | Where possible tried and tested framework or libraries should be used to check tokens. Ensure methods are not just checking token structure or syntax but perform validation of Issuer, Audience, Expiry etc |
| Is token using default signing key? | Some frameworks may contain a default signing key for tokens. This should be changed to prevent attacker minting their own tokens |
| Is application checking only token Issuer (iss)? |If application is only checking Issuer (iss) for public identity provider then anyone can create a an identity tenant that will issue tokens the application will trust. Note this issue has been found several times so believe this is a common mistake |
| Is token Audience checked? | Services should verify a tokens audience (aud) to ensure the token is intended for them. Note this issue has been found several times so believe this is a common mistake |
| Is token using strong signing key? | Without strong signing key tokens could potentially be minted by brute force of signing key |
| Is the token signing key accessible e.g. via LFI (local file inclusion)? | Ensure signing key securely stored and not accessible in web root to prevent attacker minting tokens |
| Are secrets stored in tokens? | Tokens are base 64 URL encoded and easily visible through services such as [jwt.io](https://jwt.io) and [jwt.ms/](https://jwt.ms). Do not store plaintext secrets in them (and ideally no secrets at all) |
| Are tokens stored in application/logs? | Tokens provide access to resources and where possible should not be stored. See Okta breach where [support ticket system contained customer tokens](https://www.malwarebytes.com/blog/news/2023/11/okta-breach-happened-after-employee-logged-into-personal-google-account) |
| Does the token have an excessive lifetime configured? | The longer the lifetime of the token the more likely it is to be intercepted/leaked. Applications should balance token life time with security. Ideally refresh tokens should be used |
| Are tokens sent over secure connection? | Tokens should be sent over secure connection to prevent interception |
| Are cookies containing tokens using HTTP only, SameSite (strict) flags? | If cookies used to store tokens make use of browser security mechanisms to prevent accidental leakage and XSS attacks |
| Does application enforce authorization checks that prevent requested scope being upgraded? | Application may issue token with limited scope but if attacker adds additional scope parameter to code/token exchange request that server doesnt check permissions for may grant token with additional scope |

## Microsoft AzureB2C/AD/Entra Specific Checks

This section requires additional input as is a huge area.

| Item | Description |
| -------- | ------- |
| Is application registered as any organisation directory? | If application is registered to allow any organisation directory then any user could set up AD to access application. See [Bing Bang](https://www.wiz.io/blog/azure-active-directory-bing-misconfiguration) |
| Is application using organisational AD? | If application is using organisational AD then Office 365 guest users (e.g. that have O365 documents shared with them) may have valid token |


## Okta/Auth0 Specific Checks

| Item | Description |
| -------- | ------- |
| Are attack protections enabled? | Auth0 contains various [attack protections](https://auth0.com/docs/secure/attack-protection) that can be enabled to prevent issues such as brute force attacks, suspicious logins, use of breached credentials  |
| Are secrets logged? | Ensure no secrets or tokens are logged in Auth0 e.g. from actions |
| Are any Actions, Rules or Hooks using vulnerable dependancies? | Ensure no vulnerable NPM dependancies referenced |
| Is user_metadata used to store values user should not be able to set? | The property user_metadata can be modified by users and should [not be used as a secure data store](https://auth0.com/docs/manage-users/user-accounts/metadata). User app_metadata should be used instead. |
| For database connections is Password Strength enforced? | Enforcing more complex passwords will help protect against bruteforce attacks |
| Is Password History used? | Preventing password reuse may protect against credential stuffing and bruteforce attacks |
| Is Password Dictionary option enabled? | Password dictionary prevents use of [common passwords](https://github.com/danielmiessler/SecLists/blob/master/Passwords/Common-Credentials/10k-most-common.txt) and can help protect against brute force attacks |
| Is Disallow Personal Data option enabled | Preventing use of personal data in passwords can assist with creation of stronger passwords |
| Is the administrative email up to date/monitored? | Ensure email address is up to date and monitored as may receive alerts |
| Are logs monitoried/automated alerts setup? | Logs should be actively monitored for potential attacks |



## Informational Checks

| Item | Description |
| -------- | ------- |
| What identity provider(s) are used? | Determine the identity providers in use – there may be multiple and social login (Facebook etc) may be enabled |
| Are meta data endpoints accessible? | Many identity providers will provide meta data about their configuration that can be read e.g. /.well-known/oauth-authorization-server /.well-known/openid-configuration |
| What OAuth flows are enabled? | Some identity providers will allow you to modify request parameters to use different flows |
| Are secrets contained in HTML/JS | Ensure no secrets such as Client Secret etc are stored in HTML and JS files |

## References/Further reading

With thanks to the following resources:

* [Portswigger OAuth labs - free to use!](https://portswigger.net/web-security/oauth)
* [Hacktricks OAuth Account Takeover](https://book.hacktricks.xyz/pentesting-web/oauth-to-account-takeover)
* [Implementing Authorization in Web Applications and APIs - Brock Allen & Dominick Baier](https://www.youtube.com/watch?v=-a7JbXL0hq0)
* [OpenID Connect & OAuth 2.0 – Security Best Practices - Dominick Baier](https://www.youtube.com/watch?v=jeRALmfyoqg)
* [OWASP Web Application Security Testing OAuth Weaknesses](https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/05-Authorization_Testing/05-Testing_for_OAuth_Weaknesses)

:scream_cat: :scream_cat: :scream_cat: