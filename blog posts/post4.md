# Making my own website using GitHub Pages
At this point in my job search, I think it's important to market myself using the tools that I want to display proficency in. In my [last post](https://medium.com/@austinspivey1/my-foray-into-data-visualization-using-qlik-sense-9e703b830e9a), I used Qlik Sense to this effect, showcasing my newfound skills using the program. In this post, I'll be debriefing on my (second) attempt to create a simple, static website using the Jekyll platform hosted on GitHub Pages.  

#### Still a beginner
Though I feel confident that I'm more technologically inclined that the average person, I'm still very shaky on anything related to coding and developing. As I continue to learn more and more each day, though, I realize the importance of not only showcasing what I learn for others, but also create a sort of portfolio that I can look back at to track my own progress and occasionally reassess what I should be focusing my scholastic efforts on. So, after doing some Googling, I learned that I qualify for the GitHub Student Developer Pack, which granted me access to a huge number of tools to get started in the realm of programming. One of the many advantages of this package was that I get my own .me domain free for a year, which included a jumpstart option to link the domain with GitHub Pages.  

#### Git a clue
Though I still don't understand everything about GitHub just yet, I am happy to report that I know barely enough to use it! In fact, I draft all of these blog posts in GitHub gists to take advantage of its Markdown capabilities. My first step in using Pages was to set up a repository named after my account name. After that was done, it was time to find a Jekyll theme that I liked and thought would be fun to customize. I ended up choosing the [Jekyll Cayman](https://github.com/pages-themes/cayman) theme, as the colors were appealing and it was nice and roomy, so to speak.  

Now, those who regularly use GitHub may be mortified to learn that instead of just cloning the Cayman theme and, thus, copying its files to my repository, I instead chose to be judicious about what I moved over. For instance, the theme's Git page had Gemfiles and a script folder that I didn't want, among other files and folders. So, I took the extra 5 minutes to only copy over what I wanted.  

#### Index, sweet index
The next step was to make the index page, `index.md` I opted for a Markdown file because Jekyll allowed it and I don't actually know how to code in HTML. I mean, I can look through an HTML document and report back with the general idea of it, but I don't any of the actual theory or nitty-gritty syntax behind it. For my index page, I included the title and layout in the YAML front matter (Jekyll-friendly variables can be referenced here). Then, I added in a basic Welcome To My Website-type of greeting.  

#### Accidental Dev
Great, now my index page looks pretty snazzy and I think I'm ready to jam the brakes for now until I create more blog posts and projects to display.  

But...I would love to add my headshot at the top of the page. This is where I wish I had *some* amount of front-end coding experience so that I could make this change quickly and correctly. Alas, I'm beholden to Stack Overflow, which, honestly, is probably better. Yeah, you know what? It's better. For now. Whatever, I'll learn that stuff eventually.  

Of what little I know, I know that this problem can be solved with HTML and CSS. A quick Google search confirms this suspicion and I find [the code to make it all happen](https://codepen.io/mephysto/pen/wmXOXZ). Here's what I have after I strip it down a bit:  

HTML: 
```HTML
<div class="circular"><img src="http://link-to-your/image.jpg" alt="" /></div>
```

CSS:
```CSS
.circular {
	width: 300px;
	height: 300px;
	border-radius: 150px;
	-webkit-border-radius: 150px;
	-moz-border-radius: 150px;
	background: url(http://link-to-your/image.jpg) no-repeat;
	}

.circular img {
	opacity: 0;
	filter: alpha(opacity=0);
	}
```

Nothing too crazy. At first, my headshot was super screwed up and off to the side, but what it needed was some code that I figured out with a few more Google searches:  

```CSS
background-position: 50% 50%;
background-size: 200px 200px;
margin: auto;
```
Basically, I scaled it down to a smaller size, centered the image, and then centered the object.

#### The take-away

Making my own website was a bit more involved than I initially thought it would be. I'm happy to have gotten some exposure to HTML and CSS, as they're fun to play with and are the essential building blocks to any good web-based project. I'm excited to build it out and add even more to it (with plenty of visits to StackOverflow along the way).  

Thanks for reading!
