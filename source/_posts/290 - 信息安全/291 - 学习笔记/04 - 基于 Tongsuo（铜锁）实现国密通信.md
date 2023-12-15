---
title: 基于 Tongsuo（铜锁）实现国密通信
date: 2023-12-15
categories:
  - 信息安全
tags:
  - Tongsuo
  - 国密
  - TLCP
published: true
---
## 1 Tongsuo（铜锁）的编译

```bash
git clone https://github.com/Tongsuo-Project/Tongsuo.git
cd Tongsuo
./config --prefix=/opt/tongsuo enable-ntls no-shared
make -j
sudo make install
```

## 2 生成服务端测试证书

1. 生成一个 EC 密钥对（私钥和公钥），使用 SM2 曲线参数，保存私钥为 `server_sign.key` 文件：    

```bash
openssl genpkey -algorithm ec -pkeyopt ec_paramgen_curve:sm2 -out server_sign.key
```

2. 使用私钥生成一个证书签名请求（CSR），并将其保存为 `server_sign.csr` 文件。该请求使用 SM3 散列算法，并且采用给定的主题信息：

```bash
openssl req -config $CONF_DIR/subca.cnf -key server_sign.key -new -out server_sign.csr -sm3 -nodes -subj "/C=AA/ST=BB/O=CC/OU=DD/CN=server sign"
```

- `-config $CONF_DIR/subca.cnf`: 指定配置文件，包含了生成 CSR 所需的详细信息和设置。
- `-key server_sign.key`: 指定用于生成 CSR 的私钥文件路径。
- `-new`: 创建新的证书签名请求。
- `-out server_sign.csr`: 指定生成的 CSR 文件名。
- `-sm3`: 使用 SM3 散列算法。
- `-nodes`: 生成的私钥不加密。
- `-subj "/C=AA/ST=BB/O=CC/OU=DD/CN=server sign"`: 指定证书主题信息。

3. 使用 CA 配置文件（假设为 `$CONF_DIR/subca.cnf`）中定义的设置和参数，通过 CA 签发证书，将上一步生成的 CSR 文件 `server_sign.csr` 作为输入。签发的证书输出到 `server_sign.crt` 文件中：

```bash
openssl ca -config $CONF_DIR/subca.cnf -extensions server_sign_req -days 3650 -in server_sign.csr -notext -out server_sign.crt -md sm3 -batch
```

- `-config $CONF_DIR/subca.cnf`: 指定用于签发证书的 CA 配置文件。
- `-extensions server_sign_req`: 指定要应用的扩展名。
- `-days 3650`: 设置证书有效期为 3650 天（约合 10 年）。
- `-in server_sign.csr`: 指定输入的证书签名请求文件。
- `-notext`: 生成的证书不包含文本。
- `-out server_sign.crt`: 指定生成的证书文件名。
- `-md sm3`: 使用 SM3 消息摘要算法进行签名。
- `-batch`: 在执行证书签发过程中避免交互式操作。

## 3 国密服务端程序

基于 Tongsuo（铜锁）开发服务端程序：
```C
#include <stdio.h>
#include <unistd.h>
#include <string.h>
#include <sys/socket.h>
#include <arpa/inet.h>
#include <openssl/ssl.h>
#include <openssl/err.h>

int main(int argc, char **argv)
{
    struct sockaddr_in addr;
    unsigned int addr_len = sizeof(addr);
    const SSL_METHOD *method;
    SSL_CTX *ssl_ctx = NULL;
    SSL *ssl = NULL;
    int fd = -1, conn_fd = -1;
    char *txbuf = NULL;
    size_t txcap = 0;
    int txlen;
    char rxbuf[128];
    size_t rxcap = sizeof(rxbuf);
    int rxlen;
    char *server_ip = "127.0.0.1";
    char *server_port = "443";
    int server_running = 1;
    int optval = 1;

    if (argc == 2) {
        server_ip = argv[1];
        server_port = strstr(argv[1], ":");
        if (server_port != NULL)
            *server_port++ = '\0';
        else
            server_port = "443";
    }

    method = NTLS_server_method();
    ssl_ctx = SSL_CTX_new(method);
    if (ssl_ctx == NULL) {
        perror("Unable to create SSL context");
        ERR_print_errors_fp(stderr);
        exit(EXIT_FAILURE);
    }

    SSL_CTX_enable_ntls(ssl_ctx);

    /* Set the key and cert */
    if (!SSL_CTX_use_sign_certificate_file(ssl_ctx, "certs/server/sm2_sign.crt",
                                           SSL_FILETYPE_PEM)
        || !SSL_CTX_use_sign_PrivateKey_file(ssl_ctx, "certs/server/sm2_sign.key",
                                             SSL_FILETYPE_PEM)
        || !SSL_CTX_use_enc_certificate_file(ssl_ctx, "certs/server/sm2_enc.crt",
                                             SSL_FILETYPE_PEM)
        || !SSL_CTX_use_enc_PrivateKey_file(ssl_ctx, "certs/server/sm2_enc.key",
                                            SSL_FILETYPE_PEM)) {
        ERR_print_errors_fp(stderr);
        exit(EXIT_FAILURE);
    }

    fd = socket(AF_INET, SOCK_STREAM, 0);
    if (fd < 0) {
        perror("Unable to create socket");
        exit(EXIT_FAILURE);
    }

    addr.sin_family = AF_INET;
    inet_pton(AF_INET, server_ip, &addr.sin_addr.s_addr);
    addr.sin_port = htons(atoi(server_port));

    /* Reuse the address; good for quick restarts */
    if (setsockopt(fd, SOL_SOCKET, SO_REUSEADDR, &optval, sizeof(optval)) < 0) {
        perror("setsockopt(SO_REUSEADDR) failed");
        exit(EXIT_FAILURE);
    }

    if (bind(fd, (struct sockaddr*) &addr, sizeof(addr)) < 0) {
        perror("Unable to bind");
        exit(EXIT_FAILURE);
    }

    if (listen(fd, 1) < 0) {
        perror("Unable to listen");
        exit(EXIT_FAILURE);
    }

    printf("We are the server on port: %d\n\n", atoi(server_port));
    /*
     * Loop to accept clients.
     * Need to implement timeouts on TCP & SSL connect/read functions
     * before we can catch a CTRL-C and kill the server.
     */
    while (server_running) {
        /* Wait for TCP connection from client */
        conn_fd= accept(fd, (struct sockaddr*) &addr, &addr_len);
        if (conn_fd < 0) {
            perror("Unable to accept");
            exit(EXIT_FAILURE);
        }

        printf("Client TCP connection accepted\n");

        /* Create server SSL structure using newly accepted client socket */
        ssl = SSL_new(ssl_ctx);
        SSL_set_fd(ssl, conn_fd);

        /* Wait for SSL connection from the client */
        if (SSL_accept(ssl) <= 0) {
            ERR_print_errors_fp(stderr);
            server_running = 0;
        } else {

            printf("Client TLCP connection accepted\n\n");

            /* Echo loop */
            while (1) {
                /* Get message from client; will fail if client closes connection */
                if ((rxlen = SSL_read(ssl, rxbuf, rxcap)) <= 0) {
                    if (rxlen == 0) {
                        printf("Client closed connection\n");
                    }
                    ERR_print_errors_fp(stderr);
                    break;
                }
                /* Insure null terminated input */
                rxbuf[rxlen] = 0;
                /* Look for kill switch */
                if (strcmp(rxbuf, "kill\n") == 0) {
                    /* Terminate...with extreme prejudice */
                    printf("Server received 'kill' command\n");
                    server_running = 0;
                    break;
                }
                /* Show received message */
                printf("Received: %s", rxbuf);
                /* Echo it back */
                if (SSL_write(ssl, rxbuf, rxlen) <= 0) {
                    ERR_print_errors_fp(stderr);
                }
            }
        }
        if (server_running) {
            /* Cleanup for next client */
            SSL_shutdown(ssl);
            SSL_free(ssl);
            close(conn_fd);
            conn_fd = -1;
        }
    }
    printf("Server exiting...\n");

    exit:
    /* Close up */
    if (ssl != NULL) {
        SSL_shutdown(ssl);
        SSL_free(ssl);
    }
    SSL_CTX_free(ssl_ctx);

    if (conn_fd != -1)
        close(conn_fd);
    if (fd != -1)
        close(fd);

    if (txbuf != NULL && txcap > 0)
        free(txbuf);

    return 0;
}
// gcc server.c  -I/opt/tongsuo/include/ -L/opt/tongsuo/lib64/ -lssl -lcrypto -Wl,-rpath=/opt/tongsuo/lib64 -o server
```

启动服务端：
```shell
./server 192.168.170.137:443
```
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/202312151704112.png)

## 4 国密客户端程序

```c
#include <stdio.h>
#include <unistd.h>
#include <string.h>
#include <sys/socket.h>
#include <arpa/inet.h>
#include <openssl/ssl.h>
#include <openssl/err.h>

int main(int argc, char **argv)
{
    struct sockaddr_in addr;
    const SSL_METHOD *method;
    SSL_CTX *ssl_ctx = NULL;
    SSL *ssl = NULL;
    int fd = -1;
    char *txbuf = NULL;
    size_t txcap = 0;
    int txlen;
    char rxbuf[128];
    size_t rxcap = sizeof(rxbuf);
    int rxlen;
    char *server_ip = "127.0.0.1";
    char *server_port = "443";

    if (argc == 2) {
        server_ip = argv[1];
        server_port = strstr(argv[1], ":");
        if (server_port != NULL)
            *server_port++ = '\0';
        else
            server_port = "443";
    }

    method = NTLS_client_method();
    ssl_ctx = SSL_CTX_new(method);
    if (ssl_ctx == NULL) {
        perror("Unable to create SSL context");
        ERR_print_errors_fp(stderr);
        exit(EXIT_FAILURE);
    }

    SSL_CTX_enable_ntls(ssl_ctx);
    SSL_CTX_set_verify(ssl_ctx, SSL_VERIFY_NONE, NULL);

    fd = socket(AF_INET, SOCK_STREAM, 0);
    if (fd < 0) {
        perror("Unable to create socket");
        exit(EXIT_FAILURE);
    }

    addr.sin_family = AF_INET;
    inet_pton(AF_INET, server_ip, &addr.sin_addr.s_addr);
    addr.sin_port = htons(atoi(server_port));

    if (connect(fd, (struct sockaddr*) &addr, sizeof(addr)) != 0) {
        perror("Unable to TCP connect to server");
        goto exit;
    } else {
        printf("TCP connection to server successful\n");
    }

    /* Create client SSL structure using dedicated client socket */
    ssl = SSL_new(ssl_ctx);
    SSL_set_fd(ssl, fd);

    if (SSL_connect(ssl) == 1) {
        printf("TLCP connection to server successful\n\n");

        /* Loop to send input from keyboard */
        while (1) {
            /* Get a line of input */
            txlen = getline(&txbuf, &txcap, stdin);
            /* Exit loop on error */
            if (txlen < 0 || txbuf == NULL) {
                break;
            }
            /* Exit loop if just a carriage return */
            if (txbuf[0] == '\n') {
                break;
            }
            /* Send it to the server */
            if (SSL_write(ssl, txbuf, txlen) <= 0) {
                printf("Server closed connection\n");
                ERR_print_errors_fp(stderr);
                break;
            }

            /* Wait for the echo */
            rxlen = SSL_read(ssl, rxbuf, rxcap);
            if (rxlen <= 0) {
                printf("Server closed connection\n");
                ERR_print_errors_fp(stderr);
                break;
            } else {
                /* Show it */
                rxbuf[rxlen] = 0;
                printf("Received: %s", rxbuf);
            }
        }
        printf("Client exiting...\n");
    } else {

        printf("SSL connection to server failed\n\n");

        ERR_print_errors_fp(stderr);
    }
    exit:
    if (ssl != NULL) {
        SSL_shutdown(ssl);
        SSL_free(ssl);
    }
    SSL_CTX_free(ssl_ctx);

    if (fd != -1)
        close(fd);
    if (txbuf != NULL && txcap > 0)
        free(txbuf);

    return 0;
}
// gcc client.c  -I/opt/tongsuo/include/ -L/opt/tongsuo/lib64/ -lssl -lcrypto -Wl,-rpath=/opt/tongsuo/lib64 -o client
```
执行客户端：
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/202312151703841.png)
