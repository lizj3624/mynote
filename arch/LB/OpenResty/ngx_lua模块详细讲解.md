ngx_lua模块的原理：

1、每个worker（工作进程）创建一个Lua VM，worker内所有协程共享VM；
2、将Nginx I/O原语封装后注入 Lua VM，允许Lua代码直接访问；
3、每个外部请求都由一个Lua协程处理，协程之间数据隔离；
4、Lua代码调用I/O操作等异步接口时，会挂起当前协程（并保护上下文数据），而不阻塞worker；
5、I/O等异步操作完成时还原相关协程上下文数据，并继续运行；

**ngx_lua 模块提供的指令和API等：**

| 指令名称                                      | 说明                                                         |
| --------------------------------------------- | ------------------------------------------------------------ |
| lua_use_default_type                          | 是否使用default_type指令定义的Content-Type默认值             |
| lua_code_cache                                | *_by_lua_file文件是否cache                                   |
| lua_regex_cache_max_entries                   |                                                              |
| lua_regex_match_limit                         |                                                              |
| lua_package_path                              | 用Lua写的lua外部库路径（.lua文件）                           |
| lua_package_cpath                             | 用C写的lua外部库路径（.so文件）                              |
| init_by_lua                                   | master进程启动时挂载的lua代码                                |
| init_by_lua_file                              |                                                              |
| init_worker_by_lua                            | worker进程启动时挂载的lua代码，常用来执行一些定时器任务      |
| init_worker_by_lua_file                       |                                                              |
| set_by_lua                                    | 设置变量                                                     |
| set_by_lua_file                               |                                                              |
| content_by_lua                                | handler模块                                                  |
| content_by_lua_file                           |                                                              |
| rewrite_by_lua                                |                                                              |
| rewrite_by_lua_file                           |                                                              |
| access_by_lua                                 |                                                              |
| access_by_lua_file                            |                                                              |
| header_filter_by_lua                          | header filter模块                                            |
| header_filter_by_lua_file                     |                                                              |
| body_filter_by_lua                            | body filter模块，ngx.arg[1]代表输入的chunk，ngx.arg[2]代表当前chunk是否为last |
| body_filter_by_lua_file                       |                                                              |
| log_by_lua                                    |                                                              |
| log_by_lua_file                               |                                                              |
| lua_need_request_body                         | 是否读请求体，跟ngx.req.read_body()函数作用类似              |
| lua_shared_dict                               | 创建全局共享的table（多个worker进程共享）                    |
| lua_socket_connect_timeout                    | TCP/unix 域socket对象connect方法的超时时间                   |
| lua_socket_send_timeout                       | TCP/unix 域socket对象send方法的超时时间                      |
| lua_socket_send_lowat                         | 设置cosocket send buffer的low water值                        |
| lua_socket_read_timeout                       | TCP/unix 域socket对象receive方法的超时时间                   |
| lua_socket_buffer_size                        | cosocket读buffer大小                                         |
| lua_socket_pool_size                          | cosocket连接池大小                                           |
| lua_socket_keepalive_timeout                  | cosocket长连接超时时间                                       |
| lua_socket_log_errors                         | 是否打开cosocket错误日志                                     |
| lua_ssl_ciphers                               |                                                              |
| lua_ssl_crl                                   |                                                              |
| lua_ssl_protocols                             |                                                              |
| lua_ssl_trusted_certificate                   |                                                              |
| lua_ssl_verify_depth                          |                                                              |
| lua_http10_buffering                          |                                                              |
| rewrite_by_lua_no_postpone                    |                                                              |
| lua_transform_underscores_in_response_headers |                                                              |
| lua_check_client_abort                        | 是否监视client提前关闭请求的事件，如果打开监视，会调用ngx.on_abort()注册的回调 |
| lua_max_pending_timers                        |                                                              |
| lua_max_running_timers                        |                                                              |

 

| table                         | 说明                                                         |
| ----------------------------- | ------------------------------------------------------------ |
| ngx.arg                       | 指令参数，如跟在content_by_lua_file后面的参数                |
| ngx.var                       | 变量，ngx.var.VARIABLE引用某个变量                           |
| ngx.ctx                       | 请求的lua上下文                                              |
| ngx.header                    | 响应头，ngx.header.HEADER引用某个头                          |
| ngx.status                    | 响应码                                                       |
|                               |                                                              |
| API                           | 说明                                                         |
| ngx.log                       | 输出到error.log                                              |
| print                         | 等价于 ngx.log(ngx.NOTICE, ...)                              |
| ngx.send_headers              | 发送响应头                                                   |
| ngx.headers_sent              | 响应头是否已发送                                             |
| ngx.resp.get_headers          | 获取响应头                                                   |
| ngx.timer.at                  | 注册定时器事件                                               |
| ngx.is_subrequest             | 当前请求是否是子请求                                         |
| ngx.location.capture          | 发布一个子请求                                               |
| ngx.location.capture_multi    | 发布多个子请求                                               |
| ngx.exec                      |                                                              |
| ngx.redirect                  |                                                              |
| ngx.print                     | 输出响应                                                     |
| ngx.say                       | 输出响应，自动添加'\n'                                       |
| ngx.flush                     | 刷新响应                                                     |
| ngx.exit                      | 结束请求                                                     |
| ngx.eof                       |                                                              |
| ngx.sleep                     | 无阻塞的休眠（使用定时器实现）                               |
| ngx.get_phase                 |                                                              |
| ngx.on_abort                  | 注册client断开请求时的回调函数                               |
| ndk.set_var.DIRECTIVE         |                                                              |
| ngx.req.start_time            | 请求的开始时间                                               |
| ngx.req.http_version          | 请求的HTTP版本号                                             |
| ngx.req.raw_header            | 请求头（包括请求行）                                         |
| ngx.req.get_method            | 请求方法                                                     |
| ngx.req.set_method            | 请求方法重载                                                 |
| ngx.req.set_uri               | 请求URL重写                                                  |
| ngx.req.set_uri_args          |                                                              |
| ngx.req.get_uri_args          | 获取请求参数                                                 |
| ngx.req.get_post_args         | 获取请求表单                                                 |
| ngx.req.get_headers           | 获取请求头                                                   |
| ngx.req.set_header            |                                                              |
| ngx.req.clear_header          |                                                              |
| ngx.req.read_body             | 读取请求体                                                   |
| ngx.req.discard_body          | 扔掉请求体                                                   |
| ngx.req.get_body_data         |                                                              |
| ngx.req.get_body_file         |                                                              |
| ngx.req.set_body_data         |                                                              |
| ngx.req.set_body_file         |                                                              |
| ngx.req.init_body             |                                                              |
| ngx.req.append_body           |                                                              |
| ngx.req.finish_body           |                                                              |
| ngx.req.socket                |                                                              |
| ngx.escape_uri                | 字符串的url编码                                              |
| ngx.unescape_uri              | 字符串url解码                                                |
| ngx.encode_args               | 将table编码为一个参数字符串                                  |
| ngx.decode_args               | 将参数字符串编码为一个table                                  |
| ngx.encode_base64             | 字符串的base64编码                                           |
| ngx.decode_base64             | 字符串的base64解码                                           |
| ngx.crc32_short               | 字符串的crs32_short哈希                                      |
| ngx.crc32_long                | 字符串的crs32_long哈希                                       |
| ngx.hmac_sha1                 | 字符串的hmac_sha1哈希                                        |
| ngx.md5                       | 返回16进制MD5                                                |
| ngx.md5_bin                   | 返回2进制MD5                                                 |
| ngx.sha1_bin                  | 返回2进制sha1哈希值                                          |
| ngx.quote_sql_str             | SQL语句转义                                                  |
| ngx.today                     | 返回当前日期                                                 |
| ngx.time                      | 返回UNIX时间戳                                               |
| ngx.now                       | 返回当前时间                                                 |
| ngx.update_time               | 刷新时间后再返回                                             |
| ngx.localtime                 |                                                              |
| ngx.utctime                   |                                                              |
| ngx.cookie_time               | 返回的时间可用于cookie值                                     |
| ngx.http_time                 | 返回的时间可用于HTTP头                                       |
| ngx.parse_http_time           | 解析HTTP头的时间                                             |
| ngx.re.match                  |                                                              |
| ngx.re.find                   |                                                              |
| ngx.re.gmatch                 |                                                              |
| ngx.re.sub                    |                                                              |
| ngx.re.gsub                   |                                                              |
| ngx.shared.DICT               |                                                              |
| ngx.shared.DICT.get           |                                                              |
| ngx.shared.DICT.get_stale     |                                                              |
| ngx.shared.DICT.set           |                                                              |
| ngx.shared.DICT.safe_set      |                                                              |
| ngx.shared.DICT.add           |                                                              |
| ngx.shared.DICT.safe_add      |                                                              |
| ngx.shared.DICT.replace       |                                                              |
| ngx.shared.DICT.delete        |                                                              |
| ngx.shared.DICT.incr          |                                                              |
| ngx.shared.DICT.flush_all     |                                                              |
| ngx.shared.DICT.flush_expired |                                                              |
| ngx.shared.DICT.get_keys      |                                                              |
| ngx.socket.udp                |                                                              |
| udpsock:setpeername           |                                                              |
| udpsock:send                  |                                                              |
| udpsock:receive               |                                                              |
| udpsock:close                 |                                                              |
| udpsock:settimeout            |                                                              |
| ngx.socket.tcp                |                                                              |
| tcpsock:connect               |                                                              |
| tcpsock:sslhandshake          |                                                              |
| tcpsock:send                  |                                                              |
| tcpsock:receive               |                                                              |
| tcpsock:receiveuntil          |                                                              |
| tcpsock:close                 |                                                              |
| tcpsock:settimeout            |                                                              |
| tcpsock:setoption             |                                                              |
| tcpsock:setkeepalive          |                                                              |
| tcpsock:getreusedtimes        |                                                              |
| ngx.socket.connect            |                                                              |
| ngx.thread.spawn              |                                                              |
| ngx.thread.wait               |                                                              |
| ngx.thread.kill               |                                                              |
| coroutine.create              |                                                              |
| coroutine.resume              |                                                              |
| coroutine.yield               |                                                              |
| coroutine.wrap                |                                                              |
| coroutine.running             |                                                              |
| coroutine.status              |                                                              |
| ngx.config.debug              | 编译时是否有 --with-debug选项                                |
| ngx.config.prefix             | 编译时的 --prefix选项                                        |
| ngx.config.nginx_version      | 返回nginx版本号                                              |
| ngx.config.nginx_configure    | 返回编译时 ./configure的命令行选项                           |
| ngx.config.ngx_lua_version    | 返回ngx_lua模块版本号                                        |
| ngx.worker.exiting            | 当前worker进程是否正在关闭（如reload、shutdown期间）         |
| ngx.worker.pid                | 返回当前worker进程的pid                                      |
|                               |                                                              |
|                               |                                                              |
|                               |                                                              |
| 常量                          | 说明                                                         |
| Core constants                | ngx.OK (0) ngx.ERROR (-1) ngx.AGAIN (-2) ngx.DONE (-4) ngx.DECLINED (-5) ngx.nil |
| HTTP method constants         | ngx.HTTP_GET ngx.HTTP_HEAD ngx.HTTP_PUT ngx.HTTP_POST ngx.HTTP_DELETE ngx.HTTP_OPTIONS  ngx.HTTP_MKCOL   ngx.HTTP_COPY    ngx.HTTP_MOVE    ngx.HTTP_PROPFIND  ngx.HTTP_PROPPATCH  ngx.HTTP_LOCK  ngx.HTTP_UNLOCK   ngx.HTTP_PATCH   ngx.HTTP_TRACE |
| HTTP status constants         | ngx.HTTP_OK (200) ngx.HTTP_CREATED (201) ngx.HTTP_SPECIAL_RESPONSE (300) ngx.HTTP_MOVED_PERMANENTLY (301) ngx.HTTP_MOVED_TEMPORARILY (302) ngx.HTTP_SEE_OTHER (303) ngx.HTTP_NOT_MODIFIED (304) ngx.HTTP_BAD_REQUEST (400) ngx.HTTP_UNAUTHORIZED (401) ngx.HTTP_FORBIDDEN (403) ngx.HTTP_NOT_FOUND (404) ngx.HTTP_NOT_ALLOWED (405) ngx.HTTP_GONE (410) ngx.HTTP_INTERNAL_SERVER_ERROR (500) ngx.HTTP_METHOD_NOT_IMPLEMENTED (501) ngx.HTTP_SERVICE_UNAVAILABLE (503) ngx.HTTP_GATEWAY_TIMEOUT (504) |
| Nginx log level constants     | ngx.STDERR ngx.EMERG ngx.ALERT ngx.CRIT ngx.ERR ngx.WARN ngx.NOTICE ngx.INFO ngx.DEBUG |

* [ngx_lua 模块详细讲解（基于openresty）](https://www.cnblogs.com/yanzi-meng/p/9450991.html)