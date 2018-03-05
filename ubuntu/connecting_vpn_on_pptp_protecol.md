# 基于 PPTP 协议的 VPN 连接

### 环境

- 操作系统：Ubuntu
- 系统版本：16.04.3 LTS

### 步骤

1. 打开`System Settings`应用。
2. 选择`Network`。
3. 在窗口左下角点击`+`。
4. `Interface`选择`VPN`，点击`Create...`。
5. `Connection Type`选择`PPTP`，点击`Create...`。
6. 在弹出框中填入连接信息：
   - Gateway：提供 VPN 服务的服务器IP地址。
   - User name：输入连接需要的用户名。 
   - Password：输入用户名相对应的密码。
7. 点击`Advanced...`按钮，在对话框中作如下选择后，点击`OK`
   - [ ] PAP
   - [ ] CHAP
   - [ ] MSCHAP
   - [x] MSCHAPv2
   - [x] Use Point-to-Point encryption(MPPE)
         - Security: 128-bit(most secure)
         - [ ] Allow stateful encryption
   - [x] Allow BSD data compression
   - [x] Allow Deflate data compression
   - [x] Use TCP header compression
   - [ ] Send PPP echo packets
   - [ ] Use custom unit number.
8. 点击`Save`按钮，保存 VPN 连接。
9. 任务栏上点击网络链接图标选择刚刚保存的 VPN 连接，连接成功后，即可访问 VPN 网络了。