# Welcome to SalsaBlog

## Not -Werror considered harmful 
Fri Oct 1, 2021

For the majority of my career in the private sector (think closed source), I've taken it for granted that -Wall -Weverything -Werror are always enabled. Projects I found have VC/gcc/clang/Coverity/clang scan-build on day 1, and pull requests to main need a clean build before merging. We also expect 75%+ (ideally 100%) code coverage from automated tests, which also run with the build. So it surprised me to learn that some (many?) folks in the open source and academic world hate -Werror with passion.

Here are the Pros and Cons as I see them:

### Pros:
1. The codebase is free of bugs that are so obvious a robot with a 1s timeout can find them.
2. We don't ship bugs that simply require a line of code to run to trigger
3. Devs level up, learn and get better
4. The code comes out much tighter. Dead code gets deleted.
5. The code is more portable across OS, processor architectures, and compilers

### (Apparent) Cons:
1. Pull requests can take a significantly longer to merge, especially if they need multiple rounds with the builders
2. You may lose the merge race in fast moving codebases
3. Outsiders or new developers may struggle due to development environment differences
4. Iteratively developing and building locally with -Werror is super frustrating and unnecessarily time consuming

### What can we do about the cons?
#### 1. Not a lot except to realize this is actually a Pro.
Left shift is the buzz word that's all the rage right now (usually in presos the following list is a graphic of boxes from left to right, hence left shift). When you consider the cost of a defect, a great first order approximation is that a bug found during each phase of software development costs about 10x the prior phase. This works with time as well, the further along the bug gets the more it costs and the more people it disrupts.

Oversimplified example:
1. At the developers desk $10
2. At the build machine $100
3. By colleagues in development build $1,000
4. During QA $10,000
5. Customer $100,000
6. Banner grabbing 0day $1,000,000

These numbers might sound crazy but 5 people spending 2 hours who make hundreds of dollars an hour is on the lower end by step 3. More often it's a dozen people independently refinding the bug, re-notifying the owner, and totally DOSed in the mean time. All annoyed and frustrated on top of it.

Meanwhile, in a Github/Bitbucket workflow, this need not actually impose a lot of friction on the poor soul who just wants their code to merge. The reality is that branch is sitting in Bitbucket for anyone else to pull/cherry-pick/rebase on top of locally as needed. Assuming the build works, the artifacts are available for use as well. There's really no downside to holding off that merge until it's ready. It's all upside to disrupt as few people as possible with broken code.

#### 2. 

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
