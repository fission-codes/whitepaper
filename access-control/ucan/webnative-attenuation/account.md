# Account

## Resource

The resource type is `"account"`. This is a generic account resource type that can be used with any number of services that implement it.

## Service ID

The service parameter \(`"id"`\) uniquely identifies the service in question. It may be a domain name, URI, DID, blockchain address, or other identity, depending on the service.

Fission uses the domain name as identifier.

## Capabilities

At the present time, the only capability supported by Fission is `"SUPER_USER"`

### SUPER\_USER

The ability to administer the account itself: resend authentication email, close the account, and so on.

## Example

```javascript
"account": {
  "cap": "SUPER_USER",
  "id": "runfission.com"
}
```

