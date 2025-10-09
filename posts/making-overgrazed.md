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
First the GitHub repository needed a name, which means my piece of software needed a name, which was ridiculous because it was never going to be finished or see any use. I settled on a name that referenced the [tragedy of the commons](https://en.wikipedia.org/wiki/Tragedy_of_the_commons), since Cosmo was sitting next to me and I needed to pretend to feel bad for stealing their code that they generously shared with me. Anyway I've changed it enough that I don't think I need to act guilty anymore. Thanks.

### goals
I started writing some code. Implementing the `python-markdown` library was pretty easy:

```python
md = markdown.Markdown(extensions=["footnotes", "fenced_code", "tables"])

def convert_md_to_html(input_file):

    content = input_file.read_text()
    md.reset()
    html = md.convert(content)
    return html
```

But that just turns a bunch of `# Heading`s into `<h1>Heading</h1>`s. I needed it to slot that into an actual HTML `<body>` tag and have a `<head>` and so on and so forth. And I wanted a whole bunch of other features, too: automatically crawling a directory for Markdown, working links and images, the ability to have different pages use different headers and footers, etc. And worst of all, I wanted it to be *user-friendly*.

And not just store the HTML of a page in a giant format string.

So I ended up with a plan:

1. Allow the user to write HTML 'templates' and 'snippets' to insert into those templates. Overgrazed would see a `.md` file, determine the template it wanted to be slotted into, apply snippets to that template, and then convert the Markdown and slot it into the template in the appropriate place.
2. Leave the rest of the user's files alone. Copy the directory, other than Markdown files, into a `_site` folder for deployment (`rsync` to a server or something), then convert the Markdown and put it in the new directory *in the right place* as HTML.
3. Be safe and reasonably easy to use and handle errors properly. Once again, I wasn't used to writing software for other people to use; normally I'd just patch whatever was causing an error message and try it blindly again.
