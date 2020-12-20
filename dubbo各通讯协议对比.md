<table border="1" cellspacing="0"><tbody><tr><td style="border-color:#dddddd;"><span style="color:#4f4f4f;">协议名称</span></td>
			<td style="border-color:#dddddd;"><span style="color:#4f4f4f;">实现描述</span></td>
			<td style="border-color:#dddddd;"><span style="color:#4f4f4f;">连接</span></td>
			<td style="border-color:#dddddd;"><span style="color:#4f4f4f;">使用场景</span></td>
		</tr><tr><td style="border-color:#dddddd;"><span style="color:#4f4f4f;">dubbo</span></td>
			<td style="border-color:#dddddd;"><span style="color:#4f4f4f;">传输：mina、netty、grizzy<br><br>
			序列化：dubbo、hessian2、java、json</span></td>
			<td style="border-color:#dddddd;"><span style="color:#4f4f4f;">&nbsp;&nbsp;&nbsp; dubbo缺省采用单一长连接和NIO异步通讯&nbsp;&nbsp;&nbsp;</span></td>
			<td style="border-color:#dddddd;"><span style="color:#4f4f4f;">1.传入传出参数数据包较小<br><br>
			2.消费者 比提供者多<br><br>
			3.常规远程服务方法调用<br><br>
			4.不适合传送大数据量的服务，比如文件、传视频</span></td>
		</tr><tr><td style="border-color:#dddddd;"><span style="color:#4f4f4f;">rmi</span></td>
			<td style="border-color:#dddddd;"><span style="color:#4f4f4f;">传输：java&nbsp; rmi<br><br>
			序列化：java 标准序列化</span></td>
			<td style="border-color:#dddddd;"><span style="color:#4f4f4f;">&nbsp;&nbsp;&nbsp;&nbsp;<br><br>
			连接个数：多连接<br><br>
			连接方式：短连接<br><br>
			传输协议：TCP/IP<br><br>
			传输方式：BIO</span></td>
			<td style="border-color:#dddddd;"><br><span style="color:#4f4f4f;">1.常规RPC调用<br><br>
			2.与原RMI客户端互操作<br><br>
			3.可传文件<br><br>
			4.不支持防火墙穿透</span></td>
		</tr><tr><td style="border-color:#dddddd;"><span style="color:#4f4f4f;">hessian</span></td>
			<td style="border-color:#dddddd;"><br><span style="color:#4f4f4f;">传输：Servlet容器<br><br>
			序列化：hessian二进制序列化<br>
			&nbsp;&nbsp;&nbsp;</span></td>
			<td style="border-color:#dddddd;"><br><span style="color:#4f4f4f;">&nbsp;&nbsp;&nbsp; 连接个数：多连接<br>
			&nbsp;&nbsp;&nbsp; 连接方式：短连接<br>
			&nbsp;&nbsp;&nbsp; 传输协议：HTTP<br>
			&nbsp;&nbsp;&nbsp; 传输方式：同步传输<br><br>
			&nbsp;&nbsp;&nbsp;</span></td>
			<td style="border-color:#dddddd;"><br><span style="color:#4f4f4f;">1.提供者比消费者多<br><br>
			2.可传文件<br><br>
			3.跨语言传输</span></td>
		</tr><tr><td style="border-color:#dddddd;"><span style="color:#4f4f4f;">http</span></td>
			<td style="border-color:#dddddd;"><br><span style="color:#4f4f4f;">传输：servlet容器<br><br>
			序列化：表单序列化</span></td>
			<td style="border-color:#dddddd;"><span style="color:#4f4f4f;">&nbsp;&nbsp;&nbsp; 连接个数：多连接<br>
			&nbsp;&nbsp;&nbsp; 连接方式：短连接<br>
			&nbsp;&nbsp;&nbsp; 传输协议：HTTP<br>
			&nbsp;&nbsp;&nbsp; 传输方式：同步传输</span></td>
			<td style="border-color:#dddddd;"><span style="color:#4f4f4f;">1.提供者多于消费者<br><br>
			2.数据包混合</span></td>
		</tr><tr><td style="border-color:#dddddd;"><span style="color:#4f4f4f;">webservice</span></td>
			<td style="border-color:#dddddd;"><br><span style="color:#4f4f4f;">传输：HTTP<br><br>
			序列化：SOAP文件序列化</span></td>
			<td style="border-color:#dddddd;"><span style="color:#4f4f4f;">&nbsp;&nbsp;&nbsp; 连接个数：多连接<br>
			&nbsp;&nbsp;&nbsp; 连接方式：短连接<br>
			&nbsp;&nbsp;&nbsp; 传输协议：HTTP<br>
			&nbsp;&nbsp;&nbsp; 传输方式：同步传输</span></td>
			<td style="border-color:#dddddd;"><br><span style="color:#4f4f4f;">1.系统集成<br><br>
			2.跨语言调用</span></td>
		</tr><tr><td style="border-color:#dddddd;"><span style="color:#4f4f4f;">thrift</span></td>
			<td style="border-color:#dddddd;"><br><span style="color:#4f4f4f;">&nbsp;&nbsp;&nbsp; 与thrift RPC实现集成，并在基础上修改了报文头&nbsp;&nbsp;&nbsp;</span></td>
			<td style="border-color:#dddddd;"><br><span style="color:#4f4f4f;">长连接、NIO异步传输&nbsp;&nbsp;&nbsp;</span></td>
			<td style="border-color:#dddddd;"><span style="color:#4f4f4f;">&nbsp;</span></td>
		</tr></tbody></table>





