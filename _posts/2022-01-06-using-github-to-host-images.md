---
public: true
layout: post
title: "Using GitHub to Host Images"
date: 2022-01-06 12:53 +0000
tags: github image-hosting 
---

For simple image hosting scenarios, you can host images directly out of a public GitHub respository by following these steps:

Browse to image on github.com, e.g.

```
https://github.com/appsoftwareltd/app-software-public/blob/main/graphics/logo/as_icon_text_blue_with_grey_text_2x.png
```

Here you will see the GitHub page providing access to view / download the file. But we want the raw file, without GitHub page content wrapping it.

A 'Raw' button doesn't exist for image files, but you can add `?raw=true` to end of URL.

Hit enter. You will be redirected to a URL as below.

```
https://raw.githubusercontent.com/appsoftwareltd/app-software-public/main/graphics/logo/as_icon_text_blue_with_grey_text_2x.png
```
Here is the image as found at the above URL:

<img alt="App Software Logo" src="https://raw.githubusercontent.com/appsoftwareltd/app-software-public/main/graphics/logo/as_icon_text_blue_with_grey_text_2x.png" />

**Note**: The redirect adds the `raw` sub domain part, changes domain to `githubusercontent.com` and removes the `blob` portion of the path.

Test the link in an unauthenticated browser, and then use as required.