# `Express.js` 相关

## 请求超时时间

```js
const express = require('express');
const app = express();

// 创建 HTTP 服务器
const server = app.listen(3000, () => {
  console.log('Server started');
});

// 设置超时时间为 5 秒 (单位: 毫秒)
server.setTimeout(5000); // 👈 关键设置

// 可选：监听超时事件
server.on('timeout', (socket) => {
  console.log('Request timed out. Closing socket.');
  socket.destroy(); // 强制关闭连接
});

// 示例路由
app.get('/', (req, res) => {
  // 模拟长时间操作 (超过5秒会触发超时)
  setTimeout(() => {
    res.send('Hello World');
  }, 10000); // 10秒响应
});
```





## 跨域相关（后端、运维）

### 后端启用 CORS

```js
// Node.js 示例（Express）
app.use((req, res, next) => {
  res.header('Access-Control-Allow-Origin', 'http://localhost:8080') // 你的前端地址
  res.header('Access-Control-Allow-Methods', 'GET, POST, PUT, DELETE')
  res.header('Access-Control-Allow-Headers', 'Content-Type, Authorization')
  next()
})
```



### 配置本地 Nginx 反向代理

```nginx
# nginx.conf
server {
  listen 80;
  server_name local.dev;

  location / {
    proxy_pass http://localhost:3000; # 前端服务
  }

  location /api/ {
    proxy_pass http://your-backend.com/; # 后端服务
    proxy_set_header Host $host;
  }
}
```

