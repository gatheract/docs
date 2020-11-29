[![GATHERACT](docs/assets/logo.svg)](https://gatheract.com)
[![LTIJS](docs/assets/logo-lti.svg)](https://cvmcosta.github.io/ltijs/#/provider)

> GatherAct LTI Integration API

[![Ltijs provider](https://img.shields.io/badge/ltijs-provider-brightgreen.svg)](https://cvmcosta.github.io/ltijs/#/provider)

---

## Introduction

The Learning Tools Interoperability (LTI®) protocol is a standard for integration of rich learning applications within educational environments. <sup>[ref](https://www.imsglobal.org/spec/lti/v1p3/)</sup>

Implementing the LTI® 1.3 protocol and all of it's services can be a very difficult, time consuming task. Through **GatherAct** any project can be easily turned into a fully LTI® 1.3 compliant learning tool, with minimal alterations to the project's codebase.

**GatherAct** gives the application access to all of the LTI® services and functionalities through a very simple to use API.

The **GatherAct** LTI project was built using the **IMS LTI® Advantage Complete Certified library** [Ltijs](https://site.imsglobal.org/certifications/coursekey/ltijs). For more information visit the [Ltijs documentation](https://cvmcosta.me/ltijs/#/provider).

---

## Table of Contents

- [Terminology](#terminology)
- [The LTI 1.3 Protocol](#the-lti-13-protocol)
  - [Core](#core)
  - [Deep Linking](#deep-linking)
  - [Names and Roles Provisioning](#names-and-roles-provisioning)
  - [Assignment and Grade](#assignment-and-grade)
- [API Documentation](#api-documentation)
  - [Endpoints](#endpoints)
  - [Authentication](#authentication)
  - [IdToken endpoint](#idtoken-endpoint)
    - [Retrieve IdToken](#retrieve-idtoken)
  - [Memberships endpoint](#memberships-endpoint)
    - [Retrieve memberships](#retrieve-memberships)
  - [LineItems endpoint](#lineitems-endpoint)
    - [Retrieve line items](#retrieve-line-items)
    - [Create line item](#create-line-item)
    - [Retrieve line item by ID](#retrieve-line-item-by-id)
    - [Update line item by ID](#update-line-item-by-id)
    - [Delete line item by ID](#delete-line-item-by-id)
    - [Submit scores](#submit-scores)
    - [Retrieve scores](#retrieve-scores)
  - [Errors](#errors)

---

## Terminology

- **Tool** - LTI® 1.3 compliant Learning Resources.
- **Platform** - Learning Management System (LMS) capable of consuming LTI® 1.3 compliant Learning Resources.
- **Tool Link** - Instance of a **Tool** inside of a **Platform**. A Tool Link can take many forms, but the most common one is the Activity.
- **Launch context** - Information about the entire context surrounding the launch, usually data identifying the User, Platform, Course and Activity.
- **Platform context** - Platform context references the wider scope in which a launch took place, usually a Course.
- **Membership** - User information. Always contains the user's id and roles inside the current Platform context, but can be extended to contain information like the user's name and email. More information about User memberships can be found in the [IMS Membership specification](https://www.imsglobal.org/spec/lti-nrps/v2p0#membership-container-media-type).
- **Line Item** - Grade line contained in the current Platform context (usually the current Course).

---

## The LTI® 1.3 Protocol

The learning tools interoperability (LTI®) protocol describes how any learning resource can communicate with a Learning management system (LMS) and exchange relevant information, seamlessly integrating them together to improve the educational process.

Possible interactions between Platforms and the Tools are described as Services. The LTI® 1.3 protocol officially describes 4 Services:

- **Core (Launch)**
- **Deep Linking**
- **Names and Roles Provisioning (Memberships)**
- **Assignment and Grade**

### Core

The [Core Service](https://www.imsglobal.org/spec/lti/v1p3/) (Launch) is the most basic interaction between a Platform and Tool. It describes the process through which a Tool can be launched from within a Platform, usually in the form of an embedded iframe, while receiving information about the launch context.

After a successful launch, a Tool should have access to an IdToken, a signed JWT containing the launch context and a variety of additional information.

Some of the relevant information that can be found in an IdToken:

- **User Information**
  - User ID
  - Email
  - Name
  - Role inside current Platform context (e.g. Instructor, Learner).
- **Platform information**
  - URL
  - Name
  - Version
- **Tool Information**
  - ClientID
- **Context Information**
  - Course ID
  - Activity ID
- **Custom Parameters**

A **Custom Parameter** is a key-value pair set at the moment of the creation of a Tool Link (e.g. Activity) inside a Platform. Tools offering multiple resources often use either **Custom Parameters** or **Query Parameters** to decide which resource to display after a successful launch.

More information about the LTI Launch and the IdToken can be found in the [Official IMS LTI® Specification](https://www.imsglobal.org/spec/lti/v1p3/).

### Deep Linking

The [Deep Linking Service](https://www.imsglobal.org/spec/lti-dl/v2p0) can be used to create Tool Links inside a Platform.

The Deep Linking process consists of a Launch to the Tool’s deep linking endpoint where the user can select which resource(s) they want to link. The tool will then generate a signed JWT containing the selected resources and submit this message through a self-submitting form to the Platform.

Deep Linking resources are called “Content Items”, Tool Links are only one of the available types of Content Item:

- LTI® Resource Link (Tool Link).
- Link
- File
- HTML Fragment
- Image

**GatherAct** uses Deep Linking to allow users to Launch directly to you application inside a Platform.

### Names and Roles Provisioning

The [Names and Roles Provisioning Service](https://www.imsglobal.org/spec/lti-nrps/v2p0) allows a Tool to retrieve a list of User **memberships** inside the current Platform context (usually the current Course). It is possible to filter the results with the following parameters:

- role - The User’s role within the context, e.g. Instructor, Learner.
- limit - The maximum number of results to be returned.
- rlid - Access [Resource Link level memberships](https://www.imsglobal.org/spec/lti-nrps/v2p0#resource-link-membership-service) by passing the id for the current context's resource link.

Platforms can choose what User information they want to share with Tools, only two parameters are required to be present:

- user_id - The User’s ID inside the Platform.
- roles - The User’s roles within the current Platform context.

### Assignment and Grade

The [Assignment and Grade Service](https://www.imsglobal.org/spec/lti-ags/v2p0) allows a Tool to retrieve and create scores and line items inside a Platform. These functionalities are separated into three subservices:

- **Line Item Service** - Allows Tools to retrieve, create or delete line items in the current context’s grade book.

- **Score Publish Service** - Allows Tools to publish scores to a specific line item in the current context’s grade book.

- **Result Service** - Allows Tools to retrieve scores from lines items in the current context’s grade book.

---

## API Documentation

Through the **GatherAct** LTI API it's possible to access all of the LTI® 1.3 services and functionalities.

### Endpoints

- **/idtoken [GET]** - Retrieve IdToken for the current launch context.
- **/memberships [GET]** - Retrieve a list of user memberships inside the current Platform context.
- **/lineitems [GET, POST]** - Retrieve and create line items.
  - **/:lineitemId [GET, PUT, DELETE]** - Retrieve, update and delete a line item.
    - **/scores [GET, POST]** - Retrieve or Publish scores to a line item.

### Authentication

After a successful LTI® Launch is completed, **GatherAct** will redirect the user to your application. **GatherAct** will also append a `ltik` query parameter, a signed JWT identifying the current launch context.

_GatherAct launch flow:_

![Launch flow](docs/assets/launch.png)

This token **MUST be returned whenever the Tool wants to access any of the** [LTI API endpoints](#endpoints).

_GatherAct LTI API access flow:_

![API access flow](docs/assets/access.png)

The `ltik` token can be passed to the API as a query parameter, body parameter or in the Authorization header (Bearer or LTIK-AUTH-V1).

#### Ltik in the query parameters

The `ltik` token can be passed through the URL query parameters:

> https://gatheract.com/lti/idtoken?ltik=<ltik\>

Example:

> https://gatheract.com/lti/idtoken?ltik=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c

#### Ltik in the request body

The `ltik` token can be passed through the request body:

```javascript
{
  ltik: <ltik>
}
```

Example:

```javascript
{
	ltik: 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c';
}
```

#### Bearer Authorization header

The `ltik` parameter can be passed through a Bearer Authorization header:

> Authorization: Bearer \<ltik\>

Example:

> Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c

#### LTIK-AUTH-V1 Authorization header

If there is need for an additional Authorization header, **GatherAct** implements it's own authorization schema `LTIK-AUTH-V1` with support for multiple authorization methods:

> Authorization: LTIK-AUTH-V1 Token=\<ltik\>, Additional=\<additional\>

- **Token** - Ltik authentication token.
- **Additional** - Additional authorization schema.

Example:

> Authorization: LTIK-AUTH-V1 Token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c, Additional=Bearer KxwRJSMeKKF2QT4fwpM

When using the `LTIK-AUTH-V1` authorization schema, `req.headers.authorization` will only include the `Additional` portion of the header. The `ltik` token can be found in `req.token`.

#### Order of priority

**GatherAct** will look for the `ltik` in the following order:

- LTIK-AUTH-V1 Authorization
- query
- body
- Bearer Authorization

### IdToken Endpoint

The **/idtoken** endpoint is used to retrieve an IdToken object containing the current launch context information.

#### Retrieve IdToken

| ROUTE      | METHOD | LTIK REQUIRED |
| ---------- | ------ | ------------- |
| `/idtoken` | `GET`  | ✔️            |

##### Parameters

The **/idtoken** endpoint receives no parameters.

##### Response

Returns an `IdToken` object containing the entire launch context information.

The contents of the object may vary, since Platforms might return different information, but the information is divided into easy to understand categories:

- **user** - User Information.
  - **id** - User ID.
  - **email** - User email.
  - **given_name** - User given name.
  - **family_name** - User family name.
  - **name** - User name.
  - **roles** - User roles in the current context.
- **platform** - Platform information.
  - **id** - Platform ID.
  - **url** - Platform URL.
  - **clientId** - Platform generated Client ID.
  - **deploymentId** - Platform generated Deployment ID.
  - **product_family_code** - Platform family code.
  - **version** - Platform version.
  - **name** - Platform name.
  - **description** - Platform description.
- **launch** - Launch context information.
  - **type** - Launch type.
  - **target** - Launch target URL.
  - **context** - Course or section information
    - id
    - label
    - title
    - type
  - **resource** - Activity or placement information.
    - id
    - title
  - **presentation** - Information regarding how the Tool is being displayed in the Platform.
    - locale
    - document_target
  - **custom** - Custom parameters object.
  - **lineItemId** - Line item assigned to this resource. This field is only present if there is exactly one line item assigned to this resource. Can be used to quickly access the [LineItems endpoint](#lineitems-endpoint) to [submit](#submit-scores) or [retrieve scores](#retrieve-scores).

Example response:

```javascript
{
  user: {
    id: '2',
    given_name: 'Admin',
    family_name: 'User',
    name: 'Admin User',
    email: 'admin@lms.example.com',
    roles: [
      'http://purl.imsglobal.org/vocab/lis/v2/institution/person#Administrator',
      'http://purl.imsglobal.org/vocab/lis/v2/membership#Instructor',
      'http://purl.imsglobal.org/vocab/lis/v2/system/person#Administrator'
    ]
  },
  platform: {
    id: '0c41aa0849215449d4298e58f7626c68',
    url: 'https://lms.example.com',
    clientId: 'CLIENTID',
    deploymentId: '1',
    product_family_code: 'canvas',
    version: '2020073000',
    guid: 'lms.example.com',
    name: 'LMS',
    description: 'LMS',
    lis: {
      person_sourcedid: '',
      course_section_sourcedid: ''
    },
    deepLinkingSettings: null
  },
  launch: {
    type: 'LtiResourceLinkRequest',
    target: 'http://tool.example.com?resource=value1',
    context: {
      id: '2',
      label: 'course',
      title: 'Course',
      type: [
          'CourseSection'
        ]
    },
    resource: {
      title: 'Activity',
      id: '1'
    },
    presentation: {
      locale: 'en',
      document_target: 'iframe',
      return_url: 'https://lms.example.com/mod/lti/return.php?course=2&launch_container=3&instanceid=1&sesskey=huxFIyJ0BP'
    },
    custom: {
      group: '2',
      system_setting_url: 'https://lms.example.com/mod/lti/services.php/tool/1/custom',
      context_setting_url: 'https://lms.example.com/mod/lti/services.php/CourseSection/2/bindings/tool/1/custom',
      link_setting_url: 'https://lms.example.com/mod/lti/services.php/links/{link_id}/custom'
    },
    lineItemId: 'https://lms.example.com/mod/lti/services.php/2/lineitems/2/lineitem?type_id=1'
  }
}
```

Since the idtoken receives varied information depending on the LMS, fields don't follow a strict naming convention. Fields not using camel case are usually the ones coming directly from the LMS without any modification.

##### Example usage

```javascript
const searchParams = new URLSearchParams(window.location.search);
const ltik = searchParams.get('ltik');
const idtoken = await request.get(API_URL + '/idtoken', { headers: { Authorization: 'Bearer ' + ltik } }).json();
return idtoken;
```

### Memberships Endpoint

The **/memberships** endpoint represents the [Names and Roles Provisioning Service](#names-and-roles-provisioning) and is used to retrieve a list of User **memberships**.

#### Retrieve memberships

| ROUTE          | METHOD | LTIK REQUIRED |
| -------------- | ------ | ------------- |
| `/memberships` | `GET`  | ✔️            |

##### Parameters

| PARAMETER        | TYPE      | LOCATION | REQUIRED |
| ---------------- | --------- | -------- | -------- |
| `role`           | `String`  | `query`  |          |
| `limit`          | `Number`  | `query`  |          |
| `url`            | `String`  | `query`  |          |
| `resourceLinkId` | `Boolean` | `query`  |          |

**role:** Filters memberships by role. Possible role values are defined in the [IMS Role Vocabularies](https://www.imsglobal.org/spec/lti/v1p3/#role-vocabularies).

**limit:** Limits the number of memberships returned.

**url:** Retrieves memberships from a specific URL.

In cases where not all members are retrieved when the membership limit is reached, the returned object will contain a `next` field holding an URL that can be used to retrieve the remaining members.

_When present, the `url` parameter causes every other parameter to be ignored._

The `url` parameter should be URL encoded.

**resourceLinkId:** Retrieves the Resource Link level memberships as specified in the [IMS Resource Link Membership Service specification](https://www.imsglobal.org/spec/lti-nrps/v2p0#resource-link-membership-service).

##### Response

Returns an object with a **members** field containing an array of User memberships. **Only the `user_id` and `roles` fields of each membership are required to be returned by the Platform**.

More information about User memberships can be found in the [IMS Membership specification](https://www.imsglobal.org/spec/lti-nrps/v2p0#membership-container-media-type).

Other fields present in the response can be:

- **id** - URL the memberships were retrieved from.
- **context** - Context the memberships were retrieved from.
- **next** - URL of the next membership page. **This field is only present if the membership limit was reached before the full members list was retrieved.**

Example response:

```javascript
{
  id : 'https://lms.example.com/sections/2923/memberships',
  context: {
    id: '2923-abc',
    label: 'CPS 435',
    title: 'CPS 435 Learning Analytics',
  },
  next: 'https://lms.example.com/sections/2923/memberships/pages/2',
  members : [
    {
      status : 'Active',
      name: 'Jane Q. Public',
      picture : 'https://platform.example.edu/jane.jpg',
      given_name : 'Jane',
      family_name : 'Doe',
      middle_name : 'Marie',
      email: 'jane@platform.example.edu',
      user_id : '0ae836b9-7fc9-4060-006f-27b2066ac545',
      lis_person_sourcedid: '59254-6782-12ab',
      roles: [
        'http://purl.imsglobal.org/vocab/lis/v2/membership#Instructor'
      ]
    }
  ]
}

```

##### Example usage

```javascript
const searchParams = new URLSearchParams(window.location.search);
const ltik = searchParams.get('ltik');
const query = {
	role: 'Learner',
	limit: 10
};
const response = await request
	.get(API_URL + '/memberships', { searchParams: query, headers: { Authorization: 'Bearer ' + ltik } })
	.json();
return response.members;
```

### LineItems Endpoint

The **/lineitems** endpoints represent the [Assignment and Grade Service](#assignment-and-grade) and are used to access and manipulate line items and scores.

#### Retrieve line items

| ROUTE        | METHOD | LTIK REQUIRED |
| ------------ | ------ | ------------- |
| `/lineitems` | `GET`  | ✔️            |

##### Parameters

| PARAMETER        | TYPE      | LOCATION | REQUIRED |
| ---------------- | --------- | -------- | -------- |
| `id`             | `String`  | `query`  |          |
| `resourceId`     | `String`  | `query`  |          |
| `tag`            | `String`  | `query`  |          |
| `label`          | `String`  | `query`  |          |
| `resourceLinkId` | `Boolean` | `query`  |          |
| `limit`          | `Number`  | `query`  |          |
| `url`            | `String`  | `query`  |          |

**id:** Retrieves a specific line item by ID. Line item IDs are URLs so the parameter should be URL encoded.

**resourceId:** Filters line items by resourceId.

**tag:** Filters line items by tag.

**label:** Filters line items by label.

**resourceLinkId:** Filters line items by the resource link ID attached to the current launch context.

**limit:** Limits the number of line items returned.

**url:** Retrieves line items from a specific URL.

In cases where not all line items are retrieved when the line item limit is reached, the returned object will contain a `next` field holding an URL that can be used to retrieve the remaining line items.

_When present, the `url` parameter causes every other parameter to be ignored._

The `url` parameter should be URL encoded.

##### Response

Returns an object with a **lineItems** field containing an array of line items.

More information about line items can be found in the [IMS Line Item Service specification](https://www.imsglobal.org/spec/lti-ags/v2p0/#line-item-service).

Other fields present in the response can be:

- **next** - URL of the next line item page. **This field is only present if the line item limit was reached before the full list was retrieved.**
- **prev** - URL of the previous line item page. **This field is only present if the line item limit was reached before the full list was retrieved.**
- **first** - URL of the first line item page. **This field is only present if the line item limit was reached before the full list was retrieved.**
- **last** - URL of the last line item page. **This field is only present if the line item limit was reached before the full list was retrieved.**

Example response:

```javascript
{
  next: 'https://lms.example.com/sections/2923/lineitems/pages/2',
  first: 'https://lms.example.com/sections/2923/lineitems/pages/1',
  last: 'https://lms.example.com/sections/2923/lineitems/pages/3',
  lineItems: [
    {
      id: 'https://lms.example.com/context/2923/lineitems/1',
      scoreMaximum: 60,
      label: 'Chapter 5 Test',
      resourceId: 'a-9334df-33',
      tag: 'grade',
      resourceLinkId: '1g3k4dlk49fk',
      endDateTime: '2018-04-06T22:05:03Z'
    },
    {
      id: 'https://lms.example.com/context/2923/lineitems/47',
      scoreMaximum: 100,
      label: 'Chapter 5 Progress',
      resourceId: 'a-9334df-33',
      tag: 'originality',
      resourceLinkId: '1g3k4dlk49fk'
    },
    {
      id: 'https://lms.example.com/context/2923/lineitems/69',
      scoreMaximum: 60,
      label: 'Chapter 2 Essay',
      tag: 'grade'
    }
  ]
}
```

##### Example usage

```javascript
const searchParams = new URLSearchParams(window.location.search);
const ltik = searchParams.get('ltik');
const query = {
	tag: 'grade_line',
	label: 'Grade line'
};
const response = await request
	.get(API_URL + '/lineitems', { searchParams: query, headers: { Authorization: 'Bearer ' + ltik } })
	.json();
return response.lineItems;
```

#### Create line item

| ROUTE        | METHOD | LTIK REQUIRED |
| ------------ | ------ | ------------- |
| `/lineitems` | `POST` | ✔️            |

##### Parameters

Requests **MUST** have a body representing a Line Item Object as specified in the [IMS Line Item Object Specification](https://www.imsglobal.org/spec/lti-ags/v2p0/#creating-a-new-line-item).

The only required parameters are **label** and **scoreMaximum**.

| PARAMETER      | TYPE     | LOCATION | REQUIRED |
| -------------- | -------- | -------- | -------- |
| `label`        | `String` | `body`   | ✔️       |
| `scoreMaximum` | `Number` | `body`   | ✔️       |

**label:** Line item label.

**scoreMaximum:** Maximum score allowed for the line item.

##### Response

Returns the newly created line item.

Example response:

```javascript
{
  id: 'https://lms.example.com/context/2923/lineitems/1',
  scoreMaximum: 60,
  label: 'Chapter 5 Test',
  resourceId: 'quiz-231',
  tag: 'grade',
  startDateTime: '2018-03-06T20:05:02Z',
  endDateTime: '2018-04-06T22:05:03Z'
}
```

##### Example usage

```javascript
const searchParams = new URLSearchParams(window.location.search);
const ltik = searchParams.get('ltik');
const lineitem = {
	scoreMaximum: 60,
	label: 'Grade line',
	resourceId: 'quiz-231',
	tag: 'grade',
	startDateTime: '2021-03-06T20:05:02Z',
	endDateTime: '2021-04-06T22:05:03Z'
};
const newLineitem = await request
	.post(API_URL + '/lineitems', { json: lineitem, headers: { Authorization: 'Bearer ' + ltik } })
	.json();
return newLineitem;
```

#### Retrieve line item by ID

| ROUTE                    | METHOD | LTIK REQUIRED |
| ------------------------ | ------ | ------------- |
| `/lineitems/:lineitemid` | `GET`  | ✔️            |

The line item ID is an URL so it **MUST be URL encoded**.

##### Parameters

The **/lineitems/:lineitemid** GET endpoint receives no parameters.

##### Response

Returns the requested line item.

Example response:

```javascript
{
  id: 'https://lms.example.com/context/2923/lineitems/1',
  scoreMaximum: 60,
  label: 'Chapter 5 Test',
  resourceId: 'quiz-231',
  tag: 'grade',
  startDateTime: '2018-03-06T20:05:02Z',
  endDateTime: '2018-04-06T22:05:03Z'
}
```

##### Example usage

```javascript
const searchParams = new URLSearchParams(window.location.search);
const ltik = searchParams.get('ltik');
const lineitemId = 'https://lms.example.com/context/2923/lineitems/1';
const lineitem = await request
	.get(API_URL + '/lineitems/' + encodeURIComponent(lineitemId), { headers: { Authorization: 'Bearer ' + ltik } })
	.json();
return lineitem;
```

#### Update line item by ID

| ROUTE                    | METHOD | LTIK REQUIRED |
| ------------------------ | ------ | ------------- |
| `/lineitems/:lineitemid` | `PUT`  | ✔️            |

The line item ID is an URL so it **MUST be URL encoded**.

##### Parameters

Requests **MUST** have a body representing a Line Item Object as specified in the [IMS Line Item Object Specification](https://www.imsglobal.org/spec/lti-ags/v2p0/#creating-a-new-line-item).

The only required parameters are **label** and **scoreMaximum**.

| PARAMETER      | TYPE     | LOCATION | REQUIRED |
| -------------- | -------- | -------- | -------- |
| `label`        | `String` | `body`   | ✔️       |
| `scoreMaximum` | `Number` | `body`   | ✔️       |

**label:** Line item label.

**scoreMaximum:** Maximum score allowed for the line item.

##### Response

Returns the updated line item.

Example response:

```javascript
{
  id: 'https://lms.example.com/context/2923/lineitems/1',
  scoreMaximum: 60,
  label: 'Chapter 5 Updated',
  resourceId: 'quiz-231',
  tag: 'grade',
  startDateTime: '2018-03-06T20:05:02Z',
  endDateTime: '2018-04-06T22:05:03Z'
}
```

##### Example usage

```javascript
const searchParams = new URLSearchParams(window.location.search);
const ltik = searchParams.get('ltik');
const lineitemId = 'https://lms.example.com/context/2923/lineitems/1';
const lineitem = {
	scoreMaximum: 60,
	label: 'Chapter 5 Updated'
};
const updatedLineitem = await request
	.put(API_URL + '/lineitems/' + encodeURIComponent(lineitemId), {
		json: lineitem,
		headers: { Authorization: 'Bearer ' + ltik }
	})
	.json();
return updatedLineitem;
```

#### Delete line item by ID

| ROUTE                    | METHOD   | LTIK REQUIRED |
| ------------------------ | -------- | ------------- |
| `/lineitems/:lineitemid` | `DELETE` | ✔️            |

The line item ID is an URL so it **MUST be URL encoded**.

##### Parameters

The **/lineitems/:lineitemid** DELETE endpoint receives no parameters.

##### Response

Returns `204` if the line item is successfully deleted.

##### Example usage

```javascript
const searchParams = new URLSearchParams(window.location.search);
const ltik = searchParams.get('ltik');
const lineitemId = 'https://lms.example.com/context/2923/lineitems/1';
await request.delete(API_URL + '/lineitems/' + encodeURIComponent(lineitemId), {
	headers: { Authorization: 'Bearer ' + ltik }
});
return true;
```

#### Submit scores

| ROUTE                           | METHOD | LTIK REQUIRED |
| ------------------------------- | ------ | ------------- |
| `/lineitems/:lineitemid/scores` | `POST` | ✔️            |

The line item ID is an URL so it **MUST be URL encoded**.

##### Parameters

Requests MUST have a body representing a Score Object as specified in the [IMS Score Specification](https://www.imsglobal.org/spec/lti-ags/v2p0/#example-application-vnd-ims-lis-v1-score-json-representation).

The required parameters are **userId**, **activityProgress** and **gradingProgress**. The parameters **timestamp** and **scoreMaximum** are also required but are set automatically by **GatherAct**.

| PARAMETER          | TYPE     | LOCATION | REQUIRED |
| ------------------ | -------- | -------- | -------- |
| `userId`           | `String` | `body`   | ✔️       |
| `activityProgress` | `String` | `body`   | ✔️       |
| `gradingProgress`  | `String` | `body`   | ✔️       |

**userId:** Target user ID.

**activityProgress:** Status of the user towards the activity's completion.

**gradingProgress:** Status of the grading process.

##### Response

Returns the submitted score.

Example response:

```javascript
{
  userId: '5323497',
  activityProgress: 'Completed',
  gradingProgress: 'FullyGraded',
  scoreGiven: 83,
  scoreMaximum: 100,
  comment: 'This is exceptional work.',
  timestamp: '2021-04-16T18:54:36.736+00:00',
}
```

##### Example usage

```javascript
const searchParams = new URLSearchParams(window.location.search);
const ltik = searchParams.get('ltik');
const lineitemId = 'https://lms.example.com/context/2923/lineitems/1';
const score = {
	userId: '5323497',
	activityProgress: 'Completed',
	gradingProgress: 'FullyGraded',
	scoreGiven: 83,
	comment: 'This is exceptional work.'
};
const submittedScore = await request
	.post(API_URL + '/lineitems/' + encodeURIComponent(lineitemId) + '/scores', {
		json: score,
		headers: { Authorization: 'Bearer ' + ltik }
	})
	.json();
return submittedScore;
```

#### Retrieve scores

| ROUTE                           | METHOD | LTIK REQUIRED |
| ------------------------------- | ------ | ------------- |
| `/lineitems/:lineitemid/scores` | `GET`  | ✔️            |

The line item ID is an URL so it **MUST be URL encoded**.

##### Parameters

| PARAMETER | TYPE     | LOCATION | REQUIRED |
| --------- | -------- | -------- | -------- |
| `userId`  | `String` | `query`  |          |
| `limit`   | `Number` | `query`  |          |
| `url`     | `String` | `query`  |          |

**userId:** Filters scores based on the user ID.

**limit:** Limits the number of scores returned.

**url:** Retrieves scores from a specific URL.

In cases where not all scores are retrieved when the score limit is reached, the returned object will contain a `next` field holding an URL that can be used to retrieve the remaining scores.

_When present, the `url` parameter causes every other parameter to be ignored._

The `url` parameter should be URL encoded.

##### Response

Returns an object with a **scores** field containing an array of scores.

More information about scores can be found in the [IMS Result Service specification](https://www.imsglobal.org/spec/lti-ags/v2p0/#result-service).

Other fields present in the response can be:

- **next** - URL of the next score page. **This field is only present if the score limit was reached before the full list was retrieved.**
- **prev** - URL of the previous score page. **This field is only present if the score limit was reached before the full list was retrieved.**
- **first** - URL of the first score page. **This field is only present if the score limit was reached before the full list was retrieved.**
- **last** - URL of the last score page. **This field is only present if the score limit was reached before the full list was retrieved.**

Example response:

```javascript
{
  next: 'https://lms.example.com/sections/2923/scores/pages/2',
  first: 'https://lms.example.com/sections/2923/scores/pages/1',
  last: 'https://lms.example.com/sections/2923/scores/pages/3',
  scores: [
    {
      id: 'http://localhost/moodle/mod/lti/services.php/2/lineitems/16/lineitem/results?type_id=1&user_id=2',
      userId: '2',
      resultScore: 100,
      resultMaximum: 100,
      scoreOf: 'http://localhost/moodle/mod/lti/services.php/2/lineitems/16/lineitem?type_id=1',
      timestamp: '2020-06-02T10:51:08-03:00'
    },
    {
      id: 'http://localhost/moodle/mod/lti/services.php/2/lineitems/16/lineitem/results?type_id=1&user_id=2',
      userId: '3',
      resultScore: 90,
      resultMaximum: 100,
      scoreOf: 'http://localhost/moodle/mod/lti/services.php/2/lineitems/16/lineitem?type_id=1',
      timestamp: '2020-06-02T10:51:08-03:00'
    }
  ]
}
```

##### Example usage

```javascript
const searchParams = new URLSearchParams(window.location.search);
const ltik = searchParams.get('ltik');
const lineitemId = 'https://lms.example.com/context/2923/lineitems/1';
const query = {
	userId: 2,
	limit: 10
};
const response = await request
	.get(API_URL + '/lineitems/' + encodeURIComponent(lineitemId) + '/scores', {
		searchParams: query,
		headers: { Authorization: 'Bearer ' + ltik }
	})
	.json();
return response.scores;
```

### Errors

Error responses generated by any of the **GatherAct** endpoints obey the following pattern:

```javascript
{
  status: ERROR_STATUS,
  error: ERROR_MESSAGE,
  details: ERROR_DESCRIPTIONS
}
```

Example:

```javascript
{
  status: 401,
  error: "Unauthorized",
  details: {
    message: "Missing parameter \"token\"."
    bodyReceived: {
      param: "value"
    }
  }
}
```

---

> Copyright © GatherAct - 2020
