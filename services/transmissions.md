title: Transmissions
description: Use the transmissions API to send a batch of messages through SparkPost.

# Group Transmissions
<a name="transmissions-api"></a>

A transmission is a collection of messages belonging to the same campaign. It is also known as a mailing. The Transmissions API provides the means to create and manage transmissions - to send messages.

SparkPost generates and sends the messages from your transmissions using a list of recipients and a message template. You can [store a recipient list](recipient-lists.html), or you can include recipients "inline" with your transmission request. Similarly, you can [store your message template](templates.html) or include it "inline" with your transmission request. SparkPost then sends a unique message to each recipient using the specified template. You may also choose to personalise your messages by including substitution variables in your transmissions. You can [learn about templates and substitution here](substitutions-reference.html).  Finally, you can enable engagement tracking in your transmissions to track message opens and clicks.

For details on how to get the best out of SparkPost Transmission, see [this support article](https://www.sparkpost.com/docs/tech-resources/smtp-rest-api-performance/).

## Using Postman

If you use [Postman](https://www.getpostman.com/) you can click the following button to import the SparkPost API as a collection:

[![Run in Postman](https://s3.amazonaws.com/postman-static/run-button.png)](https://app.getpostman.com/run-collection/5d9ae743a661a15d64bb)

## The Sandbox Domain

The sandbox domain `sparkpostbox.com` is available to allow each account to send test messages in advance of configuring a real sending domain. Each SparkPost account has a lifetime allowance of 5 sandbox domain messages. To send a test message from the sandbox domain, set `content.from.email` field to `localpart@sparkpostbox.com` and ensure `options.sandbox` is set to `true`.

<div class="alert alert-info"><strong>Note</strong>: you can set the 'local part' (the part before the <tt>@</tt>) to any valid email local part. <a href="#transmissions-create-post"><strong>See below</strong></a> for more details on sending mail.</div>

<div class="alert alert-info"><strong>Note</strong>: SparkPost accounts only. <strong><a href="https://www.sparkpost.com/enterprise-email/">SparkPost Enterprise</a></strong> accounts should consider using the <a href="https://support.sparkpost.com/customer/portal/articles/2560839">SparkPost Sink Server</a>.</div>

## Transmission Attributes

| Field         | Type     | Description                           | Required         | Notes   |
|--------------------|----------------      |---------------------------------------|--------------------------|--------|
|id |string |ID of the transmission |no |Read only.  A unique ID is generated for each transmission on submission. |
|state |string  |State of the transmission  | no | Read only.  Valid responses are `submitted`, `Generating`, `Success`, or `Canceled`. |
|options | JSON object | JSON object in which transmission options are defined | no | For a full description, see [Options Attributes](#header-options-attributes).
|recipients | JSON array or JSON object | Inline recipient objects or object containing stored recipient list ID |yes | Specify a stored recipient list or specify recipients inline.  When using a stored recipient list, specify the `list_id` as described in Using a Stored Recipient List.  Otherwise, provide the recipients inline using the fields described in the Recipient List API documentation for Recipient Attributes. |
|campaign_id | string |Name of the campaign|no|Maximum length - 64 bytes|
|description | string |Description of the transmission|no | Maximum length - 1024 bytes|
|metadata|JSON object|Transmission level metadata containing key/value pairs |no| Metadata is available during events through the Webhooks and is provided to the substitution engine.  A maximum of 1000 bytes of merged metadata (transmission level + recipient level) is available with recipient metadata taking precedence over transmission metadata when there are conflicts.  |
|substitution_data|JSON object|Key/value pairs that are provided to the substitution engine| no | Recipient substitution data takes precedence over transmission substitution data. Unlike metadata, substitution data is not included in Webhook events. |
|return_path | string | Email address to use for envelope FROM | no | For <span class="label label-primary"><strong>SparkPost</strong></span> accounts, the domain part of the return_path address must be a [CNAME-verified sending domain](sending-domains.html#sending-domains-verify-post).  The local part of the return_path address will be overwritten by SparkPost servers.<br><br>For <a href="https://www.sparkpost.com/enterprise-email/"><span class="label label-warning"><strong>Enterprise</strong></span></a> accounts, the return_path may be any valid email address and the localpart in the return_path will **not** be overwritten by SparkPost servers.  To support Variable Envelope Return Path (VERP), this field can also optionally be specified inside each recipient object in order to give the recipients unique envelope MAIL FROM addresses. |
|content| JSON object | Content that will be used to construct a message | yes | Specify a stored template or specify inline template content. When using a stored template, specify the `template_id` as described in Using a Stored Template.  Otherwise, provide the inline content using the fields described in Inline Content Attributes.  Maximum size - 20MBs|
|total_recipients | number | Computed total recipients | no | Read only|
|num_generated | number | Computed total number of messages generated | no |Read only|
|num_failed_generation| number| Computed total number of failed messages | no | Read only|
|num_invalid_recipients | number | Number of recipients that failed input validation |no |Read only|


### Options Attributes
| Field         | Type     | Description                           | Required   | Notes   |
|------------------------|:-:       |---------------------------------------|-------------|--------|
|start_time | string | Delay generation of messages until this datetime.  For additional information, see [Scheduled Transmissions](#header-scheduled-transmissions).  **Not available in SparkPost EU.** |no - defaults to immediate generation | Format YYYY-MM-DDTHH:MM:SS+-HH:MM. Example: `2017-02-11T08:00:00-04:00`.|
|open_tracking|boolean| Whether open tracking is enabled for this transmission| no |If not specified, the setting at template level is used, or defaults to true. |
|click_tracking|boolean| Whether click tracking is enabled for this transmission| no |If not specified, the setting at template level is used, or defaults to true. |
|transactional|boolean|Whether message is [transactional](https://www.sparkpost.com/resources/infographics/email-difference-transactional-vs-commercial-emails/) for unsubscribe and suppression purposes<br/><span class="label label-info"><strong>Note</strong></span> no `List-Unsubscribe` header is included in transactional messages. | no | If not specified, the setting at template level is used, or defaults to false. |
|sandbox|boolean|Whether to use the sandbox sending domain | no |Defaults to false. <span class="label label-primary"><strong>SparkPost</strong></span> accounts may use the sandbox |
|skip_suppression|boolean| Whether to ignore customer suppression rules, for this transmission only. | no | <a href="https://www.sparkpost.com/enterprise-email/"><span class="label label-warning"><strong>Enterprise</strong></span></a> Defaults to false. |
| ip_pool | string | The ID of a dedicated IP pool associated with your account. If this field is not provided, the account's default dedicated IP pool is used (if there are IPs assigned to it). | no | <span class="label label-primary"><strong>SparkPost</strong></span> accounts may use IP pools. For more information on dedicated IPs, see the [Support Center](https://www.sparkpost.com/docs/deliverability/dedicated-ip-pools).<br><br><a href="https://www.sparkpost.com/enterprise-email/"><span class="label label-warning"><strong>Enterprise</strong></span></a> accounts, contact your TAM for support details.
|inline_css|boolean|Whether to perform CSS inlining in HTML content<br/><span class="label label-info"><strong>Note</strong></span> only rules in `head > style` elements will be inlined | no - Defaults to false |  |

### Inline Content Attributes

The following attributes are used when specifying inline content in the transmission's `content` JSON object.

<div class="alert alert-warning"><strong>Note</strong>: the following attributes should not be present when using a stored template.</div>
<div class="alert alert-info"><strong>Note</strong>: one of <tt>html</tt>, <tt>text</tt>, or <tt>push</tt> is required. Email transmissions have additional required fields.</div>

| Field         | Type     | Description                           | Required   | Notes   |
|------------------------|:-:       |---------------------------------------|-------------|--------|
|html    |string  |HTML content for the email's text/html MIME part| yes, for email |Expected in the UTF-8 charset with no Content-Transfer-Encoding applied.  Substitution syntax is supported. |
|text    |string  |Text content for the email's text/plain MIME part| yes, for email |Expected in the UTF-8 charset with no Content-Transfer-Encoding applied.  Substitution syntax is supported.|
|push    |JSON object  |Content of push notifications| yes, for push | <a href="https://www.sparkpost.com/enterprise-email/"><span class="label label-warning"><strong>Enterprise</strong></span></a> See [Push Attributes](#header-push-attributes). |
|subject |string  |Email subject line   | yes, for email |Expected in the UTF-8 charset without RFC2047 encoding.  Substitution syntax is supported. |
|from |string or JSON  | Address `"from" : "deals@company.com"` or JSON object composed of the `name` and `email` fields `"from" : { "name" : "My Company", "email" : "deals@company.com" }` used to compose the email's `From` header| yes, for email | Substitution syntax is supported. |
|reply_to |string  |Email address used to compose the email's "Reply-To" header | no | Substitution syntax is supported. |
|headers| JSON | JSON dictionary containing headers other than `Subject`, `From`, `To`, and `Reply-To`  | no |See the [Header Notes](#header-header-notes). |
|attachments| JSON | JSON array of attachments. | no | For a full description, see [Attachment Attributes](#header-attachment-attributes). |
|inline_images| JSON | JSON array of inline images. | no | For a full description, see [Inline Image Attributes](#header-inline-image-attributes). |

#### Push Attributes
The following attributes control the contents of push notifications:

<div class="alert alert-info"><strong><a href="https://www.sparkpost.com/enterprise-email/">SparkPost Enterprise</a></strong> accounts may send push notifications. Contact your TAM for setup assistance.</div>

| Field         | Type     | Description                           | Required   | Notes   |
|------------------------|:-:       |---------------------------------------|-------------|--------|
|apns |JSON object |payload for APNs messages |At a minimum, apns or gcm is required | Used for any push notifications sent to apns devices (See [Multichannel Address attributes](recipient-lists.html#header-multichannel-address-attributes)). This payload is delivered as is. See Apple's [APNs documentation](https://developer.apple.com/library/ios/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/Chapters/TheNotificationPayload.html) for details |
|gcm |JSON object | payload for GCM messages |At a minimum, apns or gcm is required| Used for any push notifications sent to gcm devices (See [Multichannel Address attributes](recipient-lists.html#header-multichannel-address-attributes)). This payload is delivered as is. See Google's [Notification Payload Support](https://developers.google.com/cloud-messaging/http-server-ref#notification-payload-support)

#### Header Notes

* Headers such as `Content-Type` and `Content-Transfer-Encoding` are not allowed here as they are auto generated upon construction of the email.
* The `To` header should not be specified here, since it is generated from each recipient's `address.name` and `address.email`.
* Each header value is expected in the UTF-8 charset without RFC2047 encoding.
* Substitution syntax is supported.

#### email_rfc822 Notes

Alternately, the content JSON object may contain a single `email_rfc822` field. `email_rfc822` is mutually exclusive with all of the above fields.

| Field         | Type     | Description                           | Required   |
|--------------------|:-:       |---------------------------------------|-----------------------|
|email_rfc822    |string  |Pre-built message as specified by the [message/rfc822 Content-Type](http://tools.ietf.org/html/rfc2046#section-5.2.1) |no   |

* Substitutions will be applied in the top-level headers and the first non-attachment `text/plain` and
first non-attachment `text/html` MIME parts only.
* Lone `LF`s and lone `CR`s are allowed. The system will convert line endings to `CRLF` where
necessary.
* The provided `email_rfc822` should NOT be dot stuffed.  The system dot stuffs before sending the outgoing message.
* The provided `email_rfc822` should NOT contain the SMTP terminator `\r\n.\r\n`.  The system always adds this terminator.
* The provided `email_rfc822` in MIME format will be rejected if SparkPost cannot parse the contents into a MIME tree.

### Attachment Attributes

<div class="alert alert-danger">Sending attachments with malicious content is <strong>strictly prohibited</strong> by SparkPost. This includes (and is not limited to) files with `bat` and `exe` extensions.</div>

Attachments for a transmission are specified in the `content.attachments` JSON array where each JSON object in the array is described by the following fields:

| Field         | Type     | Description                           | Required   | Notes   |
|--------------------|:-:       |---------------------------------------|-------------|------------------|
|type |string |The MIME type of the attachment; e.g., `text/plain`, `image/jpeg`, `audio/mp3`, `video/mp4`, `application/msword`, `application/pdf`, etc., including the `charset` parameter (ex: `text/html; charset="UTF-8"`) if needed. The value will apply as-is to the `Content-Type` header of the generated MIME part for the attachment. | yes |  |
|name |string |The filename of the attachment (for example, `document.pdf`). This is inserted into the filename parameter of the `Content-Disposition` header. | yes | Maximum length - 255 bytes |
|data |string |The content of the attachment as a Base64 encoded string.  The string should not contain `\r\n` line breaks.  The SparkPost systems will add line breaks as necessary to ensure the Base64 encoded lines contain no more than 76 characters each. | yes | The entirety of transmission content (text + html + attachments + inline images) is limited to 20 MBs |

### Inline Image Attributes

Inline images for a transmission are specified in the `content.inline_images` JSON array where each JSON object in the array is described by the following fields:

| Field         | Type     | Description                           | Required   | Notes   |
|--------------------|:-:       |---------------------------------------|-------------|------------------|
|type |string |The MIME type of the image; e.g., `image/jpeg`.  The value will apply as-is to the `Content-Type` header of the generated MIME part for the image. | yes |  |
|name |string |The name of the inline image, which will be inserted into the `Content-ID` header. The image should be referenced in your HTML content using `<img src="cid:THIS_NAME" />`. The name must be unique within the `content.inline_images` array. | yes | Maximum length - 255 bytes |
|data |string | The content of the image as a Base64 encoded string.  The string should not contain `\r\n` line breaks.  The SparkPost systems will add line breaks as necessary to ensure the Base64 encoded lines contain no more than 76 characters each. | yes | The entirety of transmission content (text + html + attachments + inline images) is limited to 20 MBs |

### Using a Stored Template

The following attributes are used when specifying a stored template in the transmission's `content` JSON object. Note that these attributes should not be present when using inline content.

| Field         | Type     | Description                           | Required   | Notes   |
|------------------------|:-:       |---------------------------------------|-------------|--------|
|template_id|string| ID of the stored template to use | yes |Specify this field when using a stored template.  Maximum length -- 64 bytes|
|use_draft_template|boolean |Whether to use a draft template|no - defaults to false| If this field is set to `true` and no draft template exists, the transmission will fail.|

### Using a Stored Recipient List

The following recipients attribute is used when specifying a stored recipient list in the transmission. Note that this attribute should not be present when specifying recipients inline.

| Field         | Type     | Description                           | Required   | Notes   |
|------------------------|:-:       |---------------------------------------|-------------|--------|
|list_id | string  | Identifier of the stored recipient list to use | yes | Specify this field when using a stored recipient list. |

### Scheduled Transmissions
**Not available in SparkPost EU.**

Use the `options.start_time` attribute to delay generation of messages.  The scheduled time cannot be greater than 31 days from the time of submission.  If the scheduled time does not pass validation, the transmission is not accepted.  Transmissions with a scheduled time in the past _are_ accepted and undergo immediate generation.

## Create [/transmissions{?num_rcpt_errors}]

### Create a Transmission [POST]

You can create a transmission in a number of ways. In all cases, you can use the `num_rcpt_errors` parameter to limit the number of recipient errors returned.

<div class="alert alert-info"><strong>Note</strong>: Sending limits apply to SparkPost accounts only. When a transmission is created, the number of messages in the transmission is compared to the sending limit of your account. If the transmission will cause you to exceed your sending limit, the entire transmission results in an error and no messages are sent.  Note that no messages will be sent for the given transmission, regardless of the number of messages that caused you to exceed your sending limit. In this case, the Transmission API will return an HTTP 420 error code with an error detailing whether you would exceed your hourly, daily, or sandbox sending limit.</div>

#### Using Inline Email Part Content

Create a transmission using inline email part content.

#### Using Inline RFC822 Content

Create a transmission using inline RFC822 content. Content headers are not generated for transmissions providing RFC822 content. They are expected to be provided as headers contained in the RFC822 content.

#### Using a Stored Recipients List

Create a transmission using a stored recipients list by specifying the `list_id` in the `recipients` attribute.

#### Using a Stored Template

Create a transmission using a stored template by specifying the `template_id` in the `content` attribute.  The `use_draft_template` field is optional and indicates whether to use a draft version or the published version of the template when generating messages.

#### Scheduling Transmissions
**Not available in SparkPost EU.**

Create a scheduled transmission to be generated and sent at a future time by specifying `start_time` in the `options` attribute.

Scheduling a transmission that specifies a stored template will use the LATEST version of the template available at the time of scheduled generation.  The use of published versus draft versions follows the same logic in all transmission requests, whether scheduled or immediate generation. When `use_draft_template` is not specified (or set to false), the latest published version of the specified stored template is used. If `use_draft_template` is set to `true`, the latest draft version is used in the transmission instead.

Once message generation has been initiated, all messages in the transmission will use the template selected at the start of the generation. If a template update is made during the generation of a transmission that uses that template, the template update will succeed but the transmission will continue to use the version that was selected at the start of the generation.


+ Parameters
  + num_rcpt_errors (optional, number, `3`) ... Maximum number of recipient errors that this call can return, otherwise all validation errors are returned.

+ Request Create Transmission using Inline Email Part Content (application/json)

    + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf

    + Body

        {
          "options": {
            "open_tracking": true,
            "click_tracking": true,
            "transactional": false,
            "sandbox": false,
            "ip_pool": "my_ip_pool",
            "inline_css": false
          },
          "description": "Christmas Campaign Email",
          "campaign_id": "christmas_campaign",

          "metadata": {
            "user_type": "students",
            "education_level": "college"
          },

          "substitution_data": {
            "sender": "Big Store Team",
            "holiday_name": "Christmas"
          },

          "recipients": [
            {
              "address": {
                "email": "wilma@flintstone.com",
                "name": "Wilma Flintstone"
              },
              "tags": [
                "greeting",
                "prehistoric",
                "fred",
                "flintstone"
              ],
              "metadata": {
                "age": "24",
                "place": "Bedrock"
              },
              "substitution_data": {
                "customer_type": "Platinum",
                "year": "Freshman"
              }
            }
          ],
          "content": {
            "from": {
              "name": "Fred Flintstone",
              "email": "fred@flintstone.com"
            },
            "subject": "Big Christmas savings!",
            "reply_to": "Christmas Sales <sales@flintstone.com>",
            "headers": {
              "X-Customer-Campaign-ID": "christmas_campaign"
            },
            "text": "Hi {{address.name}} \nSave big this Christmas in your area {{place}}! \nClick http://www.mysite.com and get huge discount\n Hurry, this offer is only to {{user_type}}\n {{sender}}",
            "html": "<p>Hi {{address.name}} \nSave big this Christmas in your area {{place}}! \nClick http://www.mysite.com and get huge discount\n</p><p>Hurry, this offer is only to {{user_type}}\n</p><p>{{sender}}</p>"
          }
        }

+ Response 200 (application/json)

    + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf

    + Body

        {
          "results": {
            "total_rejected_recipients": 0,
            "total_accepted_recipients": 1,
            "id": "11668787484950529"
          }
        }

+ Response 400 (application/json)

        {
          "errors" : [
            {
              "description" : "Unconfigured or unverified sending domain.",
              "code" : "7001",
              "message" : "Invalid domain"
            }
          ]
        }

+ Request Create Transmission with Inline RFC822 Content (application/json)

  + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf

  + Body

        {
          "options": {
            "open_tracking": true,
            "click_tracking": true,
            "transactional": false,
            "sandbox": false,
            "ip_pool": "",
            "inline_css": false
          },
          "description": "Christmas Campaign Email",
          "campaign_id": "christmas_campaign",

          "metadata": {
            "user_type": "students",
            "education_level": "college"
          },

          "substitution_data": {
            "sender": "Big Store Team",
            "holiday_name": "Christmas"
          },

          "recipients": [
            {
              "address": {
                "email": "wilma@flintstone.com",
                "name": "Wilma Flintstone"
              },
              "tags": [
                "greeting",
                "prehistoric",
                "fred",
                "flintstone"
              ],
              "metadata": {
                "age": "24",
                "place": "Bedrock"
              },
              "substitution_data": {
                "first_name": "Wilma",
                "customer_type": "Platinum",
                "year": "Freshman"
              }
            },
            {
              "address": {
                "email": "abc@flintstone.com",
                "name": "Fred Flintstone"
              },
              "tags": [
                "greeting",
                "prehistoric",
                "fred",
                "flintstone"
              ],
              "metadata": {
                "age": "33",
                "place": "NY"
              },
              "substitution_data": {
                "first_name": "Fred",
                "customer_type": "Sliver",
                "year": "Senior"
              }
            }
          ],
          "content": {
            "email_rfc822": "Content-Type: text\/plain\r\nTo: \"{{address.name}}\" <{{address.email}}>\r\n\r\n Hi {{first_name}} \nSave big this Christmas in your area {{place}}! \nClick http://www.mysite.com and get huge discount\n Hurry, this offer is only to {{customer_type}}\n {{sender}}\r\n"
          }
        }

+ Response 200 (application/json)

  + Body

            {
              "results": {
                "total_rejected_recipients": 0,
                "total_accepted_recipients": 2,
                "id": "11668787484950529"
              }
            }

+ Response 400 (application/json)

        {
          "errors" : [
            {
              "description" : "Unconfigured or unverified sending domain.",
              "code" : "7001",
              "message" : "Invalid domain"
            }
          ]
        }

+ Request Create Transmission Using CC Header (application/json)

    + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf

    + Body

        {
          "options": {
            "open_tracking": true,
            "click_tracking": true,
            "transactional": false,
            "sandbox": false,
            "ip_pool": "my_ip_pool",
            "inline_css": false
          },
          "description": "Christmas Campaign Email",
          "campaign_id": "christmas_campaign",
          "recipients": [
            {
              "address": {
                "email": "wilma@flintstone.com"
              }
            },
            {
              "address": {
                "email": "pebbles@flintstone.com",
                "header_to": "wilma@flintstone.com"
              }
            }
          ],
          "content": {
            "from": {
              "name": "Fred Flintstone",
              "email": "fred@flintstone.com"
            },
            "subject": "Big Christmas savings!",
            "reply_to": "Christmas Sales <sales@flintstone.com>",
            "headers": {
              "CC": "pebbles@flintstone.com"
            },
            "text": "Hi, \nSave big this Christmas in your area! \nClick http://www.mysite.com and get huge discount!"
          }
        }

+ Response 200 (application/json)

    + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf

    + Body

        {
          "results": {
            "total_rejected_recipients": 0,
            "total_accepted_recipients": 2,
            "id": "11668787484950529"
          }
        }

+ Response 400 (application/json)

        {
          "errors" : [
            {
              "description" : "Unconfigured or unverified sending domain.",
              "code" : "7001",
              "message" : "Invalid domain"
            }
          ]
        }


+ Request Create Transmission with Stored Recipient List (application/json)

  + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf

  + Body

            {
                "campaign_id": "christmas_campaign",

                "recipients": {
                  "list_id": "christmas_sales_2013"
                },

                "content": {
                  "from": {
                    "name": "Fred Flintstone",
                    "email": "fred@flintstone.com"
                  },

                  "subject": "Big Christmas savings!",

                  "text": "Hi {{name}} \nSave big this Christmas in your area {{place}}! \nClick http://www.mysite.com and get huge discount\n Hurry, this offer is only to {{user_type}}\n {{sender}}",
                  "html": "<p>Hi {{name}} \nSave big this Christmas in your area {{place}}! \nClick http://www.mysite.com and get huge discount\n</p><p>Hurry, this offer is only to {{user_type}}\n</p><p>{{sender}}</p>"
                }
            }

+ Response 200 (application/json)

  + Body

            {
              "results": {
                "total_rejected_recipients": 0,
                "total_accepted_recipients": 10,
                "id": "11668787484950529"
              }
            }

+ Response 404 (application/json)

  + Body

            {
              "errors": [
                {
                  "message": "resource not found",
                  "description": "List 'christmas_sales_2013' does not exist",
                  "code": "1600"
                }
              ]
            }

+ Request Create Transmission with Stored Template (application/json)

  + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf

  + Body

            {
              "options": {
                "open_tracking": true,
                "click_tracking": true
              },

              "campaign_id": "thanksgiving_campaign",

              "content": {
                "template_id": "christmas_offer",
                "use_draft_template": false
              },

              "metadata": {
                "user_type": "students",
                "age_group": "18-35"
              },
              "substitution_data": {
                "status": "shopping",
                "holiday": "Thanksgiving"
              },

              "recipients": [
                {
                  "address": {
                    "email": "wilma@flintstone.com",
                    "name": "Wilma Flintstone"
                  },
                  "tags": [
                    "greeting",
                    "prehistoric",
                    "fred",
                    "flintstone"
                  ],
                  "metadata": {
                    "age": "24",
                    "place": "Bedrock"
                  },
                  "substitution_data": {
                    "first_name": "Wilma",
                    "last_name": "Flintstone"
                  }
                },
                {
                  "address": {
                    "email": "abc@flintstone.com"
                  },
                  "tags": [
                    "greeting",
                    "prehistoric",
                    "fred",
                    "flintstone"
                  ],
                  "metadata": {
                    "age": "33",
                    "place": "MD"
                  }
                }
              ]
            }

+ Response 200 (application/json)

  + Body

            {
              "errors": [
                {
                  "message": "transmission created, but with validation errors",
                  "code": "2000"
                }
              ],
              "results": {
                "rcpt_to_errors": [
                  {
                    "message": "required field is missing",
                    "description": "address.email is required for each recipient",
                    "code": "1400"
                  }
                ],
                "total_rejected_recipients": 1,
                "total_accepted_recipients": 1,
                "id": "11668787484950530"
              }
            }

+ Response 404 (application/json)

  + Body

            {
              "errors": [
                {
                  "message": "resource not found",
                  "description": "template 'christmas_offer' does not exist",
                  "code": "1600"
                }
              ]
            }

+ Request Number of Messages Exceeds Sending Limit (application/json)

  + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf

  + Body

            {
                "campaign_id": "christmas_campaign",

                "recipients": {
                  "list_id": "list_exceeds_sending_limit"
                },

                "content": {
                  "from": {
                    "name": "Fred Flintstone",
                    "email": "fred@flintstone.com"
                  },

                  "subject": "Big Christmas savings!",

                  "text": "Hi {{name}} \nSave big this Christmas in your area {{place}}! \nClick http://www.mysite.com and get huge discount\n Hurry, this offer is only to {{user_type}}\n {{sender}}",
                  "html": "<p>Hi {{name}} \nSave big this Christmas in your area {{place}}! \nClick http://www.mysite.com and get huge discount\n</p><p>Hurry, this offer is only to {{user_type}}\n</p><p>{{sender}}</p>"
                }
            }

+ Response 420 (application/json)

  + Body

            {
              "errors": [
                {
                  "message": "Exceed Sending Limit (hourly)",
                  "code": "2101"
                }
              ]
            }

+ Response 420 (application/json)

  + Body

            {
              "errors": [
                {
                  "message": "Exceed Sending Limit (daily)",
                  "code": "2102"
                }
              ]
            }

+ Response 420 (application/json)

  + Body

            {
              "errors": [
                {
                  "message": "Exceed Sending Limit (sandbox)",
                  "code": "2103"
                }
              ]
            }

+ Request Create Scheduled Transmission (application/json)

  + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf

  + Body

            {
                "name" : "Fall Sale",
                "campaign_id": "fall",

                "options": {
                  "start_time" : "2017-02-11T08:00:00-04:00",
                  "open_tracking": true,
                  "click_tracking": true
                },

                "recipients": {
                  "list_id": "all_subscribers"
                },

                "content": {
                  "template_id" : "fall_deals"
                }
            }

+ Response 200 (application/json)

  + Body

            {
              "results": {
                "total_rejected_recipients": 1000,
                "total_accepted_recipients": 0,
                "id": "11668787484950529"
              }
            }

+ Request Create Transmission with attachments (application/json)

    + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf

    + Body

        {
          "campaign_id" : "attachment_example",
          "recipients": [
            {
              "address": "wilma@flintstone.com"
            }
          ],
          "content": {
            "from": {
              "email": "billing@company.example",
              "name": "Example Company"
            },

            "subject": "Billing statement",
            "html": "<b>Please see your attached billing statement</b>",
            "attachments" : [
              {
                "type" : "application/pdf",
                "name" : "billing.pdf",
                "data" : "Q29uZ3JhdHVsYXRpb25zLCB5b3UgY2FuIGJhc2U2NCBkZWNvZGUh"
              },
              {
                "type" : "text/plain; charset=UTF-8",
                "name" : "explanation.txt",
                "data" : "TW92ZSBhbG9uZy4gIE5vdGhpbmcgdG8gc2VlIGhlcmUu"
              }
            ]
          }
        }

+ Response 200 (application/json)

    + Body

        {
          "results": {
            "total_rejected_recipients": 0,
            "total_accepted_recipients": 1,
            "id": "11668787484950529"
          }
        }

+ Request Create Transmission with inline images (application/json)

    + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf

    + Body

        {
          "campaign_id" : "inline_image_example",
          "recipients": [
            {
              "address": "wilma@flintstone.com"
            }
          ],
          "content": {
            "from": {
              "email": "marketing@company.example",
              "name": "Example Company"
            },

            "subject": "Inline image example",
            "html": "<html><body>Here is your inline image!<br> <img src=\"cid:my_image.jpeg\"></body></html>",
            "inline_images" : [
              {
                "type" : "image/jpeg",
                "name" : "my_image.jpeg",
                "data" : "VGhpcyBkb2Vzbid0IGxvb2sgbGlrZSBhIGpwZWcgdG8gbWUh"
              }
            ]
          }
        }

+ Response 200 (application/json)

    + Body

        {
          "results": {
            "total_rejected_recipients": 0,
            "total_accepted_recipients": 1,
            "id": "11668787484950529"
          }
        }


+ Request Create Transmission for Mobile Push Using Inline Content (application/json)
<div class="alert alert-info"><strong>Note</strong>: The following request is valid for <a href="https://www.sparkpost.com/enterprise-email/">SparkPost Enterprise accounts</a> only.</div>
    + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf

    + Body

        {
          "recipients": [
            {
              "multichannel_addresses": [
                {
                  "channel": "apns",
                  "token": "02c7830aae68d008a0616aed81a6bec40b5acf53fbca1ae46c734527ee0e885f",
                  "app_id": "flintstone.apns.domain"
                }
              ]
            },
            {
              "multichannel_addresses" : [
                {
                  "channel": "gcm",
                  "token": "kNd8dnekej:KDSNDdnedik3n3kFDJfjwJDKndkd39MNiKnd9-Dk4NbkwnyMisosowb_GixnesleE38c1nglc9dTIXL56Djdhsn90nZjkDleEixlndiHk_Sntks54g1sZdnssY2s15f_SnektTkjwse",
                  "app_id": "flintstone.gcm.domain"
                }
              ]
            }
          ],
          "content": {
            "push": {
              "apns" : {
                "aps" : {
                  "alert" : {
                    "title" : "Badge adjust alert message",
                    "body" : "Hello John. I am resetting your badge to zero"
                  },
                  "badge" : 0
                }
              },
              "gcm" : {
                "notification" : {
                  "title" : "You have Android deals",
                  "body" : "Open your Android app to check out these awesome new deals",
                  "color" : "#fa6423",
                  "icon" : "myicon"
                }
              }
            }
          }
        }

+ Response 200 (application/json)

    + Body

        {
          "results": {
            "total_rejected_recipients": 0,
            "total_accepted_recipients": 2,
            "id": "11668787493850529"
          }
        }

## Retrieve and Delete [/transmissions/{id}]

### Retrieve a Scheduled Transmission [GET]
**Not available in SparkPost EU.**

Retrieve the details about a scheduled transmission by specifying its ID in the URI path. 

The response for a transmission using an inline template will include `"template_id":"inline"`.  Inline templates cannot be specifically queried.

<div class="alert alert-info"><strong>Note</strong>: Only multi-recipient transmissions that have been submitted or completed within the last 24 hours are returned.</div>

<div class="alert alert-info"><strong>Note</strong>: Only scheduled transmissions are returned by this endpoint. Immediate transmissions cannot be retrieved.</div>

+ Parameters
    + id (required, number, `11714265276872`) ... ID of the transmission

+ Request

    + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf
            Accept: application/json

+ Response 200 (application/json)

    + Body

        {
          "results": {
            "transmission": {
              "id": "11750520427380741",
              "description": "",
              "state": "Success",
              "campaign_id": "white_christmas",
              "content": {
                "template_id": "Bob's template",
                "use_draft_template": false
              },
              "rcpt_list_chunk_size": 100,
              "rcpt_list_total_chunks": 1,
              "num_rcpts": 10,
              "num_generated": 10,
              "num_failed_gen": 0,
              "generation_start_time": "2014-05-22T15:12:59+00:00",
              "generation_end_time": "2014-05-22T15:13:00+00:00",
              "substitution_data": "",
              "metadata": {
                "is_snowing": "yes"
              },
              "options": {
                "open_tracking": "",
                "click_tracking": ""
              }
            }
          }
        }

+ Response 404 (application/json)

    + Body

            {
              "errors": [
                {
                  "message": "resource not found",
                  "description": "Resource not found:transmission id 123",
                  "code": "1600"
                }
              ]
            }

### Delete a Transmission [DELETE]
**Not available in SparkPost EU.**

Delete a transmission by specifying its ID in the URI path.

Only transmissions which are scheduled for future generation may be deleted.

<div class="alert alert-warning">Scheduled transmissions cannot be deleted if the transmission is within 10 minutes of the scheduled generation time.</div>


+ Parameters
    + id (required, string, `11714265276872`) ... ID of the transmission

+ Request

    + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf
            Accept: application/json

+ Response 204

+ Response 404 (application/json)

  + Body

          {
            "errors": [
              {
                "message": "resource not found",
                "code": "1600",
                "description": "Resource not found:transmission id 999999999"
              }
            ]
          }

+ Response 409 (application/json)

  + Body

          {
            "errors": [
              {
                "message": "too close to generation time to delete transmission",
                "code": "2003",
                "description": "Deletion time window (660 seconds) doesn't permit transmission deletion"
              }
            ]
          }

+ Response 409 (application/json)

  + Body

          {
            "errors": [
              {
                "message": "transmission database record is in an invalid state for deletion",
                "code": "2006",
                "description": "Unable to delete a transmission that is in progress (state=Generating)"
              }
            ]
          }

+ Response 409 (application/json)

  + Body

          {
            "errors": [
              {
                "message": "transmission database record is in an invalid state for deletion",
                "code": "2006",
                "description": "Unable to delete a transmission that has completed (state=Success)"
              }
            ]
          }

## Delete Transmissions By Campaign [/transmissions?campaign_id={campaign_id}]

<div class="alert alert-info"><strong>Note:</strong> SparkPost Enterprise only, account-specific configuration option.</div>


Delete all transmissions of a campaign by specifying Campaign ID in the URI path.

  + Parameters

    + campaign_id (required, string, `white-christmas`)

### Delete By Campaign ID [DELETE]


+ Request Delete all transmissions for a campaign


  + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf
            Accept: application/json

+ Response 204

## List [/transmissions{?campaign_id,template_id}]

### List All Scheduled Transmissions [GET]
**Not available in SparkPost EU.**

List an array of live transmission summary objects.  A transmission summary object contains `id`, `state`, `template_id`, `campaign_id` and `description` fields. 

By default, the list includes transmissions for all campaigns and templates.  Use the `template_id` parameter to filter by template and `campaign_id` to filter by campaign. The summary for transmissions using an inline template will include `"template_id": "inline"`.  Transmissions using inline templates cannot be filtered with `template_id`.

<div class="alert alert-info"><strong>Note</strong>: Only multi-recipient transmissions that have been submitted or completed within the last 24 hours are returned.</div>

<div class="alert alert-info"><strong>Note</strong>: Only scheduled transmissions are returned by this endpoint. Immediate transmissions cannot be retrieved.</div>


+ Parameters
  + campaign_id (optional, string,`thanksgiving`) ... ID of the campaign used by the transmissions
  + template_id (optional, string, `thanksgiving-template`) ... ID of the template used by the transmissions

+ Request

    + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf
            Accept: application/json

+ Response 200 (application/json)

    + Body

        {
          "results": [
            {
              "content" : {
                "template_id": "winter_sale"
              },
              "id": "11713562166689858",
              "campaign_id": "thanksgiving",
              "description": "",
              "state": "submitted"
            },
            {
              "content" : {
                "template_id": "inline"
              },
              "id": "11713562166689979",
              "campaign_id": "thanksgiving",
              "description": "",
              "state": "submitted"
            },
            {
              "content" : {
                "template_id": "thanksgiving-template"
              },
              "id": "11713048079237202",
              "campaign_id": "thanksgiving",
              "description": "",
              "state": "submitted"
            }
          ]
        }
