---
public: true
layout: post
title: "How to inject Google Adsense In-Article script into your HTML (ASP.NET Core Razor)"
date: 2019-12-24 00:00 +0000
tags: adsense netcore aspnet
---

This is how I solved the problem of how to inject a Google Adsense In-Article script within the HTML content of blog articles on a site that I manage.

I wanted the in article text to appear once, half way down the main body of an article. The challenge was to find a reliable and efficient way of where 'half way' in the article HTML was, and then how to inject the Google Adsense In-Article script HTML.

This is a solution for ASP.NET Core Razor. Explanation in the comments with a helper extension method below.

```html
<div>
	
	@{
	
		var entryHtml = entryViewModel.GetHtml();
	
		// Get count of possible insert locations and if there is a sufficient number,
		// inject ad script
	
		string paragraphEndStr = "</p>";
	
		int countParagraphEnds = entryHtml.Split(paragraphEndStr).Length - 1;
	
        // Only do the insert if we have enough article text (counted by paragraphs)
    
		if (countParagraphEnds > 2)
		{
            // This is the Adsense script to be inserted, replace this with your own, escaping quotes in the HTML
        
			string insertHtml = @"
	
			<div class=""mt-3 mb-3"">
				<script async src=""https://pagead2.googlesyndication.com/pagead/js/adsbygoogle.js""></script>
				<ins class=""adsbygoogle""
				style=""display:block; text-align:center;""
				data-ad-layout=""in-article""
				data-ad-format=""fluid""
				data-ad-client=""ca-pub-XXXXXXXXXXXXXXXX""
				data-ad-slot=""XXXXXXXXXXX""></ins>
				<script>
				(adsbygoogle = window.adsbygoogle || []).push({});
				</script>
			</div>";
	
			// Insert ad script in the middle of content (assuming paragraphs of the same length)
	
			entryHtml = entryHtml.ReplaceNthOccurance(paragraphEndStr, paragraphEndStr + insertHtml, countParagraphEnds / 2);
		}
	}
</div>
```

ReplaceNthOccurance is an extension method fore replacing a specific occurence of a string in text (in our case, the middle paragraph end tag) that looks like this:

```csharp
public static string ReplaceNthOccurance(this string obj, string find, string replace, int nthOccurance)
{
    if (nthOccurance > 0)
    {
        var matchCollection = Regex.Matches(obj, Regex.Escape(find));
        if (matchCollection.Count >= nthOccurance)
        {
            Match match = matchCollection[nthOccurance - 1];
            return obj.Remove(match.Index, match.Length).Insert(match.Index, replace);
        }
    }
    return obj;
}
```

Following this, the article text is modified to include the Adsense script, which causes an ad to be rendered in the middle of the article text.