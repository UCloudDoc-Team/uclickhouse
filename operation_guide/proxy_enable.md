# 开启代理

UClickhouse 可以通过网络型负载均衡 NLB 实现集群节点代理，提供统一的接入地址。

同时可以在创建 NLB 时，按需选择是否开放外网功能。

## 开启步骤

1. 创建 NLB

    如果需要外网访问，则需要购买外网弹性IP

    ![img](/images/proxy-enable-1.png)

2. NLB 添加监听器

    - 协议和端口：选择TCP协议。
    - 端口：按需填写，如要代理 UClickhouse HTTP 端口，则填8123；如果要代理 TCP 端口，则填写 9000.
    - 负载均衡算法：可以按需选择，可以使用默认的“轮询”算法。

    ![img](/images/proxy-enable-2.png)
    ![img](/images/proxy-enable-3.png)

3. NLB 监听器添加节点

    - 资源类型：内网IP
    - 可选资源：填写 UClickhouse 集群节点IP

    ![img](/images/proxy-enable-4.png)

4. NLB 健康检查

    添加节点后，需等待健康检查通过

    ![img](/images/proxy-enable-5.png)

5. 验证

    ```shell
    # 8123 端口验证
    curl -X POST "http://${username}:${password}@${NLB_IP}:8123/" -d "SELECT * from default.test"

    # 9000 端口验证
    clickhouse-client --host ${NLB_IP} --port 9000 --user ${username} --password ${password} --query "SELECT * from default.test"
    ```
