---
layout: post
title: redis sentinel介绍及一个小bug
description: 
category: blogs
---


####redissentinel简介

redis sentinel是一种特殊的redis服务器，但仍然是一种redis服务器，在启动过程中，会将运行代码从redis代码替换为sentinel代码。

sentinel的主要作用是在主从环境下，在主redis宕机后，在从redis中选出新的主redis，保证分布式缓存能正常工作。

#####配置文件

主redis：master
从redis: slave

在sentinel的配置中，通过monitor来配置监听哪一个ip port的redis服务器作为master，mymaster只是一个名字

sentinel monitor mymaster 10.171.91.230 6379 2(quorum)

启动后，sentinel会读取配置文件并且将当前的配置保存在一个结构体中，还会通过向该redis发送info信息获取master的状态，同时获取当前的slave以及其他监听此master的sentinel的信息，保存在结构体中并更新sentinel.conf文件，发现新的slave会增加 known-slave 信息，发现新的监听同一master的sentinel会增加know-sentinel信息。

sentinel known-sentinel mymaster 10.171.149.13 26379 44ae696dbb28f997d6a1993f514cca8d4999fea0
sentinel known-slave mymaster 10.171.91.230 6379

这部分是sentinel自己维护的，但是我们也可以提前设置有哪些slave,要注意的一个问题（也算是bug）是，如果know-slave 的ip port和监听的master ip port 相同的话，sentinel仍然会将master设置成slave（sentinel的设置流程下面会提到），也就是master会变成自己的slave，这是一个很悲惨的事情，master的状态会变成down，因为master自身连不上master（它在连的master就是它自己），如果sentinel们选出了新的master，那么一切会恢复正常，如果没有，= =。当然这是一种很极端的情况，但是在使用的时候也不是没有出现的可能。

#####工作流程

sentinel启动后，加载配置文件，将当前的配置保存在一个结构体sentinelRedisInstance中。

这里面会记录诸如failover_timeout等从配置文件中读取的配置项。还会记录last_ping_time,last_pong_time等等连接的状态。

另外还会通过向该redis发送info信息获取master的状态，同时获取当前的slave以及其他监听此master的sentinel的信息，保存在结构体中，如slave_master_host（当前的主ip）等，并更新sentinel.conf文件，发现新的slave会增加 known-slave 信息，发现新的监听同一master的sentinel会增加know-sentinel信息。

日志如下

21012:X 03 Dec 09:46:30.451 # Sentinel runid is 078aa98768d01727ccc8078ad67a9bfbbd65fb1b

21012:X 03 Dec 09:46:30.451 # +monitor master mymaster 10.171.91.230 6379 quorum 2

当发现有slave或master或sentinel或任何其他的redis下线后，sentinel会判定它为主观（sdown状态）下线

21012:X 03 Dec 09:47:00.480 # +sdown slave 10.171.144.224:6379 10.171.144.224 6379 @ mymaster 10.171.91.230 6379

21012:X 03 Dec 09:47:00.480 # +sdown sentinel 10.171.149.13:26379 10.171.149.13 26379 @ mymaster 10.171.91.230 6379

在一个sentinel认为master主观下线后，它会向其他sentinel确认，大家一致认为master下线之后，master就会进入客观下线（odown）的状态。

21012:X 03 Dec 10:23:16.715 # +odown master mymaster 10.171.91.230 6379 #quorum 2/2

之后sentinel之间会选举领头sentinel，选出领头sentinel后，由领头sentinel负责从当前的slave中选出一个更改为master，将其他slave修改为新master的slave，然后将原来的master记录为know-slave，这样在下一次旧的master上线后就会自动被修改为新master的slave。

领头sentinel的日志

28944:X 03 Dec 10:23:21.278 # +vote-for-leader ae03e9c7965d0182d42ba9472c7c238f99caf40a 6634

28944:X 03 Dec 10:23:21.294 # 10.171.144.224:26379 voted for ae03e9c7965d0182d42ba9472c7c238f99caf40a 6634

28944:X 03 Dec 10:23:21.345 # +elected-leader master mymaster 10.171.91.230 6379

28944:X 03 Dec 10:23:21.345 # +failover-state-select-slave master mymaster 10.171.91.230 6379

28944:X 03 Dec 10:23:21.404 # +selected-slave slave 10.171.144.224:6379 10.171.144.224 6379 @ mymaster 10.171.91.230 6379

28944:X 03 Dec 10:23:21.404 * +failover-state-send-slaveof-noone slave 10.171.144.224:6379 10.171.144.224 6379 @ mymaster 10.171.91.230 6379

28944:X 03 Dec 10:23:21.495 * +failover-state-wait-promotion slave 10.171.144.224:6379 10.171.144.224 6379 @ mymaster 10.171.91.230 6379

28944:X 03 Dec 10:23:22.322 # +promoted-slave slave 10.171.144.224:6379 10.171.144.224 6379 @ mymaster 10.171.91.230 6379

28944:X 03 Dec 10:23:22.322 # +failover-state-reconf-slaves master mymaster 10.171.91.230 6379

28944:X 03 Dec 10:23:22.403 # +failover-end master mymaster 10.171.91.230 6379

28944:X 03 Dec 10:23:22.403 # +switch-master mymaster 10.171.91.230 6379 10.171.144.224 6379

注意其中的promoted-slave和switch-master。

其他sentinel的日志

21012:X 03 Dec 10:22:47.741 * +sentinel sentinel 10.171.91.230:26379 10.171.91.230 26379 @ mymaster 10.171.91.230 6379

21012:X 03 Dec 10:23:15.810 # +new-epoch 6634

21012:X 03 Dec 10:23:15.813 # +vote-for-leader ae03e9c7965d0182d42ba9472c7c238f99caf40a 6634

21012:X 03 Dec 10:23:16.715 # +odown master mymaster 10.171.91.230 6379 #quorum 2/2

21012:X 03 Dec 10:23:16.715 # Next failover delay: I will not start a failover before Thu Dec  3 10:29:16 2015

21012:X 03 Dec 10:23:17.008 # +config-update-from sentinel 10.171.91.230:26379 10.171.91.230 26379 @ mymaster 10.171.91.230 6379

21012:X 03 Dec 10:23:17.008 # +switch-master mymaster 10.171.91.230 6379 10.171.144.224 6379

21012:X 03 Dec 10:23:17.008 * +slave slave 10.171.91.230:6379 10.171.91.230 6379 @ mymaster 10.171.144.224 6379

21012:X 03 Dec 10:23:47.066 # +sdown slave 10.171.91.230:6379 10.171.91.230 6379 @ mymaster 10.171.144.224 6379

#####小bug的问题

关于之前提到的那个小bug，可以很容易的在redis源码里修改，sentinel.c是redis sentinel的实现源码，其中包括sentinelRedisInstance等结构体和其他接口，redis的命令是通过hiredis来实现的，在其中有这样一个函数

<pre class="brush: cpp">

int sentinelSendSlaveOf(sentinelRedisInstance *ri, char *host, int port);

</pre>


这个函数就是redis用来修改master或者slave的函数

<pre class="brush: cpp">

int sentinelSendSlaveOf(sentinelRedisInstance *ri, char *host, int port) {
    char portstr[32];
    int retval;

    ll2string(portstr,sizeof(portstr),port);

    /* If host is NULL we send SLAVEOF NO ONE that will turn the instance
     * into a master. */
    if (host == NULL) {
        host = "NO";
        memcpy(portstr,"ONE",4);
    }

    /* In order to send SLAVEOF in a safe way, we send a transaction performing
     * the following tasks:
     * 1) Reconfigure the instance according to the specified host/port params.
     * 2) Rewrite the configuraiton.
     * 3) Disconnect all clients (but this one sending the commnad) in order
     *    to trigger the ask-master-on-reconnection protocol for connected
     *    clients.
     *
     * Note that we don't check the replies returned by commands, since we
     * will observe instead the effects in the next INFO output. */
    retval = redisAsyncCommand(ri->cc,
        sentinelDiscardReplyCallback, NULL, "MULTI");
    if (retval == REDIS_ERR) return retval;
    ri->pending_commands++;

    retval = redisAsyncCommand(ri->cc,
        sentinelDiscardReplyCallback, NULL, "SLAVEOF %s %s", host, portstr);
    if (retval == REDIS_ERR) return retval;
    ri->pending_commands++;

    retval = redisAsyncCommand(ri->cc,
        sentinelDiscardReplyCallback, NULL, "CONFIG REWRITE");
    if (retval == REDIS_ERR) return retval;
    ri->pending_commands++;

    /* CLIENT KILL TYPE &lt;type> is only supported starting from Redis 2.8.12,
     * however sending it to an instance not understanding this command is not
     * an issue because CLIENT is variadic command, so Redis will not
     * recognized as a syntax error, and the transaction will not fail (but
     * only the unsupported command will fail). */
    retval = redisAsyncCommand(ri->cc,
        sentinelDiscardReplyCallback, NULL, "CLIENT KILL TYPE normal");
    if (retval == REDIS_ERR) return retval;
    ri->pending_commands++;

    retval = redisAsyncCommand(ri->cc,
        sentinelDiscardReplyCallback, NULL, "EXEC");
    if (retval == REDIS_ERR) return retval;
    ri->pending_commands++;

    return REDIS_OK;
}

</pre>

ri是一个sentinelRedisInstance *，也就是当前要操作的sentinel实例，host就是要设置的masterip，port就是当前要设置的masterport，如果host为空，那么就设置为master，因此我们只需要在里面加上一个判断，如果ri中的slave_master_host与host相同，就不进行设置，就可以避免这个问题了。

然而这样改是并不能行的，因为sentinelSendSlaveOf这个函数不仅用于设置slave的master，也用来将slave提升为master，如果传入的masterip为空，则将当前的ri设置为master

所以需要在sentinelRefreshInstanceInfo这个函数中，找到如下这段内容，这个函数是用来周期性更新redis信息的，而这一段是用来处理一个本来应该是slave的redis现在是master的情况。

<pre class="brush: cpp">

 /* A slave turned into a master. We want to force our view and
             * reconfigure as slave. Wait some time after the change before
             * going forward, to receive new configs if any. */
            mstime_t wait_time = SENTINEL_PUBLISH_PERIOD*4;

            if (!(ri->flags & SRI_PROMOTED) &&
                 sentinelMasterLooksSane(ri->master) &&
                 sentinelRedisInstanceNoDownFor(ri,wait_time) &&
                 mstime() - ri->role_reported_time > wait_time)
            {
                int retval = sentinelSendSlaveOf(ri,
                        ri->master->addr->ip,
                        ri->master->addr->port);
                if (retval == REDIS_OK)
                    sentinelEvent(REDIS_NOTICE,"+convert-to-slave",ri,"%@");
            }

</pre>

需要做个判读，其实应该用 SRI_PROMOTED这个状态来修改。

<pre class="brush: cpp">

/* A slave turned into a master. We want to force our view and
             * reconfigure as slave. Wait some time after the change before
             * going forward, to receive new configs if any. */
            mstime_t wait_time = SENTINEL_PUBLISH_PERIOD*4;

            if (!(ri->flags & SRI_PROMOTED) &&
                 sentinelMasterLooksSane(ri->master) &&
                 sentinelRedisInstanceNoDownFor(ri,wait_time) &&
                 mstime() - ri->role_reported_time > wait_time)
            {
				sentinelEvent(REDIS_WARNING,"ri is %s",ri,"%@%s",ri->addr->ip);
				//&& ri->master->addr->port == ri->addr->port
				sentinelEvent(REDIS_WARNING,"master is%s",ri->master,"%@%s", ri->master->addr->ip);
				if( strcmp(ri->master->addr->ip,ri->addr->ip) == 0 && ri->master->addr->port == ri->addr->port)
				{
					sentinelEvent(REDIS_NOTICE,"slave is alredy master",ri,"%@");
					ri->master->config_epoch = ri->master->failover_epoch;
					ri->master->failover_state = SENTINEL_FAILOVER_STATE_RECONF_SLAVES;
					ri->master->failover_state_change_time = mstime();
					sentinelFlushConfig();
					sentinelEvent(REDIS_WARNING,"+promoted-slave",ri,"%@");
					sentinelResetMaster(ri->master,SENTINEL_RESET_NO_SENTINELS);
					//sentinelForceHelloUpdateForMaster(ri->master);
				}
				else
				{
					int retval = sentinelSendSlaveOf(ri,
                        ri->master->addr->ip,
                        ri->master->addr->port);
					if (retval == REDIS_OK)
						sentinelEvent(REDIS_NOTICE,"+convert-to-slave",ri,"%@");
				}
            }
</pre>
