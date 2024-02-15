# OAuth Oops (draft)

OAuth has a steep learning curve and there are many opportunities to make a mistake.

This guidance is a work in progress list contains a list of common OAuth misconfigurations and bad practices.

Many of these issues myself and colleagues have seen in the wild and some we’ve made the mistake ourselves – learn from these mistakes and build secure applications :)

## OAuth Checks

| Item    | Description |
| -------- | ------- |
| Is user sign-up enabled?  | Most identity providers have an option to enable/disable user sign-up. 
This should be disabled if not wanted otherwise any user could sign-up to your application. |
| Are older less secure flows enabled e.g. implicit flow | If specific OAuth flows are not required they should be disabled to reduce attack surface and ensure most secure approach used |

## Desktop/Mobile Applications Only