---
title: Docker修改时区
date: 2018-05-25 09:31:59
tags:
- Docker
categories: 
- Docker
---
# Docker修改时区

## 修改Dockerfile

** CentOS7**

```docker
RUN apk add --update tzdata
ENV TZ=Asia/Shanghai
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone
```

```docker
#update system timezone
#RUN cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
RUN ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime

#update application timezone
RUN echo "Asia/Shanghai" >> /etc/timezone
```