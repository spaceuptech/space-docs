---
title: "Triggering webhooks"
description: "Triggering webhooks"
date: 2020-10-22T13:28:01+05:30
draft: false
weight: 3
---

The `match` and `query` rule together with `and`/`or` rules are flexible enough to solve most complex authorization logic out there. However, in certain edge cases, we need to write custom validation logic. That's where the `webhook` rule comes into the picture.

## How it works

The basic syntax for the `webhook` rule is:

{{< highlight javascript >}}
{
  "rule": "webhook",
  "url": "<webhook-url>",
  "store": "<variable to store the response>"
}
{{< /highlight >}}

The `webhook` rule triggers an HTTP `POST` request on the URL specified in the rule. 

The target service performs custom validation and responds accordingly. The `webhook` rule is resolved, only if the webhook target responds with status code `2xx`. 

You can optionally specify the variable to store the result of the webhook so that you can use the fields from the webhook response body in your other security rules like `match`. 

### Request headers

The webhook request made by Space Cloud contains the following headers:

| Header field name | Header field value         |
|-------------------|----------------------------|
| `Content-Type`    | `application/json`         |
| `Authorization`   | `Bearer <token>`           |
| `x-sc-token`      | `Bearer <sc-access-token>` |

The `token` is nothing but the token from the client. `sc-access-token` on the other hand is a token generated by Space Cloud itself so that the webhook target can [verify whether the webhook source is Space Cloud](#verifying-webhook-source) only.

### Request body

The request body to the webhook target is a JSON object containing all the variables available inside security rules for that particular operation. 

For example, let's say that you are writing a webhook rule for database `create` operation. The variables available for the database `create` operation are `args.auth`, `args.token`, `args.doc`. Thus your webhook target, in this case, would receive a JSON body similar to this:

{{< highlight javascript >}}
{
  "args": {
    "auth": {
      ...token claims
    },
    "token": "<some-token>",
    "doc": {
      ...fields of the document to be created
    }
  }
}
{{< /highlight >}}

## Verifying webhook source

As a best practice, you should always verify that the incoming webhook request to your service is actually from Space Cloud only and not an intruder.

Space Cloud sends an access token in the headers of each webhook request to do so:

{{< highlight bash >}}
x-sc-token: Bearer <sc-access-token>
{{< /highlight >}}

The access token is generated by Space Cloud itself with the primary secret provided to it. You can check the primary secret from the `Settings` page of your project in Mission Control.

The access token contains the following claims:

{{< highlight javascript >}}
{
  "id": "<space-cloud-gateway-node-id>",
  "role": "SpaceCloud"
}
{{< /highlight >}}

To verify whether a webhook is coming from Space Cloud only, you can verify the access token with the primary secret. As an additional security step, you can also check the value of role inside the token claims.

## Forwarding the webhook results to your service

Often while securing your remote services via the `webhook` rule, you might even want to forward the result of the webhook call to your remote service. You can do that easily by using the `store` field. Just point the `store` field to any variable inside the payload (`args.params`) of the remote service.

Example:

{{< highlight javascript >}}
{
  "rule": "webhook",
  "url": "<webhook-url>",
  "store": "args.params.foo"
}
{{< /highlight >}}

`args.params` is nothing but a variable available in the security rules of remote services that contain the payload of the remote service call. (Check out the list of all available variables). Hence, storing the result in `args.params.foo` will make sure that the request payload to your remote service contains a field call `foo', which is nothing but the response of your webhook request.

## Transforming webhook requests

You can easily the transform the JWT claims and the body of the webhook request using [Go templates](https://golang.org/pkg/text/template/). Check out the [documentation of transformations](/transformations) to learn more about transformations in detail.

To specify the Go templates for JWT claims and request body, we need to configure the following fields in the `webhook` rule:

- **outputFormat:**: `yaml|json`
- **template:** `go` (Templating language. Currently only Go templates are supported.)
- **claims:** The Go template for JWT claims. If not provided, the claims will not be transformed.
- **requestTemplate:**: The Go template for the request body. If not provided, the request body will not be 

To set the templates via security rule builder, check the `Apply transformations` option and provide the output format, JWT claims template and the request body template:

![Webhook Rule Transformations](/images/screenshots/webhook-rule-transformations.png)