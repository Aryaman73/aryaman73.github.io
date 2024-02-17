# Personal Website

This is just the code for my personal website, hosted using GitHub Pages. 
    
URL: aryaman73.github.io & aryamans.me

Domain thanks to: namecheap.me

### WARNING: Build step disabled
I have disabled the Build step in the GitHub workflow file, due to issues with `resume.pdf` (as seen here: https://github.com/gohugoio/hugo/issues/7087). Remember to run `hugo build --minify` before each push to make sure that the correctly edited files are being built.

# Notes: 

Creating a new post: `hugo new posts/name-of-post.md`
Notion's exporting to markdown works surprisingly well, with headings and code blocks working as-is. Links might require more work.

External Website: Add `https` in URL for external websites. Format is `[link text](https://link.com)`

Image: Format is `![Image Alt Text](/sub-folder-in-static/name-of-file.png)`

Links:

https://gohugo.io/getting-started/quick-start/

https://github.com/adityatelange/hugo-PaperMod/wiki/Features#regular-mode-default-mode

Got tags from here: https://github.com/adityatelange/hugo-PaperMod/wiki/FAQs

Sample YML file: https://github.com/adityatelange/hugo-PaperMod/blob/exampleSite/config.yml#L106s

Hosting on GitHub Pages: https://gohugo.io/hosting-and-deployment/hosting-on-github/ 
