# 基于IDEA-搭建的go开发环境

现在Docker实在是太流行了，碰上现在工作中接触到很多docker相关的东西。深入了解docker也是非常有必要的。就像为了熟悉Rails就得学习Ruby一样，go语言学习是迈步过去的坎。

下面是记录过程，在mac平台下，使用IDEA搭建GO语言的环境。(*初级入门，熟悉的同学不要继续往下看了*)

## 安装go （mac）
1. mac用户，一般推荐使用brew安装软件。执行 `brew install go`
2. 配置环境变量：
   
```
export GOROOT=/usr/local/go
export PATH=/usr/local/go/bin:$PATH
```

## IDEA整合Go （IDEA）
- 通过plugin manager，安装go插件。如下图，搜索出来的结果：

![](/media/content/BlogPost/images/14638228590190.jpg)￼

- 重启IDEA，新建一个go project。配置go sdk。 通过project structure检查sdk。
![](/media/content/BlogPost/images/14638505606335.jpg)￼


## 来一个简单的helloworld

- 简单的代码：

```
package main

import "fmt"

func main() {
	fmt.Printf("--------Hello, world; ----------")
}
```

- 运行效果如下：
![](/media/content/BlogPost/images/14638510873693.jpg)￼

- 注意gosample运行配置，需要指定main文件：
![](/media/content/BlogPost/images/14638512029126.jpg)￼


docker源码也是基于IDEA进行开发的，所有从github上将源码clone下来阅读，体验也是很不错的。
