---
layout: post
title: lsyncd
date: 2019-06-19 15:40:00
tags:
    - [ lsyncd ]
categories:
    - [ services ]
password: cbb4693
---
## rsync
```
选项详解:
-a,--archive	递归传输文件,保持所有属性=rlptgoD
-z,--compress	压缩传递
-r,--recursive	递归传送文件
--progress	显示备份过程
-v,verbose	详细模式
--exclude=pattern	排除
--include=pattern	包含
-delete	    源文件不存在了,就会把目标删除
-P     显示同步进度
-i     找出文件间的不同

#本地同步/root下所有文件到/tmp/root
rsync -avz /root /tmp/root

#ssh本地同步到远程
rsync  -avzP -e 'ssh -p 2222' root@xx.xx.xx.xx:/docker/   #同步数据并指定进度

#ssh远程同步到本地
rsync -avz -delete root@192.168.1.3:/pro/* /tmp/pro/
```
---
## lsyncd config 
```
settings {
 logfile = "/xx/server/lsyncd/var/lsyncd.log",
 statusFile = "/xx/server/lsyncd/var/lsyncd.status",
 inotifyMode = "CloseWrite or Modify",
    maxProcesses = 8,
}
 
-- test_4: 		xx.xx.xx.xx

servers = {
	"xx.xx.xx.xx",
}

servers_usr = {
	"xx.xx.xx.xx",
}

servers_for_kdhelp = {
	"xx.xx.xx.xx",
}

--kdhelp
for _, server in ipairs(servers_for_kdhelp) do
sync {
 default.rsync,
 source = "/xx/www/kdhelp",
 target = server..":/xx/www/kdhelp",
 delay = 0,
 rsync ={
  binary ="/usr/bin/rsync",
  archive =true,
  compress =true,
  verbose = true,
  rsh = "/usr/bin/ssh -p 2020 -o StrictHostKeyChecking=no"
 },
}
end

--kuaidihelp_addresslib
for _, server in ipairs(servers) do
sync {
 default.rsync,
 source = "/xx/www/kuaidihelp_addresslib",
 target = server..":/xx/www/kuaidihelp_addresslib",
 delay = 0,
 rsync ={
  binary ="/usr/bin/rsync",
  archive =true,
  compress =true,
  verbose = true,
  rsh = "/usr/bin/ssh -p 2020 -o StrictHostKeyChecking=no"
 },
}
end

--/usr/addr
--for _, server in ipairs(servers_usr) do
--sync {
-- default.rsync,
-- source = "/usr/addr",
-- target = server..":/usr/addr",
-- delay = 0,
-- rsync ={
--  binary ="/usr/bin/rsync",
--  archive =true,
--  compress =true,
--  verbose = true,
--  rsh = "/usr/bin/ssh -p 2020 -o StrictHostKeyChecking=no"
-- },
--}
--end
--
----/usr/addrlib
for _, server in ipairs(servers_usr) do
sync {
 default.rsync,
 source = "/usr/addrlib",
 target = server..":/usr/addrlib",
 delay = 0,
 rsync ={
  binary ="/usr/bin/rsync",
  archive =true,
  compress =true,
  verbose = true,
  rsh = "/usr/bin/ssh -p 2020 -o StrictHostKeyChecking=no"
 },
}
end
```
```
settings {
 logfile = "/xx/server/lsyncd/var/lsyncd.log",
 statusFile = "/xx/server/lsyncd/var/lsyncd.status",
 inotifyMode = "CloseWrite or Modify",
    maxProcesses = 8,
}

-- web_3: xx.xx.xx.xx
-- RC_server: xx.xx.xx.xx

servers_for_code = {
	"xx.xx.xx.xx",
    "xx.xx.xx.xx",
}

servers_for_conf = {
    "xx.xx.xx.xx",  
}

--kdhelp
for _, server in ipairs(servers_for_code) do
sync {
 default.rsync,
 source = "/xx/www/kdhelp",
 target = server..":/xx/www/kdhelp",
 delay = 0,
 rsync ={
  binary ="/usr/bin/rsync",
  archive =true,
  compress =true,
  verbose = true,
  rsh = "/usr/bin/ssh -p 2020 -o StrictHostKeyChecking=no"
 },
}
end


--kuaidihelp
for _, server in ipairs(servers_for_code) do 
sync {
 default.rsync,
 source = "/xx/www/kuaidihelp",
 target = server..":/xx/www/kuaidihelp",
--excludeFrom = "/xx/www/kuaidihelp/web/www/view_c/*.php",
 delay = 0,
 rsync ={
  binary ="/usr/bin/rsync",
  archive =true,
  compress =true,
  verbose = true,
  rsh = "/usr/bin/ssh -p 2020 -o StrictHostKeyChecking=no"
 },
}
end


--kuaidihelp_admin
for _, server in ipairs(servers_for_code) do 
sync {
 default.rsync,
 source = "/xx/www/kuaidihelp_admin",
 target = server..":/xx/www/kuaidihelp_admin",
 delay = 0,
 rsync ={
  binary ="/usr/bin/rsync",
  archive =true,
  compress =true,
  verbose = true,
  rsh = "/usr/bin/ssh -p 2020 -o StrictHostKeyChecking=no"
 },
}
end


--kuaidihelp_app
for _, server in ipairs(servers_for_code) do 
sync {
 default.rsync,
 source = "/xx/www/kuaidihelp_app",
 target = server..":/xx/www/kuaidihelp_app",
 delay = 0,
 rsync ={
  binary ="/usr/bin/rsync",
  archive =true,
  compress =true,
  verbose = true,
  rsh = "/usr/bin/ssh -p 2020 -o StrictHostKeyChecking=no"
 },
}
end



--kuaidihelp_ckd
for _, server in ipairs(servers_for_code) do 
sync {
 default.rsync,
 source = "/xx/www/kuaidihelp_ckd",
 target = server..":/xx/www/kuaidihelp_ckd",
 delay = 0,
 rsync ={
  binary ="/usr/bin/rsync",
  archive =true,
  compress =true,
  verbose = true,
  rsh = "/usr/bin/ssh -p 2020 -o StrictHostKeyChecking=no"
 },
}
end


--kuaidihelp_kdy
for _, server in ipairs(servers_for_code) do 
sync {
 default.rsync,
 source = "/xx/www/kuaidihelp_kdy",
 target = server..":/xx/www/kuaidihelp_kdy",
 delay = 0,
 rsync ={
  binary ="/usr/bin/rsync",
  archive =true,
  compress =true,
  verbose = true,
  rsh = "/usr/bin/ssh -p 2020 -o StrictHostKeyChecking=no"
 },
}
end


--kuaidihelp_monitor
for _, server in ipairs(servers_for_code) do 
sync {
 default.rsync,
 source = "/xx/www/kuaidihelp_monitor",
 target = server..":/xx/www/kuaidihelp_monitor",
 delay = 0,
 rsync ={
  binary ="/usr/bin/rsync",
  archive =true,
  compress =true,
  verbose = true,
  rsh = "/usr/bin/ssh -p 2020 -o StrictHostKeyChecking=no"
 },
}
end


--kuaidihelp_operate
for _, server in ipairs(servers_for_code) do 
sync {
 default.rsync,
 source = "/xx/www/kuaidihelp_operate",
 target = server..":/xx/www/kuaidihelp_operate",
 delay = 0,
 rsync ={
  binary ="/usr/bin/rsync",
  archive =true,
  compress =true,
  verbose = true,
  rsh = "/usr/bin/ssh -p 2020 -o StrictHostKeyChecking=no"
 },
}
end


--kuaidihelp_statics
for _, server in ipairs(servers_for_code) do 
sync {
 default.rsync,
 source = "/xx/www/kuaidihelp_statics",
 target = server..":/xx/www/kuaidihelp_statics",
 delay = 0,
 rsync ={
  binary ="/usr/bin/rsync",
  archive =true,
  compress =true,
  verbose = true,
  rsh = "/usr/bin/ssh -p 2020 -o StrictHostKeyChecking=no"
 },
}
end


--kuaidihelp_verification
for _, server in ipairs(servers_for_code) do 
sync {
 default.rsync,
 source = "/xx/www/kuaidihelp_verification",
 target = server..":/xx/www/kuaidihelp_verification",
 delay = 0,
 rsync ={
  binary ="/usr/bin/rsync",
  archive =true,
  compress =true,
  verbose = true,
  rsh = "/usr/bin/ssh -p 2020 -o StrictHostKeyChecking=no"
 },
}
end


--kuaidihelp_wdadmin
for _, server in ipairs(servers_for_code) do 
sync {
 default.rsync,
 source = "/xx/www/kuaidihelp_wdadmin",
 target = server..":/xx/www/kuaidihelp_wdadmin",
 delay = 0,
 rsync ={
  binary ="/usr/bin/rsync",
  archive =true,
  compress =true,
  verbose = true,
  rsh = "/usr/bin/ssh -p 2020 -o StrictHostKeyChecking=no"
 },
}
end


--kuaidihelp_sms
for _, server in ipairs(servers_for_code) do
sync {
 default.rsync,
 source = "/xx/www/kuaidihelp_sms",
 target = server..":/xx/www/kuaidihelp_sms",
 delay = 0,
 rsync ={
  binary ="/usr/bin/rsync",
  archive =true,
  compress =true,
  verbose = true,
  rsh = "/usr/bin/ssh -p 2020 -o StrictHostKeyChecking=no"
 },
}
end

--kuaidihelp_city
for _, server in ipairs(servers_for_code) do
sync {
 default.rsync,
 source = "/xx/www/kuaidihelp_city",
 target = server..":/xx/www/kuaidihelp_city",
 delay = 0,
 rsync ={
  binary ="/usr/bin/rsync",
  archive =true,
  compress =true,
  verbose = true,
  rsh = "/usr/bin/ssh -p 2020 -o StrictHostKeyChecking=no"
 },
}
end

--kuaidihelp_tbk
for _, server in ipairs(servers_for_code) do
sync {
 default.rsync,
 source = "/xx/www/kuaidihelp_tbk",
 target = server..":/xx/www/kuaidihelp_tbk",
 delay = 0,
 rsync ={
  binary ="/usr/bin/rsync",
  archive =true,
  compress =true,
  verbose = true,
  rsh = "/usr/bin/ssh -p 2020 -o StrictHostKeyChecking=no"
 },
}
end

--nginx/conf
for _, server in ipairs(servers_for_conf) do
sync {
 default.rsync,
 source = "/xx/server/openresty/nginx/conf",
 target = server..":/xx/server/openresty/nginx/conf",
 delay = 0,
 rsync ={
  binary ="/usr/bin/rsync",
  archive =true,
  compress =true,
  verbose = true,
  rsh = "/usr/bin/ssh -p 2020 -o StrictHostKeyChecking=no"
 },
}
end

--php/conf
for _, server in ipairs(servers_for_conf) do
sync {
 default.rsync,
 source = "/xx/server/php7/etc",
 target = server..":/xx/server/php7/etc",
 delay = 0,
 rsync ={
  binary ="/usr/bin/rsync",
  archive =true,
  compress =true,
  verbose = true,
  rsh = "/usr/bin/ssh -p 2020 -o StrictHostKeyChecking=no"
 },
}
end
```
