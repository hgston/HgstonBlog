---
date: '2025-09-21T08:32:45Z'
draft: false
title: 'C实现: 简单的HTTP服务器'
categories: ["编程开发"]
---

## 用 C 语言在 Linux 环境下实现一个 HTTP 服务器


这是一个非常棒的项目，能让你体验感受网络编程、HTTP 协议、系统调用和服务器架构。我会从零开始，带你一步一步实现一个简单的、可运行的 HTTP 服务器。

---

## 一、前置知识（你需要掌握或了解的内容）

### 1. 基础 C 语言

- 变量、函数、指针、结构体
- 文件操作（`fopen`, `fread`, `fclose`）
- 字符串处理（`strcpy`, `strcat`, `strstr`, `strlen` 等）
- 动态内存（`malloc`, `free`）

如果你已经会写简单的 C 程序（比如打印、读文件、函数调用），就可以继续。

---

### 2. Linux 基础

- 如何在终端编译运行 C 程序（`gcc`）
- 基本命令：`ls`, `cd`, `mkdir`, `cat`, `curl`, `netstat`, `ps`
- 理解“端口”、“进程”、“后台运行”等概念

---

### 3. TCP/IP 与 Socket 编程基础

HTTP 是基于 TCP 的应用层协议。你需要了解：

- **Socket 是什么？** —— 网络通信的“端点”，像文件描述符一样可以读写。
- **服务器流程：**
  1. 创建 socket → `socket()`
  2. 绑定地址和端口 → `bind()`
  3. 监听连接 → `listen()`
  4. 接受客户端 → `accept()`
  5. 读写数据 → `read()` / `write()`
  6. 关闭连接 → `close()`

重点函数：

```c
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <unistd.h>
```

---

### 4. HTTP 协议基础（简化版）

我们只实现 HTTP/1.1 的 GET 请求。

一个典型的 HTTP 请求：

```
GET /index.html HTTP/1.1
Host: localhost:8080
User-Agent: curl/7.68.0
...
```

响应格式：

```
HTTP/1.1 200 OK
Content-Type: text/html
Content-Length: 123

<html>...</html>
```

你需要知道：

- 请求行（GET /path HTTP/1.1）
- 响应状态码（200, 404, 500）
- 响应头 + 空行 + 响应体

---

## 二、开发环境准备

确保你有：

- Linux 系统（Ubuntu/CentOS/WSL 都可以）
- gcc 编译器：`sudo apt install build-essential`
- 测试工具：`curl`、浏览器

---

## 三、第一步：写一个最简单的 TCP 服务器（回显服务器）

我们先不处理 HTTP，只实现“客户端发什么，服务器原样返回什么”。

创建文件 `step1_echo_server.c`

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>

#define PORT 8080
#define BUFFER_SIZE 1024

int main() {
    int server_fd, client_fd;
    struct sockaddr_in address;
    int addrlen = sizeof(address);
    char buffer[BUFFER_SIZE] = {0};

    // 1. 创建 socket
    if ((server_fd = socket(AF_INET, SOCK_STREAM, 0)) == 0) {
        perror("socket failed");
        exit(EXIT_FAILURE);
    }

    // 2. 绑定地址和端口
    address.sin_family = AF_INET;
    address.sin_addr.s_addr = INADDR_ANY; // 监听所有网卡
    address.sin_port = htons(PORT);

    if (bind(server_fd, (struct sockaddr *)&address, sizeof(address)) < 0) {
        perror("bind failed");
        close(server_fd);
        exit(EXIT_FAILURE);
    }

    // 3. 开始监听
    if (listen(server_fd, 3) < 0) {
        perror("listen failed");
        close(server_fd);
        exit(EXIT_FAILURE);
    }

    printf("服务器启动，监听端口 %d\n", PORT);

    // 4. 循环接受客户端
    while (1) {
        if ((client_fd = accept(server_fd, (struct sockaddr *)&address, (socklen_t*)&addrlen)) < 0) {
            perror("accept failed");
            continue;
        }

        printf("新客户端连接\n");

        // 5. 读取客户端数据
        int valread = read(client_fd, buffer, BUFFER_SIZE);
        if (valread > 0) {
            printf("收到: %s\n", buffer);
            // 6. 原样返回
            write(client_fd, buffer, valread);
        }

        close(client_fd);
    }

    return 0;
}
```

编译运行：

```bash
gcc step1_echo_server.c -o server
./server
```

在另一个终端测试：

```bash
telnet localhost 8080
# 或者
echo "hello" | nc localhost 8080
```

如果看到服务器打印收到的内容并返回，说明 TCP 服务器工作正常！

---

## 四、第二步：解析 HTTP 请求

现在我们让服务器能识别 HTTP 请求，提取路径。

修改上面的代码，在 `read` 之后解析第一行：

```c
// 在 read 之后添加：
char method[16], path[256], protocol[16];
sscanf(buffer, "%s %s %s", method, path, protocol);

printf("方法: %s, 路径: %s, 协议: %s\n", method, path, protocol);
```

测试：

```bash
curl -v http://localhost:8080/index.html
```

你应该看到：

```
方法: GET, 路径: /index.html, 协议: HTTP/1.1
```

---

## 五、第三步：根据路径返回文件内容

我们假设网站根目录是 `./www`，创建它并放一个 `index.html`：

```bash
mkdir www
echo "<h1>Hello from C HTTP Server!</h1>" > www/index.html
```

然后修改服务器代码，读取文件并返回：

```c
// 在解析 path 之后添加：

// 构造文件路径
char filepath[512];
if (strcmp(path, "/") == 0) {
    strcpy(filepath, "./www/index.html");
} else {
    snprintf(filepath, sizeof(filepath), "./www%s", path);
}

// 打开文件
FILE *fp = fopen(filepath, "r");
if (fp == NULL) {
    // 文件不存在，返回 404
    const char *not_found = "HTTP/1.1 404 Not Found\r\nContent-Type: text/plain\r\nContent-Length: 9\r\n\r\nNot Found";
    write(client_fd, not_found, strlen(not_found));
} else {
    // 读取文件内容到内存
    fseek(fp, 0, SEEK_END);
    long fsize = ftell(fp);
    fseek(fp, 0, SEEK_SET);

    char *content = malloc(fsize + 1);
    fread(content, 1, fsize, fp);
    content[fsize] = '\0';
    fclose(fp);

    // 构造 HTTP 响应头
    char header[512];
    snprintf(header, sizeof(header),
        "HTTP/1.1 200 OK\r\n"
        "Content-Type: text/html\r\n"
        "Content-Length: %ld\r\n"
        "Connection: close\r\n"
        "\r\n", fsize);

    // 发送响应头 + 内容
    write(client_fd, header, strlen(header));
    write(client_fd, content, fsize);

    free(content);
}
```

注意：

- 响应头和响应体之间必须有 `\r\n\r\n`
- `Content-Length` 必须准确，否则浏览器可能卡住
- 使用 `text/html` 作为默认 Content-Type（后续可扩展）

---

## 六、完整可运行代码（v1.0）

创建 `http_server.c`

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>

#define PORT 8080
#define BUFFER_SIZE 4096
#define WWW_ROOT "./www"

void serve_file(int client_fd, const char *filepath) {
    FILE *fp = fopen(filepath, "r");
    if (!fp) {
        const char *response = "HTTP/1.1 404 Not Found\r\n"
                               "Content-Type: text/plain\r\n"
                               "Content-Length: 9\r\n"
                               "Connection: close\r\n"
                               "\r\n"
                               "Not Found";
        write(client_fd, response, strlen(response));
        return;
    }

    // 获取文件大小
    fseek(fp, 0, SEEK_END);
    long fsize = ftell(fp);
    fseek(fp, 0, SEEK_SET);

    // 分配内存读取文件
    char *content = malloc(fsize + 1);
    if (!content) {
        fclose(fp);
        const char *response = "HTTP/1.1 500 Internal Server Error\r\nContent-Length: 0\r\n\r\n";
        write(client_fd, response, strlen(response));
        return;
    }

    fread(content, 1, fsize, fp);
    content[fsize] = '\0';
    fclose(fp);

    // 构造响应头（简单处理，只支持 html）
    char header[1024];
    snprintf(header, sizeof(header),
        "HTTP/1.1 200 OK\r\n"
        "Content-Type: text/html\r\n"
        "Content-Length: %ld\r\n"
        "Connection: close\r\n"
        "\r\n", fsize);

    // 发送响应
    write(client_fd, header, strlen(header));
    write(client_fd, content, fsize);

    free(content);
}

int main() {
    int server_fd, client_fd;
    struct sockaddr_in address;
    int addrlen = sizeof(address);
    char buffer[BUFFER_SIZE] = {0};

    // 创建 socket
    if ((server_fd = socket(AF_INET, SOCK_STREAM, 0)) == 0) {
        perror("socket failed");
        exit(EXIT_FAILURE);
    }

    // 设置端口复用（避免 TIME_WAIT 问题）
    int opt = 1;
    setsockopt(server_fd, SOL_SOCKET, SO_REUSEADDR, &opt, sizeof(opt));

    address.sin_family = AF_INET;
    address.sin_addr.s_addr = INADDR_ANY;
    address.sin_port = htons(PORT);

    if (bind(server_fd, (struct sockaddr *)&address, sizeof(address)) < 0) {
        perror("bind failed");
        close(server_fd);
        exit(EXIT_FAILURE);
    }

    if (listen(server_fd, 10) < 0) {
        perror("listen failed");
        close(server_fd);
        exit(EXIT_FAILURE);
    }

    printf("HTTP 服务器启动，访问 http://localhost:%d\n", PORT);

    while (1) {
        if ((client_fd = accept(server_fd, (struct sockaddr *)&address, (socklen_t*)&addrlen)) < 0) {
            perror("accept failed");
            continue;
        }

        // 读取请求
        int valread = read(client_fd, buffer, BUFFER_SIZE - 1);
        if (valread <= 0) {
            close(client_fd);
            continue;
        }
        buffer[valread] = '\0';

        // 解析第一行
        char method[16], path[256], protocol[16];
        if (sscanf(buffer, "%s %s %s", method, path, protocol) != 3) {
            close(client_fd);
            continue;
        }

        printf("请求: %s %s\n", method, path);

        // 只处理 GET
        if (strcmp(method, "GET") != 0) {
            const char *response = "HTTP/1.1 501 Not Implemented\r\nContent-Length: 0\r\n\r\n";
            write(client_fd, response, strlen(response));
        } else {
            // 构造文件路径
            char filepath[512];
            if (strcmp(path, "/") == 0) {
                strcpy(filepath, WWW_ROOT "/index.html");
            } else {
                snprintf(filepath, sizeof(filepath), "%s%s", WWW_ROOT, path);
            }
            serve_file(client_fd, filepath);
        }

        close(client_fd);
    }

    return 0;
}
```

编译运行：

```bash
mkdir www
echo "<h1>恭喜！你的 C 语言 HTTP 服务器成功运行！</h1>" > www/index.html
gcc http_server.c -o server
./server
```

测试：

- 浏览器打开：`http://localhost:8080`
- 终端测试：`curl http://localhost:8080`

---

## 七、后续可扩展功能

1. **支持多线程/多进程** —— 同时处理多个请求
2. **支持 POST 请求**
3. **支持 MIME 类型自动识别**（根据文件扩展名）
4. **支持目录列表**
5. **支持 CGI / 动态内容**
6. **日志系统**
7. **配置文件**
8. **守护进程化（daemon）**
9. **支持 HTTPS（使用 OpenSSL）**

---

## 八、推荐学习资料

- 《UNIX 网络编程 卷1：套接字联网API》—— W. Richard Stevens
- 《TCP/IP 详解 卷1》
- MDN HTTP 文档：https://developer.mozilla.org/zh-CN/docs/Web/HTTP
- Beej's Guide to Network Programming（英文，但非常清晰）

---

## 总结

你现在已经：

- 理解了 HTTP 服务器的基本原理  
- 掌握了 Linux 下 Socket 编程流程  
- 实现了一个能返回静态文件的 HTTP 服务器  
- 为后续扩展打下坚实基础

---
