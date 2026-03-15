# cs/msf流量特征

## cs

http流量特征（未魔改）：

- 被控端会发送心跳包；

- 在执行下发指令时，会看到木马攻击者访问c2服务器上面的submit.php界面，并且在访问界面会带有一个id参数；

- cs4.0版本的ua头时固定的，4.5及以上版本随机的，能避免被蓝队检测到；

- url路径；解密算法checksum8（92L，93L）；

  > /4Ekx

  使用工具解密cs心跳包，Didier Stevens

## msf

shell模式 明文传输,特征：从数据包明文中判断命令执行

meterpreter模式 tcp 加密传输，生成两个后门进行[共同点](https://so.csdn.net/so/search?q=共同点&spm=1001.2101.3001.7020)对比，但是这种非常麻烦，因为x32、x64、正反向木马的数据包都不一样，需要一直测试比对。

meterpreter模式 HTTP，特征：固定的数据包请求和返回模版（格式一致），request和response返回的数据包格式一致

meterpreter模式 HTTPS，特征：JA3/JA3S值

4d93395b1c1b9ad28122fb4d09f28c5e 652358a663590cfc624787f06b82d9ae

15af977ce25de452b96affa2addb1036 2253c82f03b621c5144709b393fde2c9
