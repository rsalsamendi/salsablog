## Not -Werror considered harmful 
Fri Oct 1, 2021

For the majority of my career in the private sector (think closed source), I've taken it for granted that -Wall -Weverything -Werror are always enabled. Projects I found have VC/gcc/clang/Coverity/clang scan-build on day 1, and pull requests to main need a clean build before merging. We also expect 75%+ (ideally 100%) code coverage from automated tests, which also run with the build. So it surprised me to learn that some (many?) folks in the open source and academic world hate -Werror with passion.

Here are the Pros and Cons as I see them:

Pros:
- The codebase is free of bugs that are so obvious a robot with a 1s timeout can find them.
- We don't ship bugs that simply require a line of code to run to trigger
- Devs level up, learn and get better
- The code comes out much tighter. Dead code gets deleted.
- The code is more portable across OS and compilers

(Apparent) Cons:
- Pull requests can take a significantly longer to merge, especially if they need multiple rounds with the builders
- You may lose the merge race in fast moving codebases
- Outsiders or new developers may struggle due to development environment differences
- Iteratively developing and building locally with -Werror is super frustrating and unnecessarily time consuming



### Markdown

Markdown is a lightweight and easy-to-use syntax for styling your writing. It includes conventions for

```markdown
Syntax highlighted code block

# Header 1
## Header 2
### Header 3

- Bulleted
- List

1. Numbered
2. List

**Bold** and _Italic_ and `Code` text

[Link](url) and ![Image](src)
```

For more details see [GitHub Flavored Markdown](https://guides.github.com/features/mastering-markdown/).

### Jekyll Themes

Your Pages site will use the layout and styles from the Jekyll theme you have selected in your [repository settings](https://github.com/rsalsamendi/salsablog/settings/pages). The name of this theme is saved in the Jekyll `_config.yml` configuration file.

### Support or Contact

Having trouble with Pages? Check out our [documentation](https://docs.github.com/categories/github-pages-basics/) or [contact support](https://support.github.com/contact) and we’ll help you sort it out.
