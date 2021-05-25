---
title: "How I used Amazon SES with MJML to create email templates in my side-project"
description: "Short story about Amazon SES"
date: 2020-11-06 00:00:00 +00:00
modified: 2020-11-06 00:00:00 +00:00
tags: [amazon-ses,mjml,template]
---

# How it started

I wanted to send emails to my users on [https://weekhabit.paraboly.com](https://weekhabit.paraboly.com). 


# First attempt: mail sending services

Of course, I started with mail sending services like SendGrid, Mailchimp, SendInBlue. They are awesome services but I faced with **various difficulties**:
- one was asking to buy a block of emails while I wanted to fly in free limits because user base is small now.
- in one I couldn't even register
- one had small UX error where I couldn't test my mail template because I removed sender but I couldn't understand it from error.

# Another attempt: Amazon SES

For introduction, Amazon SES 
- is email service, supporting sending emails from CLI and SDKs.
- doesn't have template editor to create template. All templates must be created by users.

As any programmer, I wanted to code this mail sending. 😀

Here is ther overview of CLI(AWS CLI must be installed and made login) commands for SES(more information can found in SES docs)

```shell
aws ses list-templates
aws ses get-template --template-name simple-template
aws ses create-template --cli-input-json file://simple-template.json
aws ses update-template --cli-input-json file://simple-template.json
aws ses send-templated-email --cli-input-json file://simple-template-single-user.json
aws ses send-bulk-templated-email --cli-input-json file://simple-template-bulk-users.json
```

Example of `simple-template.json`
```json
  {
    "Template": {
      "TemplateName": "simple-template",
      "SubjectPart": "Greetings, {{name}}!",
      "HtmlPart": "<h1>Hello {{name}},</h1><p>Your favorite animal is {{favoriteanimal}}.</p>",
      "TextPart": "Dear {{name}},\r\nYour favorite animal is {{favoriteanimal}}."
    }
  }
```

Example of `simple-template-single-user.json`
```json
  {
    "Source":"WeekHabit Team <week-habit-team@paraboly.com>",
    "Template": "mjml",
    "ConfigurationSetName": "ConfigSet",
    "Destination": {
      "ToAddresses": [ "someuser@gmail.com"
      ]
    },
    "TemplateData": "{ \"name\":\"Alejandro\", \"favoriteanimal\": \"alligator\" }"
  }
```

**Note**: `ConfigSet` must be created in Amazon SES console otherwise email will not be sent.

## Problem

Did you notice that `HtmlPart` is  very simple and more suitable for system emails where style is not important.

Also Amazon SES doesn't have template editor, so how to create **responsive and stylish** email?(One of many advantages of email services)


# mjml - responsive template framework

The mjml is tool which allows create responsive email. It has nice editor where you can design email template.


![mjml editor](/amazon-ses-mjml-email-template/mjml_editor.png)


Standard workflow for generating email template html:

1. Create template in editor [https://mjml.io/try-it-live](https://mjml.io/try-it-live)
2. Copy to file, let's name `simple-template.mjml`
3. run(I assume that you installed `mjml`) 
```shell
./node_modules/.bin/mjml -r simple-template.mjml -o simple-template.html
```
4. Copy `new-template.html` to `HtmlPart` part of `simple-template.json` described above.
5. Update `template` by running 

```shell
aws ses update-template --cli-input-json file://simple-template.json
```

**Example**:

![mjml editor](/amazon-ses-mjml-email-template/mjml_example.png)


---

> Article on [dev.to](https://dev.to/nurgasemetey/how-i-used-amazon-ses-with-mjml-to-create-email-templates-in-my-side-project-3oi9)
>
> Article on [Medium](https://nurgasemetey.medium.com/how-i-used-amazon-ses-with-mjml-to-create-email-templates-in-my-side-project-8118c88ae047)