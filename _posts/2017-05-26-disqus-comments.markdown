---
layout: post
title:  "Disque-comments"
date:   2017-05-26 14:49:39 +0100
comments: true
categories: jekyll disqus comments layout
---




It's cool function to have reader comments on your posts in github. So I decided to have this function from disqus in my page as well. I searched on the tutorial about how to add but actually they're not all giving the same script, I first follow the official way to setup which is well documented, then I checked another github blog how he uses, it's better in my opinion.

First I need to register a disqus account and then follow get started section to add my github url into disqus and complete the 3 steps setup.

# step1   

![step1]({{ site.url }}/images/disqusOnGithub_1.jpg)


# step2  
![step2]({{ site.url }}/images/disqusOnGithub_2.jpg)


# step3  
![step3]({{ site.url }}/images/disqusOnGithub_UniEmbeded.jpg)  

Example of universal embeded code:  
```
<div id="disqus_thread"></div>
<script>

/**
*  RECOMMENDED CONFIGURATION VARIABLES: EDIT AND UNCOMMENT THE SECTION BELOW TO INSERT DYNAMIC VALUES FROM YOUR PLATFORM OR CMS.
*  LEARN WHY DEFINING THESE VARIABLES IS IMPORTANT: https://disqus.com/admin/universalcode/#configuration-variables*/
/*
var disqus_config = function () {
this.page.url = {{page.url}};  // Replace PAGE_URL with your page's canonical URL variable
this.page.identifier = {{page.id}}; // Replace PAGE_IDENTIFIER with your page's unique identifier variable
};
*/
(function() { // DON'T EDIT BELOW THIS LINE
var d = document, s = d.createElement('script');
s.src = 'https://https-ydd9-github-io.disqus.com/embed.js';
s.setAttribute('data-timestamp', +new Date());
(d.head || d.body).appendChild(s);
})();
</script>
<noscript>Please enable JavaScript to view the <a href="https://disqus.com/?ref_noscript">comments powered by Disqus.</a></noscript>
```


## [Variables defined by Jekyll and used in configuration](http://jekyllrb.com/docs/variables/)
## [Disqus required variables](https://help.disqus.com/customer/portal/articles/472098-javascript-configuration-variables)
## [Variables declairation rules](http://downtothewire.io/2015/08/15/configuring-jekyll-for-user-and-project-github-pages)


# extra: Example to add comments counts on my page  

```  
<script id="dsq-count-scr" src="//https-ydd9-github-io.disqus.com/count.js" async></script>
```  
![step extra]({{ site.url }}/images/disqusOnGithub_Counts.jpg)


# [Other user's way I think it's better](http://sgeos.github.io/jekyll/disqus/2016/02/14/adding-disqus-to-a-jekyll-blog.html)

You defined a default layout for your web, then create a post layout based on default, this post layout should have a header, i.e. about, title, date, comments count, ... and after contents, it should have a disqus comments. When you add a new post, just specify "comments: true" in the YAML Front Formatter.

For disqus comments, you need to specify it inside _includes/disqus.html its "config"