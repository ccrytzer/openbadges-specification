---
title: Open Badges Technical Specification
subtitle: Awesome Sauce for the 21st Century
show_sidebar: true
---
# {{ page.title }}
This specification describes a method for packaging information about accomplishments, embedding it into portable image files as digital badges, and establishing an infrastructure for its validation. 

This specification includes term definitions for representations of data in Open Badges. These term definitions appear in the current [JSON-LD context (_v1.1_)](https://w3id.org/openbadges/v1)  for the Open Badges Standard.

#### Status of this Document
This document represents the current version of the Open Badges Specification. 

> **Current version of the Open Badges specification: 1.1**
- Published: 1 May 2015.

#### License
[Copyright](http://www.w3.org/Consortium/Legal/ipr-notice#Copyright) © 2014, 2015 the Contributors to the Open Badges 1.1 Specification, published by the [Badge Alliance](http://badgealliance.org) under the [W3C Community Contributor License Agreement (CLA)](https://www.w3.org/community/about/agreements/cla/). A human-readable [summary](http://www.w3.org/community/about/agreements/cla-deed/) of this license is available. Though the Badge Alliance collaborates with the [W3C Credentials Community Group](http://www.w3.org/community/credentials/), the Open Badges specification is not endorsed as a web standard by the W3C. This specification is free for anyone to use or implement.

**Edited by the [Badge Alliance Standard Working Group](http://www.badgealliance.org/open-badges-standard/)**

## Contents
 * [Badge Objects](#BadgeObjects)
   - [Assertion](#Assertion)
   - [BadgeClass](#BadgeClass)
   - [Issuer](#Issuer)
   - [Extensions](#Extensions)
 * [Implementation](#Implementation)
   - [Hosted verification](#HostedBadge)
   - [Signed verification](#SignedBadge)
 * [Other Resources](#OtherResources)
   - [Validator](#Validator)
   - [History/Changelog](#History)

## Badge Objects <a id="BadgeObjects"></a>
The Open Badges specification is made up of three types of core Badge Objects: **[Assertions](#Assertion)**, **[BadgeClasses](#BadgeClass)**, and **[Issuers](#Issuer)**. A set of one of each of these may be constructed into a valid Open Badge. Each Badge Object is a collection of properties and values. Each type of Badge Object has its own list of mandatory and optional properties as well as restrictions on the values those properties may take. They are published as JSON for interoperability. Since 1.1, Open Badges must be valid [JSON-LD](http://www.w3.org/TR/json-ld/).

#### Extensions
_since: 1.1_ 
Each Badge Object may have additional properties beyond those defined here. Some of these additional properties may take the form of an Open Badges Extension, a structure that follows a standard format for collaboratively extending Badge Objects so that any issuer, earner, or consumer can understand the information added to badges. ([More...](#Extensions))

#### Badge Baking
_since: 0.5_
Badge assertions may be "baked" into image files as portable credentials. Baking is currently supported for PNG and SVG formats. (See [Baking Specification](baking) for implementation)

#### Hosted Verification
_since: 0.5_
A hosted assertion is a file containing a well-formatted badge assertion in JSON served with the content-type `application/json`. This should live at a stable URL on your server (for example, [https://example.org/beths-robotics-badge.json](https://example.org/beths-robotics-badge.json)) -- it is the source of truth for the badge and any future verification attempt will hit that URL to make sure the badge exists and was issued by you. ([More...](#HostedBadge))

#### Signed Verification
_since: 1.0_
A signed badge is in the form of a [JSON Web Signature](http://self-issued.info/docs/draft-ietf-jose-json-web-signature.html). Signed badges use the Backpack javascript [issuer API](https://github.com/mozilla/openbadges/wiki/Issuer-API) to pass a signature rather than a URL. The JSON representation of the badge assertion should be used as the JWS payload. ([More...](#SignedBadge))


## Assertion <a id="Assertion"></a> ([example](./examples/#Assertion))
Assertions are representations of an awarded badge, used to share information about a badge belonging to one earner. Assertions are packaged for transmission as JSON objects with a set of mandatory and optional properties. Fields marked **in bold letters** are mandatory.

{::options parse_block_html="true" /}
<div class="table-wrapper">

| Property | Expected Type | Description |
| -------- | ------------- | ----------- |
| **@context** | JSON-LD Context | `https://w3id.org/openbadges/v1` or valid JSON-LD context array or object including the 1.1 Open Badges Context
| id | URL | Unique IRI for the Assertion. If using hosted verification, this should be the URL where the assertion is accessible.
| **type** | JSON-LD type | valid JSON-LD representation of the Assertion type. In most cases, this will simply be the string `Assertion`. An array including `Assertion` and other string elements that are either URLs or compact IRIs within the current context are allowed.
| <a id="uid"></a>**uid** | Text | Unique Identifier for the badge. This is expected to be *locally* unique on a per-origin basis, not globally unique. |
| <a id="recipient"></a>**recipient** | [IdentityObject](#IdentityObject) | The recipient of the achievement. |
| **badge** | URL | URL that describes the type of badge being awarded. The endpoint should be a [BadgeClass](#badgeclass) |
| **verify** | [VerificationObject](#VerificationObject) | Instructions for third parties to verify this assertion. |
| <a id="issueDate"></a>**issuedOn** | [DateTime](#dateTime) | Date that the achievement was awarded. |
| image | URL | URL of an image representing this user's achievement. This must be a PNG or SVG image, and should be prepared via the [Baking specification](https://github.com/mozilla/openbadges/wiki/Badge-Baking). An 'unbaked' image for the badge is defined in the [BadgeClass](#BadgeClass) |
| <a id="evidence"></a>evidence | URL | URL of the work that the recipient did to earn the achievement. This can be a page that links out to other pages if linking directly to the work is infeasible. |
| <a id="expirationDate"></a>expires | [DateTime](#dateTime) | If the achievement has some notion of expiry, this indicates a date when a badge should no longer be considered valid. |

</div>


#### <a id="IdentityObject"></a>IdentityObject
A collection of information about the recipient of a badge.

{::options parse_block_html="true" /}
<div class="table-wrapper">

Property | Expected Type | Description
--------|------------|-----------
**identity** | [identityHash](#identityHash) or Text | Either the hash of the identity or the plaintext value. If it's possible that the plaintext transmission and storage of the identity value would leak personally identifiable information where there is an expectation of privacy, it is strongly recommended that an IdentityHash be used.
**type** | [IdentityType](#identitytype) | The type of identity.
<a id="hashed"></a>**hashed** | Boolean | Whether or not the `identity` value is hashed.
<a id="salt"></a>salt | Text | If the recipient is hashed, this should contain the string used to salt the hash. If this value is not provided, it should be assumed that the hash was not salted.

</div>

#### <a id="VerificationObject"></a>VerificationObject
A collection of information allowing a consumer to authenticate the Assertion.

{::options parse_block_html="true" /}
<div class="table-wrapper">

Property | Expected Type | Description
--------|------------|-----------
**type** | VerificationType | The type of verification method.
**url** | URL | If the type is "hosted", this should be a URL pointing to the assertion on the issuer's server. If the type is "signed", this should be a link to the issuer's public key.

</div>

## <a id="BadgeClass"></a>BadgeClass ([example](./examples/#BadgeClass))
A collection of information about the accomplishment recognized by the Open Badge. Many assertions may be created corresponding to one BadgeClass

{::options parse_block_html="true" /}
<div class="table-wrapper">

Property | Expected Type | Description
--------|------------|-----------
**@context** | JSON-LD Context | `https://w3id.org/openbadges/v1` or valid JSON-LD context array or object including the 1.1 Open Badges Context
**id** | URL | Unique IRI for the BadgeClass, the URL where it is published.
**type** | JSON-LD type | valid JSON-LD representation of the BadgeClass type. In most cases, this will simply be the string `BadgeClass`. An array including `BadgeClass` and other string elements that are either URLs or compact IRIs within the current context are allowed.
**name** | Text | The name of the achievement.
**description** | Text | A short description of the achievement.
**image** | [Data URI](http://en.wikipedia.org/wiki/Data_URI_scheme) or URL | URL of an image representing the achievement.
<a id="Criteria"></a>**criteria** | URL | URL of the criteria for earning the achievement. If the badge represents an educational achievement, consider marking up this up with [LRMI](http://www.lrmi.net/)
**issuer** | URL | URL of the organization that issued the badge. Endpoint should be an [IssuerOrganization](#Issuer)
alignment | Array of [AlignmentObject](#Alignment)s | List of objects describing which educational standards this badge aligns to, if any.
<a id="Tags"></a>tags | Array of Text | List of tags that describe the type of achievement.

</div>

#### <a id="Alignment"></a>AlignmentObject

{::options parse_block_html="true" /}
<div class="table-wrapper">

Property | Expected Type | Description
--------|------------|-----------
**name** | Text | Name of the alignment.
**url** | URL | URL linking to the official description of the standard.
description | Text | Short description of the standard

</div>

## <a id="Issuer"></a>IssuerOrganization ([example](./examples/#Issuer))
A collection of information about the entity or organization issuing the Open Badge. Each issuer may correspond to many BadgeClasses. Anyone can create and host an Issuer file to start issuing Open Badges.

{::options parse_block_html="true" /}
<div class="table-wrapper">

Property | Expected Type | Description
--------|------------|-----------
**@context** | JSON-LD Context | `https://w3id.org/openbadges/v1` or valid JSON-LD context array or object including the 1.1 Open Badges Context
**id** | URL | Unique IRI for the hosted IssuerOrganization file.
**type** | JSON-LD type | valid JSON-LD representation of the Issuer type. In most cases, this will simply be the string `IssuerOrg`. An array including `IssuerOrg` and other string elements that are either URLs or compact IRIs within the current context are allowed.
<a id="name"></a>**name** | Text | The name of the issuing organization.
**url** | URL | URL of the institution
<a id="description"></a>description | Text | A short description of the institution
<a id="image"></a>image | [Data URI](http://en.wikipedia.org/wiki/Data_URI_scheme) or URL | An image representing the institution
<a id="email"></a>email | Text | Contact address for someone at the organization.
<a id="RevocationList"></a>revocationList | URL |  URL of the Badge Revocation List. The endpoint should be a JSON representation of an object where the keys are the `uid` or `id` of a revoked badge assertion, and the values are the reason for revocation. It is only recommended that signed badge issuers use this method when they have revoked at least one badge.

</div>

## [Extensions](./extensions/) <a id="Extension"></a>
_since 1.1_

The 1.1 version of the Open Badges Specification introduces Extensions as a means for issuers to add additional metadata to Badge Objects beyond what the standard specifies itself. Additional properties are allowed without using Extensions, but Extensions allow issuers to declare how they are adding information so that it can be understood by others and other issuers can add the same sort of information in a compatible way. See the [Extensions](./extensions/) page for specific examples and extensions ready to use in Badge Objects.

Extension authors define and host a new [JSON-LD](http://json-ld.org) context file describing all the terms the extension covers. These context files may further define any [JSON-schema](http://json-schema.org/) that implementations of the extension should pass. If used, each schema is linked from the context and hosted as a separate JSON-schema files. Extensions are implemented in Open Badges as JSON objects inside an Assertion, BadgeClass or Issuer with their own link to the extension context and declaration of type.

{::options parse_block_html="true" /}
<div class="table-wrapper">

Property | Expected Type | Description
--------|------------|-----------
**@context** | URL | JSON-LD context file shared among all implementations of the extension.
**type** | array of IRIs | IRIs or compact IRIs within the OBI or extension context that describe the type of data contained in the extension. These are used to map optional JSON-schema validation to the extension. Must include 'extension' as one element.
\*anyProperties | Any | Any property names defined in the extension context may be used with any valid JSON value. 

</div>

An extension value should be included as a JSON object containing the `@context` and `@type` properties and any new properties whose names are mapped in the context file referenced by `@context`. 

The property name for the extension should map to an IRI within the `@context` defined at the root of the extended Badge Object. It is possible to use a fully qualified IRI (e.g. `http://example.org/newBadgeExtension`) or a compact IRI within the extension namespace defined in the [OBI context](./1.1/context.json), like `extension:newBadgeExtension`. In either case, the IRI should correspond to where a human-readable definition of the extension resides. For extensions using the `extension` namespace, this definition may be contributed to the community [extensions repository](./extensions/) on this site.

See [example extensions](./extensions/).

## Validation <a id="validation"></a>
_since 1.1_
Open Badges v1.1 implements an optional JSON-schema based mechanism of ensuring badge objects conform to syntactic requirements of the specification. JSON-schema can ensure that required properties exist and that expected data types are used. From the [context](./v1/context.json)s for badge objects and extensions, a `validation` array may contain links to various JSON-schema against which badge objects may be tested. There are two proposed methods of specifying which component of a badge object should be matched against the JSON-schema validator: TypeValidation and FrameValidation. As of 1.1, only TypeValidation is implemented.

For example, this portion of the current Open Badges context links to a validator for Assertions. It indicates through TypeValidation that it should be run against JSON objects with the JSON-LD type of `Assertion` ([https://w3id.org/openbadges#Assertion]).
{% highlight json %}
{
  ...
  "validation": [
    {
      "type": "TypeValidation",
      "validatesType": "Assertion",
      "validationSchema": "https://openbadgespec.org/v1/schema/assertion.json"
    },
    ...
  ]
}
{% endhighlight %}

#### <a id="TypeValidation"></a>Type Validation 
_since 1.1_
Validators using the TypeValidation method match the schema indicated by the validator's `validationSchema` property against a JSON badge object document or portion of such a document that matches the validator's `validatesType` JSON-LD `type`.

{::options parse_block_html="true" /}
<div class="table-wrapper">

Property | Expected Type | Description/expected value
--------|------------|-----------
**type** | string/compact IRI | `TypeValidation`
**validatesType** | string/compact IRI | Valid JSON-LD type for a badge component, such as `Assertion`, `extensions:ApplyLink`, or `https://w3id.org/openbadges/extensions#ApplyLink`. Compact forms preferred.
**validationSchema** | URL | Location of a hosted JSON-schema

</div>

#### <a id="FrameValidation"></a> Frame Validation
_status: proposed_
Validators that someday use the proposed FrameValidation method pass JSON-LD objects through the JSON-LD Frame indicated by the `validationFrame` property and test the result against the JSON-schema indicated by the validator's `validationSchema` property.




## <a id="additional-properties"></a>Additional Properties

Badges consist of sets of claims, properties with values that apply to Issuer Organizations, all earners of a badge, or individual badge recipients. Outside of extensions, additional properties may be added to these claim sets so long as they are mapped to an IRI, as JSON-LD mapped in the context and do not clash with existing properties. For example, if a badge object creatorCreators of Badge Objects may:

1. Add individual mappings to the Badge Object's context: `"@context":["https://w3id.org/openbadges/v1", {"foo": "http://example.org/foo"}]`
2. Link to additional context files in the Badge Object's context: `"@context":["https://w3id.org/openbadges/v1", "http://example.org/context"]`
3. Add new properties using full IRIs as keys (or with compact IRIs in the existing context): `"http://example.org/foo":"bar"` or `"schema:comment":"baz"` where the IRI leads to the vocabulary definition for the term.

**Processors should preserve all properties when rehosting or retransmitting**.

If a property would be useful beyond internal use, an Extension is a recommended way to establish common practice for adding certain sets of information to badge objects.

### Primitives

* Boolean
* Text
* Array
* <a id="dateTime"></a>DateTime - Either an [ISO 8601](http://en.wikipedia.org/wiki/ISO_8601) date or a standard 10-digit Unix timestamp.
* <a id="type"></a>type - A string expressing the type of a Badge Object component. Currently applies to `identityType` and `verificationType`.
* URL - Fully qualified URL, including protocol, host, port if applicable, and path.
* IRI - In JSON-LD and Linked Data, IRIs (Internationalized Resource Identifiers) may look like fully qualified URLs or be namespaced within the JSON-LD context to be expanded to a full IRI.
* <a id="identityType"></a>IdentityType - Type of identity being represented. Currently the only supported value for many earner applications like the [Mozilla Backpack](http://backpack.openbadges.org) is "email".
* <a id="identityHash"></a>IdentityHash - A hash string preceded by a dollar sign ("$") and the algorithm used to generate the hash. For example: `sha256$28d50415252ab6c689a54413da15b083034b66e5` represents the result of calculating a SHA256 hash on the string "mayze". For more information, see [how to hash & salt in various languages](https://github.com/mozilla/openbadges/wiki/How-to-hash-&-salt-in-various-languages.).
* <a id="verificationType"></a>VerificationType - Type of verification. Can be either "hosted" or "signed".



# <a id="Implementation"></a> Implementation

## Hosted Badges <a id="HostedBadge"></a>

The badge assertion should live at a publicly accessible URL specified. Make sure that you are properly [setting the content-type](./examples/#setting-content-type) to `application/json`.

### Revoking
To mark a hosted assertion as revoked, respond with an HTTP Status of
`410 Gone` and a body of `{"revoked": true}`.

## Signed Badges ([example](./examples/#SignedBadge)) <a id="SignedBadge"></a>

A signed badge is in the form of a
[JSON Web Signature](http://self-issued.info/docs/draft-ietf-jose-json-web-signature.html):

{% highlight html %}
<encoded JWS header>.<encoded JWS payload>.<encoded JWS signature>
{% endhighlight %}

[See an example on the examples page.](./examples/#SignedBadge)

The JSON representation of the badge assertion should be used as the JWS
payload. For compatibility purposes, using an RSA-SHA256 is highly
recommended.

An example, with linebreaks for display purposes:

The public key corresponding to the private key used to the sign the
badge should be publicly accessible via HTTP and specified in the `verify.url`
property of the badge assertion.

### Revoking a Signed Badge

To mark a badge as revoked, add an entry to the resource pointed at by
the IssuerOrganization `revocationList` URL with the **uid** of the
badge and a reason why the badge is being revoked. See an [example](examples/#RevocationList).

## Badge Verification

An assertion will either be raw JSON (hosted assertion) or a JWS object
(signed assertion)

It is STRONGLY RECOMMENDED that a display implementation
show the `verify.url`, with the origin (protocol, hostname, port if
non-default) highlighted.

The use of the term "eventual 200 OK" is meant to mean that 3xx
redirects are allowed, as long as the request eventually terminates on a
resource that returns a 200 OK.

### Signed Assertion

1. Unpack the JWS payload. This will be a JSON string representation of
the badge assertion.

2. Parse the JSON string into a JSON object. If the parsing operation
fails, assertion MUST be treated as invalid.

3. Assert structural validity.

4. Extract the `verify.url` property from the JSON object. If their is no
`verify.url` property, or the `verify.url` property does not contain a valid
URL, assertion MUST be treated as invalid.

5. Perform an HTTP GET request on `verify.url` and store public key. If the
HTTP status is not 200 OK (either directly or through 3xx redirects),
the assertion MUST be treated as invalid.

6. With the public key, perform a JWS verification on the JWS object. If
the verification fails, assertion MUST be treated as invalid.

7. Retrieve the revocation list from the IssuerOrganization object and
ensure the `uid` of the badge does not appear in the list.

8. If the above steps pass, assertion MAY BE treated as valid.

### Hosted Assertion

1. Perform an HTTP GET request on the `verify.url`. If the HTTP Status
is not eventually 200 OK, assertion MUST BE treated as invalid.

2. Assert structural validity


## Other Resources <a id="OtherResources"></a>

## Assertion Validator <a id="Validator"></a>
[http://validator.openbadges.org/](http://validator.openbadges.org/)

## History <a id="History"></a>
 * [From 1.0 to 1.1](history/1.1.html) 
 * [From 0.5 to 1.0](https://github.com/mozilla/openbadges/wiki/Assertion-Specification-Changes)
 * [Early history of the specification](https://github.com/mozilla/openbadges-backpack/wiki/Assertions/_history)
