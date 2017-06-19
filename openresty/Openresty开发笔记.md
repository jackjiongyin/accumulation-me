# Openresty开发笔记

## LuaJit和Lua
`LuaJit`是使用`C`语言和`Jit`技术编写的解释器，完成兼容`Lua5.1`版本，与标准`Lua`语法无异,对于`Lua5.2`,只支持部分编译，并且在底层`lua c api`上存在变化（`luaL_register`和`luaL_setfuncs`）


## 获取`ngx.shared.DICT`键值对

### 代码
	local keys = ngx.shared.DICT:get_keys(n)
	for index, key in pairs(keys) do
		ngx.say(ngx.shared.DICT:get(key))
	end

### 原因
`ngx.shared.DICT`为*非性质队列*，即使用红黑树实现列表结构，所以不能单纯用`pairs`遍历

## 单例`timer`
使用`ngx.worker.id()`绑定`timer`到特定一个`worker_id`上,即使改worker因为种种原因导致crash掉后，新生的worker仍然继承该id，这样就能保证定时器的可靠性

	init_worker_by_lua_block {
		local delay = 3  -- in seconds
		local new_timer = ngx.timer.at
		local log = ngx.log
		local ERR = ngx.ERR
		local check
		
		check = function(premature)
		 if not premature then
		     -- do the health check or other routine work
		     local ok, err = new_timer(delay, check)
		     if not ok then
		         log(ERR, "failed to create timer: ", err)
		         return
		     end
		 end
		end
		
		if 0 == ngx.worker.id() then
		 local ok, err = new_timer(delay, check)
		 if not ok then
		     log(ERR, "failed to create timer: ", err)
		     return
		 end
		end
	}
