# Dog Pile Effect

## 解析
当缓存超时失效的时候，同一时间有大量的请求访问服务的时候，造成服务器压力突增或宕机的情况

## 解决方法

### 使用独立进程更新

### 加入锁机制

	local data = redis:get(key)
	if data != nil then
		return data
	end

	if lock:lock(key) == success then
		redis:add(key, data)
		lock:unlock(key)
		return data
	else
		whiel(true) do
			usleep(1000)
			data = redis:get(key)
			if data != nil then
				return data
			end
		end
