---
title: 'hdfs支持双栈ipv4和ipv6'
date: 2017-11-20T22:37:40.475Z
tags: java
---
### 前言
ipv6标准制定了这么多年，终于要开始被大力推进了。最近要求hdfs既要支持ipv4，又要支持ipv6。所以特意研究了，怎么同时能支持ipv4和ipv6。
### 理论基础
ipv4和ipv6都是属于网络层，我们平常做的程序很少有机会去做到网络层，除非做的路由器和交换机。我们开发的程序大都是在传输层以上，基于tcp协议或者udp的协议。所以，我猜测应该我需要做的东西不是很多。找了找java建立网络的相关api。果然就被我发现了。

```
 /**
     * Create a server with the specified port, listen backlog, and
     * local IP address to bind to.  The <i>bindAddr</i> argument
     * can be used on a multi-homed host for a ServerSocket that
     * will only accept connect requests to one of its addresses.
     * If <i>bindAddr</i> is null, it will default accepting
     * connections on any/all local addresses.
     * The port must be between 0 and 65535, inclusive.
     * A port number of {@code 0} means that the port number is
     * automatically allocated, typically from an ephemeral port range.
     * This port number can then be retrieved by calling
     * {@link #getLocalPort getLocalPort}.
     *
     * <P>If there is a security manager, this method
     * calls its {@code checkListen} method
     * with the {@code port} argument
     * as its argument to ensure the operation is allowed.
     * This could result in a SecurityException.
     *
     * The {@code backlog} argument is the requested maximum number of
     * pending connections on the socket. Its exact semantics are implementation
     * specific. In particular, an implementation may impose a maximum length
     * or may choose to ignore the parameter altogther. The value provided
     * should be greater than {@code 0}. If it is less than or equal to
     * {@code 0}, then an implementation specific default will be used.
     * <P>
     * @param port  the port number, or {@code 0} to use a port
     *              number that is automatically allocated.
     * @param backlog requested maximum length of the queue of incoming
     *                connections.
     * @param bindAddr the local InetAddress the server will bind to
     *
     * @throws  SecurityException if a security manager exists and
     * its {@code checkListen} method doesn't allow the operation.
     *
     * @throws  IOException if an I/O error occurs when opening the socket.
     * @exception  IllegalArgumentException if the port parameter is outside
     *             the specified range of valid port values, which is between
     *             0 and 65535, inclusive.
     *
     * @see SocketOptions
     * @see SocketImpl
     * @see SecurityManager#checkListen
     * @since   JDK1.1
     */
    public ServerSocket(int port, int backlog, InetAddress bindAddr) throws IOException {
        setImpl();
        if (port < 0 || port > 0xFFFF)
            throw new IllegalArgumentException(
                       "Port value out of range: " + port);
        if (backlog < 1)
          backlog = 50;
        try {
            bind(new InetSocketAddress(bindAddr, port), backlog);
        } catch(SecurityException e) {
            close();
            throw e;
        } catch(IOException e) {
            close();
            throw e;
        }
    }
    
```
我们在建离socket连接，就是tcp的连接的时候，需要传递三个参数。我去查了下这三个数的含义。从上面的注释可以发现，port表示要绑定的端口号，backlog等待建立连接的最大连接数，binAddr则就是要绑定的端口号。如果不传入binAddr这个参数，那么就会默认监听这台机器上所有地址。

那么问题，基本上就简单解决了好多，我只要在建立rpc请求的时候，不绑定ip地址和端口号，那么我不就可以同时监听ipv4和ipv6的地址么。果然，我在hdfs中找到了相关的配置项。

```
<property>
  <name>dfs.namenode.http-bind-host</name>
  <value></value>
  <description>
    The actual adress the HTTP server will bind to. If this optional address
    is set, it overrides only the hostname portion of dfs.namenode.http-address.
    It can also be specified per name node or name service for HA/Federation.
    This is useful for making the name node HTTP server listen on all
    interfaces by setting it to 0.0.0.0.
  </description>
</property>
```
果然配置成0.0.0.0，namenode的50070端口，ipv4和ipv6的端口都可以访问。

