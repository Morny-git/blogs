> \##npm i docsify-cli -g 
>
> docsify init ./docs
>
> docsify serve docs
>
> \##git 
>
> cd docs 
>
> git init 
>
> git add . 
>
> git commit -m "commit" 
>
> git remote add origin https://github.com/Morny-git/blogs.git
>
> 出现错误 remote origin already exists 
>
> git remote rm origin 
>
> 再重新执行 git remote add origin https://github.com/Morny-git/blogs.git 
>
> git push -u origin master 
>
> 出现错误 failed to push som refs to…….， 
>
> git pull origin master
>
> 访问：https://morny-git.github.io/blogs/

或者

```console
git fetch origin
git add .
git commit -m "commit"
git push origin master

```