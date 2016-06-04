# git rebase，避免无谓的merge


最近一个项目，我们有这么一个场景：
>    有3台服务，server a,b,c。当我们在a上add文件并push到git server，然后通知server b,c去本地更新自己的本地仓库，并将3台server上最终的指针同步到ZK上，当3台server上的指针满足2台以上为最新的指针的时候，就认为这次操作成功。
 
但在实际操作中，会发现很难出现指针一致的情况，一般会出现3个不一样的指针，具体如下：


```
server  a: current  head  :  aaaaa  —>  add   -> commit —> push —> last head: abcde
server  b: current  head  :  aaaaa  —>  pull  —> (head: abc —-> last head : ddddd)
Server  c: current  head  :  aaaaa  —>  pull  —> (head: abc —-> last head : eeeee)
```
 
通过观察server b和c上的git log，会发现这样一个现象：server b和c在进行pull操作，一般会先讲git server上最新指针拉到本地，但随后又会主动做一个merge操作，并产生新的指针，而这一次merge操作其实没有任何意义，*gitpull为什么会有这样的一个操作？*
 
git pull在设计之初，它的行为就是将远端的仓库和本地仓库进行合并，这种就是类似与将2个同名称的branch做merge，但如果这2个branch是完全同步的，那么，这个merge动作就没有任何的意义，也会导致我们上面的这个项目中一致性需求难以满足。
 
在本地测试了下，发现可以使用gitfetch + gitrebase来解决这个问题：

原来使用gitpull的做法：


```
Git git = gitInit(localPath);
PullResult pullResult;
try {
	pullResult = git.pull().call();
} catch (GitAPIException e) {
	logger.error(GIT_API_ERROR + “, pull Exception”, e);
}
```
 
使用rebase之后的做法:
        
```
RebaseResult rebaseResult;
try {            
	Git git = gitInit(localPath);
	fetch(localPath);   
	ObjectId head = git.getRepository().getRefDatabase().getRef(“refs/remotes/origin/” + branch).getObjectId(); 
	rebaseResult = git.rebase().setUpstream(head).call();
 } catch (Exception e) { 
	logger.error(GIT_API_ERROR + “, rebase Exception”, e);
}
```
 
 
参考：
**git-rebase Manual Page** : [https://www.kernel.org/pub/software/scm/git/docs/git-rebase.html](https://www.kernel.org/pub/software/scm/git/docs/git-rebase.html)
**使用git rebase避免无谓的merge** : [http://ihower.tw/blog/archives/3843](http://ihower.tw/blog/archives/3843)

