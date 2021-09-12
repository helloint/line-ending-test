# Git Line Ending Test
这仓库里有4个文件. `lf.txt`是LF的, `crlf.txt`是CRLF的.  
另外2个svg文件本来是想用于测试git对于文本文件的检测判断用的, 目前暂时没用到.

## Test
设置不同的`core.autocrlf`值, 分别checkout仓库, 查看文件`LF/CRLF`的情况.  
测试结果:  
* `true`:  
文件都变成了`crlf`

* `input` 和 `false`:  
`crlf`的还是`crlf`, `lf`的还是`lf`, 没变.  

测试结果符合下面的示意图(必须的).

## 解释 | git config
Git有个配置文件叫 `core.autocrlf`.
他有3个值: `true | false | input`. 其实还有个神秘的默认值.  
关于这个值的含义, stackoverflow上已经有很多权威解释和讨论, 比如[这个](https://stackoverflow.com/questions/1967370/git-replacing-lf-with-crlf), [这个](https://stackoverflow.com/questions/10418975/how-to-change-line-ending-settings), 还有[这个](https://stackoverflow.com/questions/2517190/how-do-i-force-git-to-use-lf-instead-of-crlf-under-windows). 但我最喜欢第一个讨论里面的这张图, 搬过来看看:  

> How autocrlf works:

```                                             
core.autocrlf=true:      core.autocrlf=input:     core.autocrlf=false:
        repo                     repo                     repo
      ^      V                 ^      V                 ^      V
     /        \               /        \               /        \
crlf->lf    lf->crlf     crlf->lf       \             /          \      
   /            \           /            \           /            \
```
> Here crlf = win-style end-of-line marker, lf = unix-style (and mac osx).

刚才上面提到的那个`神秘的默认值`其实是在你安装Git客户端时, 安装程序根据你的操作系统帮你默认选择了, 而你没注意, 就同意了.
> Default value for core.autocrlf is selected during git installation and stored in system-wide gitconfig (%ProgramFiles(x86)%\git\etc\gitconfig on windows, /etc/gitconfig on linux).

之所以说他是默认值, 是因为当你敲:  
`git config --global core.autocrlf`  
返回为空, 让你误以为没有设置. 但其实只是`git global config`里没配置, 而系统里是有默认值的. 

这个默认值, 在Windows上是`auto`(你仍然可以修改, 但改之前务必需要弄懂它的含义), 在Linux / macOS上是`input`.  
我觉得这个配置是相对比较合理的.  
因为结合我们的实际工作场景: 有些人工作在macOS上, 有些人工作在Windows上, 只要保证仓库里的`text`文件永远是LF的, 那么大家就可以都不受影响的工作.  
既然问题可以简单解决, 那自然不建议再去显示的配置`git config --global core.autocrlf`, 以及进一步的, 在project层面去配置`.gitattributes`.  
简单就是美~

## .gitattributes
可以用来覆盖`git config`. 具体解释可以参考上面的引用文章. 我的建议是, 在没搞清楚之前, 不要用.  

对`.gitattributes`的测试([commit](https://github.com/helloint/line-ending-test/commit/a4bc496ee13b41e285421b4addca3b7136f38c3e)):  
1. 创建`crlf.txt`, 用的`crlf`
2. Windows上default是`true`, 按理说`crlf.txt`上传到仓库以后, 就变成`lf`的了, 接下来即使你设置为`input`, 他也不会变成`crlf`.  
但我通过`.gitattributes`设置`text eol=crlf`强制提交为`crlf`, 使得他在仓库里仍然是`crlf`, 从而可以完成后续的操作.  

注: 协作环境(Team Work)下不要做这样的事情. 这会导致macOS用户checkout到了`crlf`的文件, 从而产生意料之外的问题, 从而破环简单原则.
