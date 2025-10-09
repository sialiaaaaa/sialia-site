template: blog

# making a static site generator

## why
I originally built my website with [Cobalt](https://cobalt-org.github.io/), which worked well enough until I disagreed with any of its decisions for how to turn Markdown into HTML. Then I looked into [Jekyll](https://jekyllrb.com/), which is much more popular and much more powerful and written in Ruby so that was a non-starter. So of course I found [Overgrazed](https://github.com/sialiaaaaa/overgrazed/) and that seemed to work exactly the way I wanted it to because I wrote it.

Here is how I did that.

## how
```python
import markdown

html_file.write_text(md.convert(md_file.read_text()))
```

## how for real
"Every static site generator is terrible," I announced. I was sitting on the floor of my living room, laptop on the coffee table in front of me, scrolling a Jekyll documentation and sighing at every mention of a 'gemfile' or 'bundler'.

"I don't use one," announced [Cosmo](https://cosmo.tardis.ac), my very good developer friend. "I just wrote a quick Python script to convert Markdown to HTML."

"But I want all these nice features, like serving the site locally while I work on it or being able to write little hotswappable bits of HTML."

"Oh," said Cosmo. "My script stores the HTML of a page in a giant format string."

After some debate and explanation on both sides, I reached a decision.

"Let me see your code, and I'll see if I can adapt it to my use case."

I received the source of Cosmo's website.

"Remind me how to use `uv`?"

"There's already a `uv.lock` in there, so just `uv run` it."

"No, it needs its own environment. I've decided I'm just making a static site generator."

### name
First the GitHub repository needed a name, which means my piece of software needed a name, which was ridiculous because it was never going to be finished or see any use. I settled on a name that referenced the [tragedy of the commons](https://en.wikipedia.org/wiki/Tragedy_of_the_commons), since
