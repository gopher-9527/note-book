# git配置不同的账户
当本地开发时需要同时使用多个git账户时，可以为每一个账户配置对应的ssh，从而实现在本地使用多个git账户对不同的项目进行提交。
### 为每个账号生成不同 SSH key
在mac中，使用下面的指令生产SSH key
```sh
ssh-keygen -t rsa -C "account1@example.com" -f ~/.ssh/id_rsa_1
ssh-keygen -t rsa -C "account2@example.com" -f ~/.ssh/id_rsa_2
```
然后在github中添加对应的ssh key。
### 配置SSH
编辑 ~/.ssh/config：
```sh
Host github-1
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_rsa_1

Host github-2
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_rsa_2
```
### 配置git项目
在对应的git项目中配置remote url
```sh
git remote set-url origin git@github-1:xxx/repo.git
```

完成上述的配置之后，就可以在本地使用不同的帐号对不同的项目进行修改和提交了。

ps: **可以手动指定git使用的ssh key**
```sh
GIT_SSH_COMMAND='ssh -i ~/.ssh/id_rsa_gopher9527' git push --set-upstream origin main
```