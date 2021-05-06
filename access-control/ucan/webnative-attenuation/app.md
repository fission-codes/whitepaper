# Web Application

## Resource

The resource key \(type\) is `”web”`. The resource value \(identifier\) is the hostname \(i.e. subdomains + domain name\).

## Capabilities

Web app capabilities are monotone, where each level “contains” the capabilities below it.

### 1. REVISE

Publish an updated version of the web app.

### 2. SUPER\_USER

> Property rights are theoretical socially-enforced constructs in economics for determining how a resource or economic good is used and owned. \[...\] the right to transfer the good to others, alter it, abandon it, or destroy it \(the right to ownership cessation\)  
>   
> — Wikipedia, [Property Rights \(economics\)](https://en.wikipedia.org/wiki/Property_rights_%28economics%29)

The ability to destroy the app, or transfer it to another owner. “Transfer" here is the transfer of ownership over the actual head pointer, as all other data can be copied with sufficient read permissions.

Includes Capability 1.

## FAQ

#### Why is there no CREATE capability?

Apps can always be created; it is not possible to restrict the ability to deploy an app, only updates to an existing app.

#### How do I create an app alias? \(e.g. alias foo.fission.app at example.com\)

In this scenario, you already have `foo.fission.app` deployed, and would like to alias that at `example.com`. This is achieved via [DNSLink](https://dnslink.io/) in a TXT record, and thus controlled at with the [`domain` resource](https://app.gitbook.com/@runfission/s/whitepaper/access-control/ucan/webnative-attenuation/domain-name).

