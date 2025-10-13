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
Every static site generator is terrible," I announced. I was sitting on the floor of my living room, laptop on the coffee table in front of me, scrolling a Jekyll documentation and sighing at every mention of a 'gemfile' or 'bundler'.

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
md = markdown.Markdown(extensions=[
                            "footnotes",
                            "fenced_code",
                            "tables"
                            ])

def convert_md_to_html(input_file):

    content = input_file.read_text()
    md.reset()
    html = md.convert(content)
    return html
```

But that just turns a bunch of `# Heading`s into `<h1>Heading</h1>`s. I needed it to slot that into an actual HTML `<body>` tag and have a `<head>` and so on and so forth. And I wanted a whole bunch of other features, too: automatically crawling a directory for Markdown, working links and images, the ability to have different pages use different headers and footers, etc. And worst of all, I wanted it to be *user-friendly*.

And not just store the HTML of a page in a giant format string.

So I ended up with a plan:

1. A website would have a site *source directory* and *destination directory*. Overgrazed would copy all files in the source directory to the destination directory. If any of those files were Markdown, they'd be converted to HTML first.

2. When the Markdown was converted to HTML, it'd be formatted into user-defined HTML templates. Those templates would also be able to reference user-defined HTML "snippets" (smaller pieces of HTML like `<head>`s and `<header>`s and `<footer>`s). These "templates" and "snippets" would be stored in the source directory, but ignored by Overgrazed when it was copying everything.

3. Overgrazed wouldn't be dangerous to run. I am far too used to writing scripts for only myself, and letting them `shutil.rmtree(".")` with impunity, trusting myself to point them at the proper directories. I would learn how to handle errors and write reasonably safe code in Python. I might also read the style guide. Y'know, maybe.

### how do you write python
ohmygodi'veneverwrittenusablesoftwarebeforewheredoistart

Okay slow down. There would be no function-writing until I knew how to make a command-line program that worked. First thing I did was learn how to implement `argparse`, the Python module for reading command-line arguments and making them look pretty:

```text
uv run main.py --help
usage: main.py [-h]

options:
  -h, --help         show this help message and exit
```

Well, okay. I added a `verbose` flag[^1] and `input_file` and `output_file` options. Great! Read those into variables, then do the thing in the code blocks above that I've repeated twice now. Markdown to HTML. Done!



Well, no. I actually want to crawl the whole directory- but first I need to do the copying- but- hm. Okay. One thing at a time. I wrote some functions to crawl the directory, find all the Markdown files in it, and call the `convert` function on them. Then it stored that HTML in a string. So that's the first of my goals achieved, to some extent.

### achieving the second goal


All along I used `os` to do all the filesystem stuff, by the way, which turned out to be a mistake. But more on that later. For now, what it meant was I had an *awful* function to figure out the destination to put HTML files in:

```python
def build_site(site_dir, dest_dir):
    shutil.copytree(
        site_dir,
        dest_dir,
        ignore=lambda directory, contents: {name for name in contents if is_ignored_filename(name)}.union({name for name in contents if name.endswith('.md') or name.startswith('_')}),
        dirs_exist_ok=True) # Copy irrelevant files over

    for root, dirs, files in os.walk(site_dir): # Walk the site directory searching for Markdown files
        for filename in files:
            if filename.endswith(".md"):
                current_file_path = root.replace(site_dir, "")
                built_page = build_page(site_dir, os.path.join(root, filename)) # Build the Markdown files into pages
                new_filename = Path(filename).stem + ".html"
                print(f"{dest_dir} {current_file_path} {new_filename}")
                with open(os.path.join(dest_dir, current_file_path, new_filename), "w") as f: # Write the built pages into the same location at the destination
                    f.write(built_page)
```


[^1]: I eventually removed the flag, but not before writing a `vprint()` function which I went on to not use. And all that happened surprisingly late into development. I guess I assumed verbosity would be at some point useful, or I at least wanted to preserve the flag as an example of how to implement it.
