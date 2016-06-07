# 分布式系统测试–使用HttpServer的一个并发问题

上周发布的一个系统，出现了一个很诡异的现象。抽象一下描述,问题大概就是这样的：

> *需求：*
> 一次http请求，通过url的params来读取服务器上的一个日志，并将日志内容返回给用户。

> *问题表现：*
> 存在一定的机率，一次请求返回的内容，与期望的内容不一致。也就是所谓的串日志问题。
 
这个问题出现后，我们都认为后台请求应该没有问题，因为我们是直接用的JDK自带的HttpServer来起的服务，心中对JDK还是有一些信任的。所以我们还是前端发送请求的时候发乱了，返回的结果和请求是能够匹配的。
 
为了定位问题，我们在后台加了些日志，对比了每次请求和请求的返回内容。在1~2天的监控下，发现真的出现了请求和返回内容之间存在不一致的地方，而前端发送的请求没有错，问题直接定位到后台处理的问题。
 
简单说来，大致就类似这样：前端发送请求  http://127.0.0.1/getlog?id=123&path=testlog,期望是读取的是testlog文件，但是实际确是把devlog的内容返回给前端了。由于定位到这个现象，问题也就可以定位到后台读文件拿参数是错的。
 
我们怎么获取url中的参数呢，翻下源码，如下：

```
Map<String, String> Params = (Map<String, String>);
httpExchange.getAttribute(“parameters”); 
 
```

JDK上是这么定义这个方法的，getAttribute：
> Filter modules may store arbitrary objects with HttpExchange instances as an out-of-band communication mechanism. Other Filters or the exchange handler may then access these objects. 
 
并没有说他存在线程不安全，至于到底是不是这里出现的问题了，写了一个测试程序：
1000并发，下发10000个请求，看看出现多少次这样的问题。结果很明显，统计下来，发现存在这种问题的数据接近1000条左右，这问题就非常明显了。
 
httpExchange.getAttribute()在一定并发的情况下，存在线程不安全的问题。  
 

```
public class HttpServersPerfTest extends BaseCase {
 
    private static final Log logger = LogFactory.getLog(NodeHttpServersPerfTest.class);
    int maxThread = 1000;
    private int totalTask = 1000;
 
    @Test
    public void testRequestRunningLog() throws InterruptedException {
        execute();
    }
 
    private void execute() throws InterruptedException {
        ExecutorService pool = Executors.newFixedThreadPool(maxThread);
 
        final CountDownLatch countDownLatch = new CountDownLatch(totalTask);
 
        for (int n = 0; n < totalTask; n++) {
            pool.execute(new RequestHttpServerThread(countDownLatch));
        }
 
        try {
            countDownLatch.await();
            System.err.println(“==============>>>>> 下发结束” );
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
 
    private class RequestHttpServerThread implements Runnable {
        private CountDownLatch countDownLatch;
 
        public RequestHttpServerThread(CountDownLatch countDownLatch) {
            this.countDownLatch = countDownLatch;
        }
 
        /**
         * 任务执行,这里实际上在服务器上准备好一堆文件，文件内容按行将文件名写入
         * 读取到内容可以和文件名进行比较 
         */
        public void run() {
            Random rd = new Random();
            int index = rd.nextInt(60);
            if (index < 10){
                index = index + 10;
            }
            String indexStr = Integer.toString(index);
            String path = indexStr+”-“+indexStr+”-“+indexStr+”/”;
            LogGetter logGetter = new LogGetter();
            logGetter.setPath(path);
            String runningLog = logGetter.getRunningLog(0L);
 
            if(runningLog.contains( “T3_000000000” + indexStr)){
                System.out.println(“==============>”+”exit”);
            }else
            {
                String[] lines = runningLog.split(“\n”);
                System.out.println(“期望:” +  indexStr + “\n”  + “实际请求的：” + lines[0] + “\n”);
            }
        }
    }
} 
 
```

定位到问题之后，我们就决定放弃使用这个方法，自己重写一个parserQuery的方法，后台拿到url之后重新对url进行参数的解析。
 

```
Map<String, String> params = HttpUtils.parseQuery(httpExchange.getRequestURI().getQuery()); 
 
public static Map<String, String> parserQuery(String query) throws UnsupportedEncodingException {
        Map<String, String> parameters = new HashMap<String, String>();
        if (query != null) {
            String pairs[] = query.split(“[&]”);
            for (String pair : pairs) {
                String param[] = pair.split(“[=]”);
                String key = null;
                String value = null;
                if (param.length > 0) {
                    key = URLDecoder.decode(param[0], System.getProperty(“file.encoding”));
                }
                if (param.length > 1) {
                    value = URLDecoder.decode(param[1], System.getProperty(“file.encoding”));
                }
                parameters.put(key, value);
            }
        }
        return parameters;
    } 
 
```

使用了新的params之后，重新使用1000并发，发10000，50000请求，都无法再出现类似的问题，该问题得到解决。 

