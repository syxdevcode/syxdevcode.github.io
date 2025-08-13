---
title: CentOS7/Red-Hat密码更换周期和复杂度设置
date: 2020-12-31 11:15:17
tags:
- Linux
- CentOS7
- Red Hat
- 安全策略
- Linux基础
categories: Linux基础
---

## 密码更换周期设置

密码定期更换配置在 `/etc/login.defs` 中:

```sh
PASS_MAX_DAYS	90   # 密码到期时间
PASS_MIN_DAYS	6    # 初始密码更改时间
PASS_MIN_LEN	8    # 密码最小长度
PASS_WARN_AGE	10   # 密码过期提示时间
```

**注意：**

在 `/etc/login.defs` 中设置仅影响创建用户，而不会影响现有用户。
如果设置为现有用户，使用命令 `chage -M（days）（user）`，例如：`chage -M 60 root`。

**chage命令**

chage 命令是用来修改帐号和密码的有效期限。

语法

chage [选项] 用户名

选项

* -m：密码可更改的最小天数。为零时代表任何时候都可以更改密码。
* -M：密码保持有效的最大天数。
* -w：用户密码到期前，提前收到警告信息的天数。
* -E：帐号到期的日期。过了这天，此帐号将不可用。
* -d：上一次更改的日期。
* -i：停滞时期。如果一个密码已过期这些天，那么此帐号将不可用。
* -l：例出当前的设置。由非特权用户来确定他们的密码或帐号何时过期。

示例

```sh
# 查看root配置
chage -l root

# 设置root密码90天过期
chage -M 90 root
```

## 密码复杂度设置

CentOS7/RHEL7 开始使用 pam_pwquality 模块进行密码复杂度策略的控制管理。pam_pwquality 替换了原来 Centos6/RHEL6 中的 pam_cracklib 模块，并向后兼容。

### pam_pwquality模块设置

retry=N  允许重试N次
difok=N 新密码必需与旧密码不同的位数  difok=3 新密码必须与旧密码有3位不同
minlen=N  最小位数
ucredit=N  大写字母位数
lcredit=N  小写字母位数
dcredit=N  数字个数
ocredit=N  特殊字母的个数

两种方式实现:

* 1，直接指定 pam_pwquality 模块参数，在 `/etc/pam.d/system-auth` 中的 `password    requisite     pam_pwquality.so` 行尾添加具体参数，比如 `minlen=16 ucredit=-1 lcredit=-1 ocredit=-1 dcredit=-1` ，表示最小密码长度16位，数字，大小写字母，特殊字符均至少包含1位。

N >= 0:密码中最多有多少个;
N < 0 :密码中最少有多少个;

```sh
password    requisite     pam_pwquality.so try_first_pass local_users_only retry=3 authtok_type= minlen=8 ucredit=-1 lcredit=-1 ocredit=-1 dcredit=-1
```

* 2，通过修改 `/etc/security/pwquality.conf` 参数文件，定义密码复杂度规则。

修改 `pwquality.conf` 参数文件有2种方法。

（1）直接修改 `/etc/security/pwquality.conf` 。
（2）使用 `authconfig` 命令修改，修改后最终会体现在 `/etc/security/pwquality.conf`。

```sh
# 最小长度
authconfig --passminlen=8 --update
grep "^minlen" /etc/security/pwquality.conf

# 设置同一类的允许连续字符的最大数目
authconfig --passmaxclassrepeat=6 --update
grep "^maxclassrepeat" /etc/security/pwquality.conf

# 至少需要一个小写字符
authconfig --enablereqlower --update
grep "^lcredit" /etc/security/pwquality.conf

# 至少需要一个大写字符
authconfig --enablerequpper --update
grep "^ucredit" /etc/security/pwquality.conf

# 至少需要一个数字
authconfig --enablereqdigit --update
grep "^dcredit" /etc/security/pwquality.conf

# 至少一个特殊字符
authconfig --enablereqother --update
grep "^ocredit" /etc/security/pwquality.conf

# 以下需要单独设置
vim /etc/security/pwquality.conf

# 添加一下配置

# 设置单调字符序列的最大长度
maxsequence = 5

# 新密码中不能出现在旧密码中的字符数
difok = 5

# 检查来自用户 passwd 条目的 GECOS 字段的长度超过3个字符的字是否包含在新密码中。
gecoscheck = 1

# 设置不能包含在密码中的Ssace分隔的单词列表
badwords = test test1 test2
```

示例：

```sh
vim /etc/security/pwquality.conf
grep -vE "^#|^$" /etc/security/pwquality.conf
```

```sh
minlen = 8

# 新密码所需的字符类（数字、大写、小写、其他）的最小字符数。
minclass = 1

# 新密码中允许的连续相同字符的最大数量。如果值为 0，则禁用检查。
maxrepeat = 0

maxclassrepeat = 6
lcredit       = -1
ucredit       = -1
dcredit       = -1
ocredit       = -1
```

### Centos7 继续使用 pam_cracklib 模块检验密码复杂度

vim /etc/pam.d/system-auth

```sh
# 添加到 pam_pwquality.so 所在行的上行
password    requisite     pam_cracklib.so try_first_pass retry=3 type=  minlen=8 ucredit=-1 lcredit=-1 ocredit=-1 dcredit=-1
```

### Read Hat配置密码复杂度

vim /etc/pam.d/system-auth

```sh
password   requisite      pam_cracklib.so retry=3 difok=3 minlen=8 ucredit=-1 lcredit=-1 dcredit=-1 ocredit=-1
```

参考：

[CentOS7上密码复杂度设置](https://www.jianshu.com/p/f7bd6dfffc34)

[CentOS7密码复杂度配置](https://blog.csdn.net/longfeizzu/article/details/101377584)

[chage命令](https://man.linuxde.net/chage)

[CentOS 7 设置密码规则](https://blog.csdn.net/wh211212/article/details/53992772)