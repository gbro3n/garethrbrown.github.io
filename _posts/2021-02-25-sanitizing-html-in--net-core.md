---
public: true
layout: post
title: "Sanitizing HTML in .NET Core"
date: 2021-02-25 00:00 +0000
tags: html net-core
---

Looking through options for sanitizing HTML, I found my way to the following library:

[https://github.com/mganss/HtmlSanitizer](https://github.com/mganss/HtmlSanitizer)

HtmlSantizer uses a whitelist approach to HTML sanitization. A whitelist approach to HTML sanitization is more secure in that there is less scope for missing dangerous tags and attributes. It also works well in a markdown context where a limited set of known tags will make up the output HTML.

If you want to allow additional tags and attributes to remain in the output HTML, you can configure the `HtmlSanitizer`class as follows:

```csharp
var sanitizer = new HtmlSanitizer();  
sanitizer.AllowedAttributes.Add("class");  
var sanitized = sanitizer.Sanitize(html);
```

Rick Strahl’s blog provides a good overview of some of the concerns in the following two blog posts:

- https://weblog.west-wind.com/posts/2012/jul/19/net-html-sanitation-for-rich-html-input
- https://weblog.west-wind.com/posts/2018/Aug/31/Markdown-and-Cross-Site-Scripting