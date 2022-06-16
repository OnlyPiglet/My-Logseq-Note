title:: openresty ngx.socket.tcp 分析
lua 调用方法
```lua
local sock = ngx.socket.tcp()
sock:settimeout(5000)
sock:connect(...)
```

```c
static void
ngx_resolver_expire(ngx_resolver_t *r, ngx_rbtree_t *tree, ngx_queue_t *queue)
{
    time_t                now;
    ngx_uint_t            i;
    ngx_queue_t          *q;
    ngx_resolver_node_t  *rn;

    ngx_log_debug0(NGX_LOG_DEBUG_CORE, r->log, 0, "resolver expire");

    now = ngx_time();

    for (i = 0; i < 2; i++) {
        if (ngx_queue_empty(queue)) {
            return;
        }

        q = ngx_queue_last(queue);

        rn = ngx_queue_data(q, ngx_resolver_node_t, queue);

        if (now <= rn->expire) {
            return;
        }

        ngx_log_debug2(NGX_LOG_DEBUG_CORE, r->log, 0,
                       "resolver expire \"%*s\"", (size_t) rn->nlen, rn->name);

        ngx_queue_remove(q);

        ngx_rbtree_delete(tree, &rn->node);

        ngx_resolver_free_node(r, rn);
    }
}
```

```c
//接受 lua_State 参数，创建tcp连接
static int
ngx_http_lua_socket_tcp_connect(lua_State *L)
```
