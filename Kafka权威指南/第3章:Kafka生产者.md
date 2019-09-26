
# 第3章:Kafka生产者

1. 创建Kafka生产者
   1. bootstrap.servers
      1. broker地址清单，格式为host:port，不需要配置所有broker，但是建议最少配置2个
   2. key.serializer
      1. kafka键序列化器，必须指定
      2. 需要继承org.apache.kafka.common.serialization.Serializer类
      3. 默认提供了ByteArraySerializer、StringSerializer、IntegerSerializer
   3. value.serializer
      1. kafka值序列化器,同key.serializer
   4. 发送消息的方式
      1. 发送并忘记(fire-and-forget)
         1. 直接发送给服务器，不关心是否到达，失败会重试，但可能丢失数据
      2. 同步发送
         1. 使用send方法发送并返回一个future对象，通过get方法获取发送结果
      3. 异步发送
         1. 使用send方法发送并指定一个回调函数，在服务器返回响应时调用该函数
   5. broker返回的错误
      1. 可重试错误，生产者重试失败后会返回错误
      2. 不可重试错误，生产者直接返回错误
   6. 生产者的配置
      1. acks
         1. 指定必须有多少个分区副本收到消息生产者才认为写入成功
         2. acks=0 不等待服务器响应，最快，不稳定
         3. acks=1 等待leader返回响应认为成功
         4. acks=all 等待所有leader返回响应认为成功，最慢，最安全
      2. buffer.memory
         1. 生产者发送数据时的缓冲区大小
         2. max.block.ms配置会在缓冲区空间不足且send发送阻塞时允许最大的阻塞时间
      3. compression.type
         1. 发送数据的压缩方式，默认不压缩
         2. 往往会造成kafka性能瓶颈
      4. retries
         1. 当生产者收到可重试错误时，重试发送数据的次数
         2. 默认间隔100ms，通过retry.backoff.ms修改间隔
      5. batch.size
         1. 当多个消息被发送到同一个分区中时，socket每一批数据的字节大小
         2. 批次没有满时也会发送，不影响发送速度
         3. 设置太大浪费内存，设置太小有性能开销
      6. linger.ms
         1. 等待批次填满的最大时间，默认发送线程空闲就会立即发送
         2. 设置为大于0的值可以增大吞吐量
      7. max.in.flight.requests.per.connection
         1. 指定生产者在收到服务器响应之前可以发送多少个消息
         2. 设置越大内存开销越大，吞吐量越高
         3. 设置为1表示消息顺序发送
      8. request.timeout.ms
         1. 生产者在发送消息后等待服务器响应的超时时间
      9. metadata.fetch.timeout.ms
         1. 生产者获取原数据时的等待超时时间
      10. timeout.ms
          1. 生产者等待同步副本响应的等待超时时间
      11. max.block.ms
          1. 生产者发送数据或这获取元数据时最大阻塞时间
      12. max.request.size
          1. 单个消息最大大小，需要小于服务端的message.max.bytes配置
      13. receive.buffer.size、send.buffer.size
          1. socket 接收和发送数据包的缓存大小
   7. 顺序保证
      1. 同一个分区内可以保证有序
   8. 分区
      1. 相同key的数据会写入到同一个分区,key可以为空
      2. 如果key为空就采用轮询的方式发送到各个分区中
      3. 如果key不为空采用散列分配分区，分区数量修改以后会改变散列规则
