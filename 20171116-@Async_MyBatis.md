# 2017/11/16
## 一、 Spring中@Async的使用
Spring 3.x之后，内置了@Async来实现多线程异步调用。

**配置方式：**

1. java编程方式

        @Configuration  
        @EnableAsync  
        public class SpringConfig {  
          
            /** Set the ThreadPoolExecutor's core pool size. */  
            private int corePoolSize = 10;  
            /** Set the ThreadPoolExecutor's maximum pool size. */  
            private int maxPoolSize = 200;  
            /** Set the capacity for the ThreadPoolExecutor's BlockingQueue. */  
            private int queueCapacity = 10;  
          
            private String ThreadNamePrefix = "MyLogExecutor-";  
          
            @Bean  
            public Executor logExecutor() {  
                ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();  
                executor.setCorePoolSize(corePoolSize);  
                executor.setMaxPoolSize(maxPoolSize);  
                executor.setQueueCapacity(queueCapacity);  
                executor.setThreadNamePrefix(ThreadNamePrefix);  
          
                // rejection-policy：当pool已经达到max size的时候，如何处理新任务  
                // CALLER_RUNS：不在新线程中执行任务，而是有调用者所在的线程来执行  
                executor.setRejectedExecutionHandler(new ThreadPoolExecutor.CallerRunsPolicy());  
                executor.initialize();  
                return executor;  
            }  
          
        }  
    
2. xml配置方式

        <task:executor id="myexecutor" pool-size="5"  />  
        <task:annotation-driven executor="myexecutor"/>  

**使用方法：**在方法上添加@Async

根据是否有返回值，存在两种情况

1. 无返回值

        @Async  //标注使用  
        public void asyncMethodWithVoidReturnType() {  
            System.out.println("Execute method asynchronously. "  
              + Thread.currentThread().getName());  
        }  

2. 有返回值：在调用一步方法的方法中加一个死循环检测是否返回了值

        @Async 
        public Future<String> asyncMethodWithReturnType() {  
            System.out.println("Execute method asynchronously - "  
              * Thread.currentThread().getName());  
            try {  
                Thread.sleep(5000);  
                return new AsyncResult<String>("hello world !!!!");  
            } catch (InterruptedException e) {  
                //  
            }  
           
            return null;  
        }  

        public void testAsyncAnnotationForMethodsWithReturnType()  
       throws InterruptedException, ExecutionException {  
            System.out.println("Invoking an asynchronous method. "  
              * Thread.currentThread().getName());  
            Future<String> future = asyncAnnotationExample.asyncMethodWithReturnType();  
           
            while (true) {  ///这里使用了循环判断，等待获取结果信息  
                if (future.isDone()) {  //判断是否执行完毕  
                    System.out.println("Result from asynchronous process - " + future.get());  
                    break;  
                }  
                System.out.println("Continue doing something else. ");  
                Thread.sleep(1000);  
            }  
        }  

## 二、 MyBatis动态插入数据
### trim标签实现动态insert
[source1](http://blog.csdn.net/h12kjgj/article/details/55003713)

    <insert id="insertMessage" parameterType="com.sf.ccsp.member.client.request.MessageReq">
           insert cx_customer_message
            <trim prefix="(" suffix=")" suffixOverrides="," >
              ID,MEMBERID,
              <if test='messageClassify != null and messageClassify != "" '>
                 MESSAGEE_CLASSIFY,
              </if>
              <if test='messageCode != null and messageCode != "" '>
                 MESSAGE_CODE,
              </if>
             <if test='messageContent != null and messageContent != "" '>
                 MESSAGE_CONTENT,
             </if>
            </trim>
        <trim prefix="values (" suffix=")" suffixOverrides="," >
          #{id, jdbcType=VARCHAR},
          #{memberId, jdbcType=VARCHAR},
          <if test='messageClassify != null and messageClassify != "" '>
            #{messageClassify, jdbcType=VARCHAR},
          </if>
          <if test='messageCode != null and messageCode != "" '>
            #{messageCode, jdbcType=VARCHAR},
          </if>
          <if test='messageContent != null and messageContent != "" '>
            #{messageContent, jdbcType=VARCHAR},
          </if>
        </trim>  
    </insert>

trim标签的属性

1. prefix：前缀覆盖并增加其内容。也就是给中的sql语句加上前缀；
2. suffix：后缀覆盖并增加其内容。给包裹的sql语句加上后缀；
3. prefixOverrides：前缀判断的条件。取消指定的前缀，如where；
4. suffixOverrides：后缀判断的条件。取消指定的后缀，如and | or.，逗号等。

### if else的替代
[source2](http://beijicy.blog.sohu.com/281460867.html)

当我们需要使用else的时候，可以使用choose来替代。因为MyBatis中没有else标签

    <choose>
        <when test="processStatus != null &amp;&amp; processStatus != '' &amp;&amp; processStatus != '-110'">
           and process_status = #{processStatus,jdbcType=VARCHAR}
        </when>
        <otherwise>
            and process_status != 1
        </otherwise>
    </choose>





