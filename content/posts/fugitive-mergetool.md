---
title: "Fugitive Mergetool"
date: 2020-03-24T12:32:00-06:00
draft: false
---

For Vim users, [Fugitive](https://github.com/tpope/vim-fugitive) is an amazing plugin to integrate Vim with Git. Below is how you can use Vim and Fugitive as your Git mergetool. I learned this from the post [here](https://coderwall.com/p/qbtnsw/use-fugitive-as-git-mergetool). I am putting it here too because there needs to be more of this on the internet.
```
git config --global mergetool.fugitive.cmd 'vim -f -c "Gdiff" "$MERGED"'
git config --global merge.tool fugitive
```

On another note, I just learned that a personal blog is a great place to put info that *I* don't want to forget. :):
