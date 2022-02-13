# RTOS14
嵌入式实时操作系统14——信号量
## 1.信号量

**操作系统通常有3种的任务通信方式：**

> 1、信号量，用于多任务之间的同步。
>  2、互斥量，用于避免多任务之间共享资源的竞争。 
>  3、消息队列，用于多任务之间的收发消息机制。


**信号量在操作系统中用于实现任务同步**，通过同步机制可以实现多个任务合作，让多任务之间按照先后顺序执行。

**这种机制就像我们生活中的交通红绿信号灯**。汽车停在红绿信号灯路口，当红绿信号灯变成绿灯时，汽车启动并通过路口。这种行为逻辑并不是红绿信号灯亮的时候通过光电效应触发汽车的油门让汽车启动，而是因为司机看到了红绿信号灯变为绿灯，司机踩下油门启动汽车通过路口。**司机看到红绿信号灯变为绿灯，随后踩下油门这个一组动作就是同步**。

![在这里插入图片描述](https://img-blog.csdnimg.cn/1f9316b8ce434601ae077109ec2a3c52.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAbGl5aW51bzIwMTc=,size_18,color_FFFFFF,t_70,g_se,x_16)

信号量是在多任务环境下使用的一种机制, 它负责协调各个任务, 以实现多个任务合作。通常情况下信号量可以分为以下两类：

> 1、二进制信号量：只允许信号量取0或1值，其只能被一个任务获取。
> 2、整型信号量：信号量取值是整数，可以被多个任务同时获得，直到信号量的值变为0。

信号量通过一个计数器控制任务访问，如果信号量的值是一个非负整数，任务获取它后会将该整数减1。如果信号量的计数器大于0，则访问被允许，计数器减1；如果号量的计数器为0，则访问被禁止，所有试图获取它的任务都将处于等待状态。

## 2.信号量操作

信号量的两个基本操作：**释放（发送）信号量，获取（等待）信号量。**

释放信号量的流程如下：
1、判断是否有任务等待该信号量。
2、若有任务等待该信号量，则将任务从挂起表中移除加入就绪表，并根据优先级执行任务切换。
3、若无任务等待该信号量，则将该信号量设置为“有效”。


获取信号量的流程如下：
1、判断该信号量是否为“有效”。
2、若该信号量为“无效”，将任务从就绪表中移除加入挂起表，切换任务。
3、若该信号量为“有效”，则继续运行。

例如任务A需要等待信号甲再执行一个特定操作，当信号甲为“无效”时任务A进入休眠状态，操作系统让任务B运行，假设任务A优先级高于任务B,当任务B释放信号甲后（信号为“有效”），操作系统会暂停任务B运行任务A，使得任务A的特定动作得以执行，运行图如下：

![在这里插入图片描述](https://img-blog.csdnimg.cn/160420d49e4a471e81a8f936662377ef.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAbGl5aW51bzIwMTc=,size_19,color_FFFFFF,t_70,g_se,x_16)


## 3.工程实例分析

假设现在有这样一个项目：利用光电开关实现一个药品流水线上的电子计数仪。当药品经过传感器时仪表计数值加一，显示器显示计数数值，同时使用ESP8266模块将数据发送给服务器。

![在这里插入图片描述](https://img-blog.csdnimg.cn/c6f733e119234de5895d02e7868ca044.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAbGl5aW51bzIwMTc=,size_19,color_FFFFFF,t_70,g_se,x_16)

光电开关的工作原理就是当**有障碍物挡住光信号时光电开关输出高电平**，当**无障碍物挡住光信号时光电开关输出低电平**。

![在这里插入图片描述](https://img-blog.csdnimg.cn/c1b0263216a14003b963dc04ed029427.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAbGl5aW51bzIwMTc=,size_19,color_FFFFFF,t_70,g_se,x_16)

软件使用了分模块的设计方法，每个功能独立成一个模块，提高了软件系统的扩展性。每个功能模块采用了3层的分层设计，底层处理硬件相关操作，中间层用来进行驱动控制和逻辑控制，顶层的用来处理与其他业务模块的数据交互，分层设计提高了软件系统的移植性。

![在这里插入图片描述](https://img-blog.csdnimg.cn/c7736bf1e5c04e7eb1646c01a896c627.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAbGl5aW51bzIwMTc=,size_19,color_FFFFFF,t_70,g_se,x_16)

程序分为3个任务：

> 1、显示任务，完成显示计数的数值的功能。
>  2、WIFI任务，通过WIFI模块将计数数值发送给服务器。
> 3、GPIO任务，使用中断功能捕获光电开关的电平信号。

3个任务的优先级如下：

![在这里插入图片描述](https://img-blog.csdnimg.cn/f7760b7d60eb441fa864b1f3ef3439b5.png)

软件设计思路：当有药品经过光电开关时，产生一个上升沿信号，MCU捕获上升沿中断信进入中断程序更新计数数值（计数值加一)，同时中断程序发送信号量给WIFI任务，WIFI任务将计数数值发给服务器，软件系统在空闲时间内执行显示任务。运行时序图如下：

![在这里插入图片描述](https://img-blog.csdnimg.cn/39086f9e9f394c0dbb2b38f83bbd27e3.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAbGl5aW51bzIwMTc=,size_19,color_FFFFFF,t_70,g_se,x_16)

由时序图可系统运行时，WIFI任务优先级较高先运行，WIFI任务等待信号时进入休眠状态，操作系统进行任务切换执行低优先级显示任务。当光电开关产生有效电平信号后GPIO中断程序开始执行，中断程序将计数值加一（全局变量），然后向WIFI任务发送信号，WIFI任务被执行，WIFI任务向服务器发送计数数值后等待信号量，WIFI任务进入休眠状态，操作系统进行任务切换执行低优先级显示任务。

## 4.信号量代码分析

参考FreeRTOS源码（删减部分代码），分析发送信号量函数和获取信号量函数的源码，这两个函数名如下：

```c
#define xSemaphoreGive( xSemaphore )		xQueueGenericSend( ( QueueHandle_t ) ( xSemaphore ), NULL, semGIVE_BLOCK_TIME, queueSEND_TO_BACK )
xSemaphoreGive( xSemaphore )

#define xSemaphoreTake( xSemaphore, xBlockTime )		xQueueSemaphoreTake( ( xSemaphore ), ( xBlockTime ) )
xSemaphoreTake( xSemaphore, xBlockTime )
```


**发送信号量函数分析如下（删减部分代码）：**

```c
BaseType_t xQueueGenericSend( QueueHandle_t xQueue, const void * const pvItemToQueue, TickType_t xTicksToWait, const BaseType_t xCopyPosition )
{
	BaseType_t xEntryTimeSet = pdFALSE, xYieldRequired;
	TimeOut_t xTimeOut;
	Queue_t * const pxQueue = xQueue;
	/* 循环判断 */
	for( ;; )
	{
		taskENTER_CRITICAL();
		{
			/* 判断该信号量数值 */
			if( ( pxQueue->uxMessagesWaiting < pxQueue->uxLength ) || ( xCopyPosition == queueOVERWRITE ) )
			{
				traceQUEUE_SEND( pxQueue );
				/* 读取信号量数值  */
				UBaseType_t uxPreviousMessagesWaiting = pxQueue->uxMessagesWaiting;
				xYieldRequired = prvCopyDataToQueue( pxQueue, pvItemToQueue, xCopyPosition );
				/* 判断列表是否为空 */
				if( listLIST_IS_EMPTY( &( pxQueue->xTasksWaitingToReceive ) ) == pdFALSE )
				{
					/* 将任务从挂起表中移除，并将任务加入就绪表 */
					if( xTaskRemoveFromEventList( &( pxQueue->xTasksWaitingToReceive ) ) != pdFALSE )
					{
						/* 设置任务切换标志 */
						queueYIELD_IF_USING_PREEMPTION();
					}
					else
					{
						mtCOVERAGE_TEST_MARKER();
					}
				}
				else if( xYieldRequired != pdFALSE )
				{
					queueYIELD_IF_USING_PREEMPTION();
				}
				else
				{
					mtCOVERAGE_TEST_MARKER();
				}
			}
			taskEXIT_CRITICAL();

			vTaskSuspendAll();
			prvLockQueue( pxQueue );
		} 
	}
}
```

**获取信号量函数分析如下（删减部分代码）：**

```c
BaseType_t xQueueSemaphoreTake( QueueHandle_t xQueue, TickType_t xTicksToWait )
{
	BaseType_t xEntryTimeSet = pdFALSE;
	TimeOut_t xTimeOut;
	Queue_t * const pxQueue = xQueue;
	/* 循环判断 */
	for( ;; )
	{
		taskENTER_CRITICAL();
		{
			/* 读取信号量数值  */
			const UBaseType_t uxSemaphoreCount = pxQueue->uxMessagesWaiting;
			/* 信号量数值大于0 */
			if( uxSemaphoreCount > ( UBaseType_t ) 0 )
			{
				traceQUEUE_RECEIVE( pxQueue );
				/* 信号量数值减一 */
				pxQueue->uxMessagesWaiting = uxSemaphoreCount - ( UBaseType_t ) 1;
				/* 判断列表是否为空 */
				if( listLIST_IS_EMPTY( &( pxQueue->xTasksWaitingToSend ) ) == pdFALSE )
				{
					/* 将挂起表中的第一个任务加入就绪表 */
					if( xTaskRemoveFromEventList( &( pxQueue->xTasksWaitingToSend ) ) != pdFALSE )
					{
						/* 设置任务切换标志 */
						queueYIELD_IF_USING_PREEMPTION();
					}
					else
					{
						mtCOVERAGE_TEST_MARKER();
					}
				}
				else
				{
					mtCOVERAGE_TEST_MARKER();
				}

				taskEXIT_CRITICAL();
				
				return pdPASS;
			}

		}
		taskEXIT_CRITICAL();

		vTaskSuspendAll();
		prvLockQueue( pxQueue );
	} 
}
```

> <font color=red>**未完待续…
实时操作系统系列将持续更新
创作不易希望朋友们点赞，转发，评论，关注。
您的点赞，转发，评论，关注将是我持续更新的动力
作者：李巍
Github：liyinuoman2017
CSDN：liyinuo2017
今日头条：程序猿李巍**
