---
title: 'top命令定位CPU占用率高的问题'
date: 2019-12-16
categories: Linux
---
1. 使用`top`命令定位CPU占用率高的进程
    ```sh
    [kfz@web-srv ~]$ top
    top - 14:43:57 up 867 days, 23:02, 17 users,  load average: 1.00, 1.03, 1.12
    Tasks: 337 total,   1 running, 333 sleeping,   3 stopped,   0 zombie
    Cpu(s):  9.1%us,  1.1%sy,  0.0%ni, 89.2%id,  0.0%wa,  0.3%hi,  0.3%si,  0.0%st
    Mem:  30810956k total, 30057420k used,   753536k free,   145708k buffers
    Swap:  4095996k total,   887216k used,  3208780k free, 11503860k cached

      PID USER      PR  NI  VIRT  RES  SHR S %CPU %MEM    TIME+  COMMAND                                                                                             
    29151 root      20   0 13.2g 909m 6312 S 74.0  3.0   2973:19 java -jar yq_mix-2.18.2.jar --sp #这个进程的CPU占用率最高74%
    12581 root      20   0 13.6g 647m 6348 S  5.3  2.2 852:41.47 java -jar yq_wx-2.18.2.jar --spr
       1 root      20   0 19356  388  216 S  0.0  0.0   4:27.22 /sbin/init                                                                                           
       2 root      20   0     0    0    0 S  0.0  0.0   0:00.00 [kthreadd]                                                                                           
       3 root      RT   0     0    0    0 S  0.0  0.0  23927:30 [migration/0]                                                                                        
       4 root      20   0     0    0    0 S  0.0  0.0  5125660h [ksoftirqd/0]                                                                                        
       5 root      RT   0     0    0    0 S  0.0  0.0   0:00.00 [stopper/0]                                                                                          
       6 root      RT   0     0    0    0 S  0.0  0.0   1839:18 [watchdog/0]                                                                                         
       7 root      RT   0     0    0    0 S  0.0  0.0   0:37.21 [migration/1]                                                                                        
       8 root      RT   0     0    0    0 S  0.0  0.0   0:00.00 [stopper/1]                                                                                          
       9 root      20   0     0    0    0 S  0.0  0.0   2420:52 [ksoftirqd/1]                                                                                        
      10 root      RT   0     0    0    0 S  0.0  0.0   2:16.68 [watchdog/1]                                                                                         
      11 root      RT   0     0    0    0 S  0.0  0.0   0:29.29 [migration/2]                                                                                        
      12 root      RT   0     0    0    0 S  0.0  0.0   0:00.00 [stopper/2]                                                                                          
      13 root      20   0     0    0    0 S  0.0  0.0   2451:57 [ksoftirqd/2]                                                                                        
      14 root      RT   0     0    0    0 S  0.0  0.0   2:16.32 [watchdog/2]                                                                                         
      15 root      RT   0     0    0    0 S  0.0  0.0   0:26.05 [migration/3]                                                                                        
      16 root      RT   0     0    0    0 S  0.0  0.0   0:00.00 [stopper/3]                                                                                          
      17 root      20   0     0    0    0 S  0.0  0.0   2434:14 [ksoftirqd/3]                                                                                        
      18 root      RT   0     0    0    0 S  0.0  0.0   2:12.57 [watchdog/3]                                                                                         
      19 root      RT   0     0    0    0 S  0.0  0.0   0:30.03 [migration/4]                                                                                        
      20 root      RT   0     0    0    0 S  0.0  0.0   0:00.00 [stopper/4]                                                                                          
      21 root      20   0     0    0    0 S  0.0  0.0   2480:04 [ksoftirqd/4]
   ```
2. 使用`top -Hp process_id`定位CPU占用率高的线程
  ```sh
  [kfz@web-srv ~]$ top -Hp 29151
  top - 14:44:29 up 867 days, 23:02, 17 users,  load average: 1.00, 1.03, 1.11
  Tasks: 142 total,   1 running, 141 sleeping,   0 stopped,   0 zombie
  Cpu(s):  9.8%us,  1.4%sy,  0.0%ni, 87.4%id,  0.0%wa,  0.6%hi,  0.7%si,  0.1%st
  Mem:  30810956k total, 30022148k used,   788808k free,   145708k buffers
  Swap:  4095996k total,   887216k used,  3208780k free, 11505108k cached

    PID USER      PR  NI  VIRT  RES  SHR S %CPU %MEM    TIME+  COMMAND                                                                                             
  12120 root      20   0 13.2g 879m 6312 R 54.2  2.9 239:57.85 java #此线程CPU占用率最高54.2%
  30010 root      20   0 13.2g 879m 6312 S  1.0  2.9  19:20.48 java                                                                                                 
  29153 root      20   0 13.2g 879m 6312 S  0.3  2.9  14:26.89 java                                                                                                 
  29157 root      20   0 13.2g 879m 6312 S  0.3  2.9  14:05.47 java                                                                                                 
  29158 root      20   0 13.2g 879m 6312 S  0.3  2.9  14:16.49 java                                                                                                 
  29159 root      20   0 13.2g 879m 6312 S  0.3  2.9  14:15.03 java                                                                                                 
  29160 root      20   0 13.2g 879m 6312 S  0.3  2.9  14:22.51 java                                                                                                 
  29151 root      20   0 13.2g 879m 6312 S  0.0  2.9   0:00.00 java                                                                                                 
  29152 root      20   0 13.2g 879m 6312 S  0.0  2.9   0:20.70 java                                                                                                 
  29154 root      20   0 13.2g 879m 6312 S  0.0  2.9  14:17.66 java                                                                                                 
  29155 root      20   0 13.2g 879m 6312 S  0.0  2.9  14:15.75 java                                                                                                 
  29156 root      20   0 13.2g 879m 6312 S  0.0  2.9  14:08.52 java                                                                                                 
  29161 root      20   0 13.2g 879m 6312 S  0.0  2.9  23:08.82 java                                                                                                 
  29162 root      20   0 13.2g 879m 6312 S  0.0  2.9   0:03.02 java                                                                                                 
  29163 root      20   0 13.2g 879m 6312 S  0.0  2.9   0:02.02 java                                                                                                 
  29164 root      20   0 13.2g 879m 6312 S  0.0  2.9   0:00.00 java                                                                                                 
  29165 root      20   0 13.2g 879m 6312 S  0.0  2.9   1:09.15 java                                                                                                 
  29166 root      20   0 13.2g 879m 6312 S  0.0  2.9   1:10.49 java                                                                                                 
  29167 root      20   0 13.2g 879m 6312 S  0.0  2.9   1:08.28 java                                                                                                 
  29168 root      20   0 13.2g 879m 6312 S  0.0  2.9   0:17.32 java                                                                                                 
  29169 root      20   0 13.2g 879m 6312 S  0.0  2.9   1:47.39 java                                                                                                 
  29170 root      20   0 13.2g 879m 6312 S  0.0  2.9   6:29.87 java                                                                                                 
  29179 root      20   0 13.2g 879m 6312 S  0.0  2.9   0:07.48 java                                                                                                 
  29180 root      20   0 13.2g 879m 6312 S  0.0  2.9   0:00.14 java                                                                                                 
  29181 root      20   0 13.2g 879m 6312 S  0.0  2.9   3:38.87 java                                                                                                 
  29182 root      20   0 13.2g 879m 6312 S  0.0  2.9   0:56.56 java                                                                                                 
  29183 root      20   0 13.2g 879m 6312 S  0.0  2.9   0:58.37 java                                                                                                 
  29184 root      20   0 13.2g 879m 6312 S  0.0  2.9   0:57.26 java                                                                                                 
  29185 root      20   0 13.2g 879m 6312 S  0.0  2.9   0:36.28 java                                                                                                 
  29186 root      20   0 13.2g 879m 6312 S  0.0  2.9   0:00.00 java                                                                                                 
  29187 root      20   0 13.2g 879m 6312 S  0.0  2.9   0:46.03 java                                                                                                 
  29188 root      20   0 13.2g 879m 6312 S  0.0  2.9   0:47.67 java                                                                                                 
  29189 root      20   0 13.2g 879m 6312 S  0.0  2.9   0:02.90 java
  ```
3. 将线程id转换为16进制：`printf %x thread_id`
  ```sh
  [kfz@web-srv ~]$ printf %x 12120
  2f58
  ```
4. 定位代码位置：`sudo -u root jstack 进程号 | grep -A 200 线程的16进制ID号`
  ```sh
  [kfz@web-srv ~]$ sudo -u root /usr/java/jdk1.8.0_11/bin/jstack 29151 | grep -A 200 2f58
  [sudo] password for kfz: 
  "TimerScheduler-17" #301 prio=5 os_prio=0 tid=0x00007f4d9800d000 nid=0x2f58 waiting on condition [0x00007f4c9f9bf000]
    java.lang.Thread.State: WAITING (parking)
    at sun.misc.Unsafe.park(Native Method)
    - parking to wait for  <0x000000072413f8f0> (a java.util.concurrent.CountDownLatch$Sync)
    at java.util.concurrent.locks.LockSupport.park(LockSupport.java:175)
    at java.util.concurrent.locks.AbstractQueuedSynchronizer.parkAndCheckInterrupt(AbstractQueuedSynchronizer.java:836)
    at java.util.concurrent.locks.AbstractQueuedSynchronizer.doAcquireSharedInterruptibly(AbstractQueuedSynchronizer.java:997)
    at java.util.concurrent.locks.AbstractQueuedSynchronizer.acquireSharedInterruptibly(AbstractQueuedSynchronizer.java:1304)
    at java.util.concurrent.CountDownLatch.await(CountDownLatch.java:231)
    at rx.internal.operators.BlockingOperatorToFuture$2.get(BlockingOperatorToFuture.java:101)
    at com.netflix.hystrix.HystrixCommand$4.get(HystrixCommand.java:423)
    at com.netflix.hystrix.HystrixCommand.execute(HystrixCommand.java:344)
    at feign.hystrix.HystrixInvocationHandler.invoke(HystrixInvocationHandler.java:159)
    at com.sun.proxy.$Proxy243.search(Unknown Source)
    at trs.cloud.search.api.driver.SearchPrepareStatement.executeQuery(SearchPrepareStatement.java:85)
    at com.alibaba.druid.filter.FilterChainImpl.preparedStatement_executeQuery(FilterChainImpl.java:3188)
    at com.alibaba.druid.filter.FilterEventAdapter.preparedStatement_executeQuery(FilterEventAdapter.java:465)
    at com.alibaba.druid.filter.FilterChainImpl.preparedStatement_executeQuery(FilterChainImpl.java:3185)
    at com.alibaba.druid.filter.FilterEventAdapter.preparedStatement_executeQuery(FilterEventAdapter.java:465)
    at com.alibaba.druid.filter.FilterChainImpl.preparedStatement_executeQuery(FilterChainImpl.java:3185)
    at com.alibaba.druid.proxy.jdbc.PreparedStatementProxyImpl.executeQuery(PreparedStatementProxyImpl.java:181)
    at com.alibaba.druid.pool.DruidPooledPreparedStatement.executeQuery(DruidPooledPreparedStatement.java:227)
    at trs.cloud.yq.service.MonitorMixAlertService.selectMsgInThreeSituation(MonitorMixAlertService.java:142) #在此定位到异常的代码位置
    at trs.cloud.yq.service.MonitorMixAlertService.getMsgByCkm(MonitorMixAlertService.java:105)
    at trs.cloud.yq.timer.SchedulerManager.lambda$getMsgByCkm$0(SchedulerManager.java:84)
    at trs.cloud.yq.timer.SchedulerManager$$Lambda$894/203401172.run(Unknown Source)
    at trs.cloud.yq.timer.TimerScheduler.lambda$schedule$0(TimerScheduler.java:83)
    at trs.cloud.yq.timer.TimerScheduler$$Lambda$895/890733699.run(Unknown Source)
    at java.lang.Thread.run(Thread.java:745)
    at org.springframework.scheduling.support.DelegatingErrorHandlingRunnable.run(DelegatingErrorHandlingRunnable.java:54)
    at org.springframework.scheduling.concurrent.ReschedulingRunnable.run(ReschedulingRunnable.java:93)
    at java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:511)
    at java.util.concurrent.FutureTask.run(FutureTask.java:266)
    at java.util.concurrent.ScheduledThreadPoolExecutor$ScheduledFutureTask.access$201(ScheduledThreadPoolExecutor.java:180)
    at java.util.concurrent.ScheduledThreadPoolExecutor$ScheduledFutureTask.run(ScheduledThreadPoolExecutor.java:293)
    at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1142)
    at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:617)
    at java.lang.Thread.run(Thread.java:745)
    ```