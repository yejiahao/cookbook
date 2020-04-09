### CentOS 安装 Git
***git.inst.sh***
```sh
#!/bin/sh

package=$1.tar.gz
instPath=/usr/git

if [[ "$1" != "" ]]
then
    # gettext-devel|perl-devel|zlib-devel: required by 'make' directive
    yum -y install gettext-devel perl-devel zlib-devel
    # curl-devel: required by 'git clone' directive using https
    yum -y install curl-devel
    # download
    echo "downloading ${package}"
    wget -c -P ~/upload/ https://mirrors.edge.kernel.org/pub/software/scm/git/${package}
    echo "downloaded ${package}"
    # extract and install
    tar -zxvf ~/upload/${package} -C ~/ && cd ~/$1/
    ./configure --prefix=${instPath}/$1
    make && make install
    # delete source folder
    rm -rf ~/$1/
    # (create|update) symbolic link
    ln -snf $1/ ${instPath}/home

    exit 0
fi

echo "Usage: $0 [version(e.g., git-2.21.0)]"
```
***/etc/profile (source /etc/profile)***
```properties
# set git profile
GIT_HOME=/usr/git/home
PATH=$PATH:$GIT_HOME/bin
export PATH GIT_HOME
```

### 查看版本
```
git --version
```

### 配置别名
```
git config --global alias.lg "log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit"
git config --global alias.st status
git config --global alias.co checkout
git config --global alias.ci commit
git config --global alias.br branch
git config --global alias.mr merge
git config --global alias.tg tag
```

### 注册git账号，生成公私钥对
```
git config --global user.name "yejiahao"
git config --global user.email "<email addr>"
ssh-keygen -t rsa -C "<email addr>"
```

### 删除远程[分支|标签]
```
git push origin :refs/heads/分支名
git push origin :refs/tags/标签名
```

### Windows 使用 Beyond Compare 作为对比合并工具
***.gitconfig***
```properties
[diff]
tool = bc4
[difftool]
prompt = false
[difftool "bc4"]
cmd = "\"g:/beyond compare 4/BComp.exe\" \"$LOCAL\" \"$REMOTE\""
[merge]
tool = bc
[mergetool]
prompt = false
[mergetool "bc4"]
cmd = "\"g:/beyond compare 4/BComp.exe\" \"$LOCAL\" \"$REMOTE\" \"$BASE\" \"$MERGED\""
```

### merge 和 rebase 区别
```sh
git merge dev-3
git merge dev-1
git merge dev-2

*   1a6f2e0 - (HEAD -> master) Merge branch 'dev-2' (70 seconds ago) <yejiahao>
|\
| * 1556baa - (dev-2) dev-2 1 (20 minutes ago) <yejiahao>
* |   46caa7b - Merge branch 'dev-1' (84 seconds ago) <yejiahao>
|\ \
| * | 1b86b2b - (dev-1) dev-1 1 (19 minutes ago) <yejiahao>
| |/
* | a6590ce - (dev-3) dev-3 1 (2 minutes ago) <yejiahao>
|/
* 01c5fd2 - master 2 (30 minutes ago) <yejiahao>
* 0ee43b7 - master 1 (30 minutes ago) <yejiahao>
```
```sh
git rebase dev-3
git rebase dev-1
git rebase dev-2

* a1e7a33 - (HEAD -> master) dev-3 1 (2 seconds ago) <yejiahao>
* 1ad7cdc - dev-1 1 (2 seconds ago) <yejiahao>
* 1556baa - (dev-2) dev-2 1 (23 minutes ago) <yejiahao>
* 01c5fd2 - master 2 (33 minutes ago) <yejiahao>
* 0ee43b7 - master 1 (33 minutes ago) <yejiahao>
```

### fork 仓库同步
```sh
$ git remote add forked <forked repo url>.git

$ git remote -v
forked    <forked repo url>.git (fetch)
forked    <forked repo url>.git (push)
origin  <own remote repo url>.git (fetch)
origin  <own remote repo url>.git (push)

$ git fetch forked
From <forked repo url>
 * [new branch]      master     -> forked/master

$ git merge forked/master
Updating 81a0fa3..faf9793
Fast-forward
 ...

$ git lg
* faf9793 - (HEAD -> master, forked/master) third ci
* f2d7b5b - second ci
* 81a0fa3 - (origin/master, origin/HEAD) first ci

$ git push
Total 0 (delta 0), reused 0 (delta 0)
To <own remote repo url>.git
   81a0fa3..faf9793  master -> master

$ git lg
* faf9793 - (HEAD -> master, origin/master, origin/HEAD, forked/master) third ci
* f2d7b5b - second ci
* 81a0fa3 - first ci
```