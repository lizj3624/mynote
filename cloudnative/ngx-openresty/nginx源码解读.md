一、HTTP请求处理

ngx_event_accept中ls-handler(c)===>(ngx_http_init_connection)

ngx_http_init_connection函数
1）根据IP地址查找addr_conf配置
2) 设置读事件的回调:rev->handler = ngx_http_wait_request_handler;
3）h2时初始化ngx_http_v2_init
4) https时初始化ngx_http_ssl_handshake===>>调用ngx_http_ssl_handshake_handler===>>回调ngx_http_ssl_servername查找对应server块
5) 处理proxy_protocol
6) 处理读事件回调rev-handler(rev)===》ngx_http_wait_request_handler
7) 加入读事件

1、ngx_http_wait_request_handler(ngx_event_t *rev) 开始解析请求数据，这个函数时有读操作时，epoll事件驱动回调函数，connection的read回调函数
   ngx_http_init_connection(ngx_connection_t *c)初始化connection，设置回调函数
       rev = c->read;
       rev->handler = ngx_http_wait_request_handler;
   函数处理过程
   1）调用connection的recv回调函数接受数据
      c->recv(c, b->last, size)  ===> socket recv函数
   2）如果有proxy-protocol时，处理proxy-protocol协议
   3）创建request数据结构 ngx_http_create_request(ngx_connection_t *c)
   4）重新设置c->read的回调函数
      rev->handler = ngx_http_process_request_line;
      调用ngx_http_process_request_line解析buff的数据

2、ngx_http_process_request_line 逐行解析buff中的请求数据
   1）ngx_http_read_request_header获取所有的header数据
   2）ngx_http_parse_request_line解析请求行数据  "GET / HTTP1.1"
   3）解析请求行成功后，如果http_version小于1.0时，ngx_http_process_request
   3）再次设置rev->handler = ngx_http_process_request_headers，解析请求头的函数
      调用一次ngx_http_process_request_headers处理buff数据
   4）ngx_http_run_posted_requests(c)	  
      调用request的r->write_event_handler(r)  ==ngx_http_core_run_phases
	  
3、ngx_http_core_run_phases 请求头解析完成后，对请求开始多个阶段处理
   ngx_http_block解析"HTTP"，添加各阶段的checker
   ngx_http_init_phase_handlers
   
4、ngx_http_process_request_headers  
   1）逐行解析http头ngx_http_parse_header_line
   2）ngx_http_process_request_header 处理http头
   3）ngx_http_process_request(ngx_http_request_t *r)
      c->read->handler = ngx_http_request_handler;
      c->write->handler = ngx_http_request_handler;
      r->read_event_handler = ngx_http_block_reading;   
   4）ngx_http_handler
      r->write_event_handler = ngx_http_core_run_phases; 处理http请求的几个阶段
      ngx_http_core_run_phases(r);   
	  
5、ngx_http_core_content_phase	  
       if (r->content_handler) {
           r->write_event_handler = ngx_http_request_empty_handler;
           ngx_http_finalize_request(r, r->content_handler(r)); 执行content_handler的回调函数，然后再执行ngx_http_finalize_request
           return NGX_OK;
        }
		
		ngx_http_update_location_config更content_handler = clcf->handler(ngx_http_proxy_handler)
	    每个upstream_handler都设置clcf->handler = 相应的处理函数，在这个阶段开始选择upstream

6、ngx_http_proxy_handler函数开始处理proxy_pass		
   1）ngx_http_upstream_create(r) 创建upstream数据结构
   2）设置各种回调函数
      u->create_request = ngx_http_proxy_create_request;
      u->reinit_request = ngx_http_proxy_reinit_request;
      u->process_header = ngx_http_proxy_process_status_line;
      u->abort_request = ngx_http_proxy_abort_request;
      u->finalize_request = ngx_http_proxy_finalize_request;
   3）rc = ngx_http_read_client_request_body(r, ngx_http_upstream_init); 
      先调用ngx_http_upstream_init，在调用函数读取client端的body数据
      ngx_http_request_body_filter body过滤的hook

   4）在如下函数设置clcf->handler = ngx_http_proxy_handler;
      ngx_http_proxy_pass =====>> ngx_http_proxy_commands "proxy_pass"
      ngx_http_proxy_merge_loc_conf ===>>ngx_http_proxy_module_ctx->merge_loc_conf
         
   
7、ngx_http_upstream_init_request
   相关处理，支持HTTPS访问
   ngx_http_upstream_connect

8、ngx_http_upstream_connect(ngx_http_request_t *r, ngx_http_upstream_t *u)
   1）ngx_event_connect_peer连接后端一个实例
   2）设置各种回调函数 
      c->write->handler = ngx_http_upstream_handler;
      c->read->handler = ngx_http_upstream_handler;

      u->write_event_handler = ngx_http_upstream_send_request_handler;
      u->read_event_handler = ngx_http_upstream_process_header;   
   3）ngx_http_upstream_send_request(r, u, 1);
      ngx_http_upstream_process_header
      ngx_http_upstream_send_response	  

二、NGX_SSL_ASYNC异步
   1、ngx_connection_s的结构体新增
      异步事件 ngx_event_t  *async;
      异步事件fd ngx_socket_t  async_fd;
      异步事件标记 ngx_flag_t async_enable;
      异步事件fd的个数 unsigned num_async_fds:8; 加入epoll时需要这个个数的fds都加入
      ngx_close_listening_sockets 关闭listening是需要将connection上绑定的async_fds全部从epoll删除
      ngx_get_connection 从内存池获取connection时 需要初始化connection上有关async的数据结构
      ngx_close_connection 关闭时，清楚connection上的async的数据结构

   2、epoll
      ngx_event_s结构体新增
      异步标记  unsigned         async:1;
      异步callback ngx_event_handler_pt  saved_handler;

      ngx_event_actions_t 新增
      将connection中async_fds加入epoll 
          ngx_int_t  (*add_async_conn)(ngx_connection_t *c);
          ngx_epoll_add_async_connection
      将connection中async_fds删除epoll 
          ngx_int_t  (*del_async_conn)(ngx_connection_t *c, ngx_uint_t flags);
          ngx_epoll_del_async_connection
      在epoll_wait地方，ngx_epoll_process_events增加对async事件的处理

   3、openssl
      在ngx_http_ssl_srv_conf_t结构体新增
      配置开关：ngx_flag_t  async_enable; “ssl_async on”

      ngx_ssl_handshake
          在ssl握手后返回状态部分，对异步状态作分析
      新增如下异步回调函数：
      static void ngx_ssl_handshake_async_handler(ngx_event_t * aev);
      static void ngx_ssl_read_async_handler(ngx_event_t * aev);
      static void ngx_ssl_write_async_handler(ngx_event_t * aev);
      static void ngx_ssl_shutdown_async_handler(ngx_event_t *aev);

三、Upstream部分
1、ngx_http_upstream_next()
    ngx_http_upstream_connect()
    ngx_http_upstream_ssl_init_connection()
    ngx_http_upstream_ssl_handshake_handler()
    ngx_http_upstream_send_request()
    ngx_http_upstream_send_request_handler()
    ngx_http_upstream_process_header()
    ngx_http_upstream_test_next()    

2、ngx_http_upstream_ssl_init_connection      
     ngx_http_upstream_set_round_robin_peer_session(ngx_http_upstream_empty_set_session)
        ngx_ssl_set_session 

3、upstream peer
   ngx_http_upstream_init_request()或者ngx_http_upstream_resolve_handler()
       ngx_http_upstream_create_round_robin_peer
   ngx_http_upstream_init_round_robin_peer


四、Nginx配置解析
1、NGX_CORE_MODULE
   ngx_core_module==>ngx_regex, ngx_thread_pool
   ngx_enevt_module
   ngx_http_module
   ngx_mail_module
   ngx_conf_module
   ngx_stream_module
   ngx_log_module     

   typedef struct {
       ngx_str_t             name;
       void               *(*create_conf)(ngx_cycle_t *cycle);
       char               *(*init_conf)(ngx_cycle_t *cycle, void *conf);
   } ngx_core_module_t;

2、NGX_EVENT_MODULE
   ngx_devpoll_module
   ngx_epoll_module
   ngx_eventport_module
   ngx_iocp_module
   ngx_kqueue_module
   ngx_poll_module
   ngx_select_module
   ngx_win32_select_module
   ngx_event

    typedef struct {
        ngx_str_t *name;

        void  *(*create_conf)(ngx_cycle_t *cycle);
        char  *(*init_conf)(ngx_cycle_t *cycle, void *conf);

        ngx_event_actions_t     actions;
    } ngx_event_module_t;   

3、NGX_HTTP_MODULE
   这中类型模块比较多，列举常用几个主要
   ngx_http_core
   ngx_http_request
   ngx_http_upstream*...
   ngx_http_v2
   ngx_http_proxy
   ngx_http_ssl
   ngx_http_lua
   ngx_http_upstream_check

   typedef struct {
       ngx_int_t   (*preconfiguration)(ngx_conf_t *cf);
       ngx_int_t   (*postconfiguration)(ngx_conf_t *cf);

       void       *(*create_main_conf)(ngx_conf_t *cf);
       char       *(*init_main_conf)(ngx_conf_t *cf, void *conf);

       void       *(*create_srv_conf)(ngx_conf_t *cf);
       char       *(*merge_srv_conf)(ngx_conf_t *cf, void *prev, void *conf);

       void       *(*create_loc_conf)(ngx_conf_t *cf);
       char       *(*merge_loc_conf)(ngx_conf_t *cf, void *prev, void *conf);
   } ngx_http_module_t;

4、NGX_STREAM_MODULE
   这种类型的模块也比较多，列举常用一部分
   ngx_stream_core
   ngx_stream_access
   ngx_stream_proxy
   ngx_stream_ssl  
   ngx_stream_lua
   ngx_stream_upstream_check

   typedef struct {
       ngx_int_t                    (*preconfiguration)(ngx_conf_t *cf);
       ngx_int_t                    (*postconfiguration)(ngx_conf_t *cf);

       void                        *(*create_main_conf)(ngx_conf_t *cf);
       char                        *(*init_main_conf)(ngx_conf_t *cf, void *conf);

       void                        *(*create_srv_conf)(ngx_conf_t *cf);
       char                        *(*merge_srv_conf)(ngx_conf_t *cf, void *prev,
                                                   void *conf);
   } ngx_stream_module_t;
      

5、如果要实现一个模块，就必须定义如下三个结构体
   1）ctx结构体，也就是NGX_CORE_MODULE，NGX_EVENT_MODULE、NGX_HTTP_MODULE、NGX_STREAM_MODULE等其中一个
   2）command结构体，模块所支持的指令
      struct ngx_command_s {
          ngx_str_t             name;
          ngx_uint_t            type;
          char               *(*set)(ngx_conf_t *cf, ngx_command_t *cmd, void *conf);
          ngx_uint_t            conf;
          ngx_uint_t            offset;
          void                 *post;
      }; 

  3）定义实现ngx_module_s结构，
    其中ctx就是前面说的ctx
    commands就是前面说的command
    还有一些callback
    struct ngx_module_s {
        ngx_uint_t            ctx_index;
        ngx_uint_t            index;
    
        char                 *name;
    
        ngx_uint_t            spare0;
        ngx_uint_t            spare1;
    
        ngx_uint_t            version;
        const char           *signature;
    
        void                 *ctx;
        ngx_command_t        *commands;
        ngx_uint_t            type;
    
        ngx_int_t           (*init_master)(ngx_log_t *log);
    
        ngx_int_t           (*init_module)(ngx_cycle_t *cycle);
    
        ngx_int_t           (*init_process)(ngx_cycle_t *cycle);
        ngx_int_t           (*init_thread)(ngx_cycle_t *cycle);
        void                (*exit_thread)(ngx_cycle_t *cycle);
        void                (*exit_process)(ngx_cycle_t *cycle);
    
        void                (*exit_master)(ngx_cycle_t *cycle);
    
        uintptr_t             spare_hook0;
        uintptr_t             spare_hook1;
        uintptr_t             spare_hook2;
        uintptr_t             spare_hook3;
        uintptr_t             spare_hook4;
        uintptr_t             spare_hook5;
        uintptr_t             spare_hook6;
        uintptr_t             spare_hook7;
    };

6、Nginx配置文件解析不错的文章
  https://segmentfault.com/a/1190000016913713
  https://segmentfault.com/a/1190000016922188    

五、七层负载健康检查模块-ngx_http_upstream_check_module
   定义模块：
   ngx_module_t  ngx_http_upstream_check_module = {
       NGX_MODULE_V1,
       &ngx_http_upstream_check_module_ctx,   /* module context */
       ngx_http_upstream_check_commands,      /* module directives */
       NGX_HTTP_MODULE,                       /* module type */
       NULL,                                  /* init master */
       NULL,                                  /* init module */
       ngx_http_upstream_check_init_process,  /* init process */
       NULL,                                  /* init thread */
       NULL,                                  /* exit thread */
       NULL,                                  /* exit process */
       NULL,                                  /* exit master */
       NGX_MODULE_V1_PADDING
   };

   将upstream中的server加入到定时器
      ngx_http_upstream_check_add_timer
      回调 ngx_http_upstream_check_begin_handler
           ngx_http_upstream_check_connect_handler
               ngx_event_connect_peer链接后端，加入epoll
      同时设置timeout的定时器回调 ngx_http_upstream_check_timeout_handler
      根据不同协议，设置不同处理的回调函数，主要放在这个结构体中ngx_check_types

六、四层负载健康检查-ngx_stream_upstream_check_module
   定义模块：
   cmd：
      "check": 开启健康检查
      "check_keepalive_requests": 
      "check_http_send":
      "check_http_expect_alive":
      "check_shm_size"
    先调用cmd set(cf, cmd, conf)

    ctx:  ngx_stream_block部门调用，http模块的在ngx_http_block
       create_main_conf: ngx_stream_upstream_check_create_main_conf
       init_main_conf: ngx_stream_upstream_check_init_main_conf
       create_srv_conf: ngx_stream_upstream_check_create_srv_conf
    ctx的几个函数：
    preconfiguration
    postconfiguration
    create_xxx_conf
    init_xxx_conf
    merge_xxx_conf
    都在解析"http"指令函数ngx_http_block, "stream"指令函数ngx_stream_block
    "mail"指令函数ngx_mail_block等函数调用

    ngx_conf_handler函数调用指令函数set()

   ngx_module_t  ngx_stream_upstream_check_module = {
       NGX_MODULE_V1,
       &ngx_stream_upstream_check_module_ctx,   /* module context */
       ngx_stream_upstream_check_commands,      /* module directives */
       NGX_STREAM_MODULE,                       /* module type */
       NULL,                                  /* init master */
       NULL,                                  /* init module */
       ngx_stream_upstream_check_init_process,  /* init process */
       NULL,                                  /* init thread */
       NULL,                                  /* exit thread */
       NULL,                                  /* exit process */
       NULL,                                  /* exit master */
       NGX_MODULE_V1_PADDING
   };     

   ngx_worker_process_init调用init_process
   ngx_init_modules函数调用init_master

   init_master暂时没有调用

   全局的stream_peers_ctx

   ngx_stream_upstream_check_init_process
       ngx_stream_upstream_check_add_timers将servers加入到定时器中
       设置回调函数ngx_stream_upstream_check_begin_handler， 
         ngx_stream_upstream_check_timeout_handler
         ngx_stream_upstream_check_connect_handler
         ngx_event_connect_peer 连接后后端，将fd加入到epoll
   ngx_check_types
   不同的类型的健康检查类型，设置不同的回调函数。
   支持UDP健康检查
   { NGX_CHECK_TYPE_UDP,
    ngx_string("udp"),
    ngx_string("X"),/* zhoucx: default send data.
                        (we must send some data and then call recv to trigger icmp error response).*/
    0,
    ngx_stream_upstream_check_send_handler,
    ngx_stream_upstream_check_peek_handler,
    ngx_stream_upstream_check_http_init,
    NULL,
    ngx_stream_upstream_check_http_reinit,
    1,    // (changxun): when send data, we need pool
    0 },


七、dyups模块
   “dyups_interface”
   ngx_http_dyups_interface
       clcf->handler = ngx_http_dyups_interface_handler;
   ngx_http_dyups_interface_read_body
       ngx_http_read_client_request_body

   在ngx_http_core_module
       ngx_http_update_location_config
           r->content_handler = clcf->handler;
   ngx_http_core_content_phase在这个阶段开始调用ngx_http_finalize_request(r, r->content_handler(r));

八、epoll
   1) Nginx中Epoll时间的应用https://blog.csdn.net/xiajun07061225/article/details/9250341
   2) epoll的详解 https://blog.csdn.net/xiajun07061225/article/details/9250579   
      一颗红黑树，一张准备就绪句柄链表，少量的内核cache，就帮我们解决了大并发下的socket处理问题
      epoll_creat() 创建ef句柄，创建红黑树，一个就绪句柄的双向链表
      epoll_ctl 将fd注册到红黑树，并设置回调函数，fd有事件时，kernel调用回调函数，将有事件的fd插入到就绪的双向链表
      epoll_wait 只需遍历就绪的的双向链表即可

九、ngx_lua
  https://github.com/openresty/lua-nginx-module      

十、Nginx优秀的架构设计
   1、优秀的模块化设计
   2、事件驱动架构
   3、请求的多阶段的异步处理
   4、master-worker的多进程管理模式
   5、平台无关的代码实现
   6、优秀的内存池设计
   7、同一的HTTP过滤模块  

十一、Nginx Proxy cache
   https://www.lisirlife.com/2019/09/25/Nginx%E7%BC%93%E5%AD%98%E6%A8%A1%E5%9D%97%E8%AF%A6%E8%A7%A3-proxy-cache/ 

十二、Nginx的内存管理、内存池管理
    https://www.cnblogs.com/didiaoxiong/p/nginx_memory.html
    https://blog.csdn.net/v_july_v/article/details/7040425
    1、内存对齐
    2、内存分页
    3、内存分配归结为大内存分配和小内存分配
    4、connection，request都是用内存池分配的
    5、内存池用链表

十三、Nginx的共享内存管理
    https://segmentfault.com/a/1190000017016228
    https://my.oschina.net/u/2310891/blog/672539
    https://blog.csdn.net/juzihongle1/article/details/77891953
    
    1、将内存按照页进行分配，每页的大小相同, 此处设为page_size。
    2、将内存块按照2的整数次幂进行划分, 最小为8bit, 最大为page_size/2。例如，假设每页大小为4Kb, 则将内存分为8, 16, 32, 64, 128, 256, 512, 1024, 2048共9种，每种对应一个slot, 此时slots数组的大小n即为9。申请小块内存(申请内存大小size <= page_size/2)时，直接给用户这9种中的一种，例如，需要30bit时，找大小为32的内存块提供给用户。
    3、每个页只会划分一种类型的内存块。例如，某次申请内存时，现有内存无法满足要求，此时会使用一个新的页，则这个新页此后只会分配这种大小的内存。
    4、通过双向链表将所有空闲的页连接。图中ngx_slab_pool_t中的free变量即使用来链接空闲页的。
    5、通过slots数组将所有小块内存所使用的页链接起来。
    6、对于大于等于页面大小的空间请求，计算所需页数，找到连续的空闲页，将空闲页的首页地址返回给客户使用，通过每页的管理结构ngx_slab_page_t进行标识。
    7、所有页面只会有3中状态，空闲、未满、已满。空闲，未满都是通过双向链表进行整合，已满页面则不存在与任何页面，当空间被释放时，会将其加入到某个链表。     
