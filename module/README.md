# HTTP
HTTPModule 这是一个收HTTP请求的模块
实现了IModule接口
可以使用NewHTTPModule来实例化一个，需要的参数，可以自己写，也可以读配置
如果想对它进行扩展，可以用他组合一个新的类，然后自己写实例化的方法，可以借鉴NewHTTPModule
希望可以处理更多的请求分类，可以在重载完Init方法后，使用	mod.httpServer.Handler.HandleFunc 添加你自己需要的请求
因为我考虑的是这个模块一个实例化就处理一组消息，所有从这个地方进来的都走统一的逻辑
当然 你也可以按自己的实际需要分出来。
可以做到不同的请求分支有不同的消息格式。
可以自定义基础消息结构
# WebSocket
WebSocketModule是一个收发WebSocket消息的管理器
每一个用户连接上来后，都会开一个新的协程进行操作。
你可以把这个连接句柄引用到你自己的用户对象上面，这样就可以在别的协程上主动的对这个用户进行发送消息
每一个连接如果在一段时间里没有收到这个用户的消息，就会主动的断开连接
每一个连接对象event.WebSocketModel都封装在这里面，里面还可以设置一个断开连接时执行的回调，可以用来做一下当连接断开时要做的事；
比如断开连接进就对用户数据进行一次保存啊或是写到缓存啊或是做卸载的准备等等

# LOGIC
LogicModule 模块，是一个用户处理业务逻辑的管理器
我们可以把业务逻辑放到指定KEYID的协程上运行。
可以用这个方法保证一些逻辑是单协程执行，减少锁的使用。
# SQL
SqlDataModule 模块，是一个处理数据保存的协程管理器
我们可以把指定KEYID分组的数据库操作放到同一个协程上运行。保证数据保存的顺序。
同时也做到了业务逻辑与DB操作分开进行。
# Memory
这个模块用来管理内存数据，什么时候可以释放的管理器
比如，一个用户离线后，一定时间了后，就可以卸载这个用户的数据

MemoryModule.AddListenMsg
是用来添加数据到管理器，也是用来重置时间的方法
数据继承IMemoryModel接口，实现里面的所有方法
其中GetKey用来表示数据的唯一标识
RunAutoEvents，表示添加进管理器时运行的方法，如果之前已在管理器中还没有被去除就不会运行它
UnloadRun，设置的时间到了之后运行的方法，如果你确认这个数据是要被卸载的话，就返回true
DoneRun,当服务器要关闭的时候，会运行的方法。可以用于关闭在RunAutoEvents里启动的协程
以上三个方法都在同一个协程运行。