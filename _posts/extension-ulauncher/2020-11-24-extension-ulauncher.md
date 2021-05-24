---
title: "How I wrote Ulauncher extension for most used timedate macros"
description: "Soccer tool for ..."
date: 2020-11-24 00:00:00 +00:00
modified: 2020-11-24 00:00:00 +00:00
keywords: [amazon-ses,mjml,template]
---

# Why?

I wrote a lot. And in my writings I use timestamps, dates, times, time-ranges.

I use following datetime formats
- 2020-11-01
- 15-18
- 14:58

For them I used custom bash scripts which were copying to clipboard the date formats I wanted. These scripts were manually added to Linux's custom shortcuts. 

Example of bash script:

```shell
#!/bin/sh
alias setclip='xclip -selection c'
alias getclip='xclip -selection clipboard -o'
printf $(date +"%Y-%m-%d") | setclip
```

Initially, I was really happy but after some time number of custom scripts increased. I had difficulties with remembering them.

![chelsea](/extension-ulauncher/ulauncher-extension-shortcuts.png){:class="img-responsive"}


# How?

I stumbled to [Ulauncher](https://ulauncher.io/) and it is downed on me that I could create extension for these timedate scripts.

For people who don't know, [Ulauncher](https://ulauncher.io/) is application launcher to which many developers write many extensions. 

Some examples:
- emoji extension
- process murderer
- google translate
- spotify player


Here is the tutorial [link](http://docs.ulauncher.io/en/latest/extensions/tutorial.html) for extension development.

# About my extension

Ulauncher requires that extension must have `manifest.json` file to operate.

Example of `manifest.json`

```json
{
  "required_api_version": "^2.0.0",
  "name": "Timestamp macros",
  "description": "Copy to clipboard the most used timedate formats",
  "developer_name": "Nurgazy Nazhimidinov",
  "icon": "images/icon.png",
  "options": {
    "query_debounce": 0.05
  },
  "preferences": [
    {
      "id": "time_kw",
      "type": "keyword",
      "name": "Timemacros",
      "default_value": "tm"
    }
  ]
}

```

It says that if you write keyword **Timemacros**, extension will be on launcher(**WARNING** not clicked).

![chelsea](/extension-ulauncher/ulauncher-extension-keyword.png){:class="img-responsive"}


---

if you write **Timemacros** and click 

or 

write **tm** and `SPACE`, then `KeywordQueryEventListener(EventListener)` will invoked.

Here is the code:

```python
class DemoExtension(Extension):

    def __init__(self):
        super(DemoExtension, self).__init__()
        self.subscribe(KeywordQueryEvent, KeywordQueryEventListener())

class KeywordQueryEventListener(EventListener):

    def on_event(self, event, extension):
        items = []
        logger.info('preferences %s' % json.dumps(extension.preferences))
        logger.info(event.get_keyword()) # gives the keyword 'tm'
        items.append(ExtensionResultItem(icon='images/icon.png',
                                         name='YYYY-MM-DD',
                                         description='{0:%Y-%m-%d}'.format(datetime.datetime.now()),
                                         on_enter=CopyToClipboardAction('{0:%Y-%m-%d}'.format(datetime.datetime.now()))))

        items.append(ExtensionResultItem(icon='images/icon.png',
                                         name='HH:mm',
                                         description='{0:%H:%M}'.format(datetime.datetime.now()),
                                         on_enter=CopyToClipboardAction(
                                             '{0:%H:%M}'.format(datetime.datetime.now()))))

        items.append(ExtensionResultItem(icon='images/icon.png',
                                         name='YYYY-MM-DD HH:mm',
                                         description='{0:%Y-%m-%d %H:%M}'.format(datetime.datetime.now()),
                                         on_enter=CopyToClipboardAction(
                                             '{0:%Y-%m-%d %H:%M}'.format(datetime.datetime.now()))))
            
        return RenderResultListAction(items)


if __name__ == '__main__':
    DemoExtension().run()

```

As you can see `KeywordQueryEventListener(EventListener)` and `on_event` is run and `RenderResultListAction(items)` is returned and presented to user.

Our `items` would be
- YYYY-MM-DD 
- HH:mm 
- YYYY-MM-DD HH:mm 

Here is the screenshot of items:

![chelsea](/extension-ulauncher/ulauncher-extension-items.png){:class="img-responsive"}


Code for item:

```python
items.append(ExtensionResultItem(icon='images/icon.png',
                                         name='YYYY-MM-DD',
                                         description='{0:%Y-%m-%d}'.format(datetime.datetime.now()),
                                         on_enter=CopyToClipboardAction('{0:%Y-%m-%d}'.format(datetime.datetime.now()))))

```

Explanation for `ExtensionResultItem`: 

`name` - title shown to user, look at screenshot above

`description` - subtext which is also shown to user, look at screenshot above

`on_enter` - most important part, says what to do if user clicks on this items. In my case, it is `CopyToClipboardAction` action which copies data to clipboard.


# Summary

At the end, I developed extension with enabled me to discard my custom bash scripts and forget about keyboard shortcuts for them.

[Source code on Github](https://github.com/nurgasemetey/ulauncher-timestamp-macros)


Deprecated bash scripts 😄

![chelsea](/extension-ulauncher/ulauncher-extension-bash-scripts.png){:class="img-responsive"}


---

> [Article on dev.to](https://dev.to/nurgasemetey/how-i-wrote-ulauncher-extension-for-most-used-timedate-macros-c5i)
> 
> [Article on Medium](https://nurgasemetey.medium.com/how-i-wrote-ulauncher-extension-for-most-used-timedate-macros-f6ad14ee0bad)