# 微服务日志收集

微服务体系由于服务数量比较多， 当需要查看日志时, 一个个的去查看似乎不太现实。于是便出现了elasticsearch，这种日志收集服务

## 原理
1. 服务端输出日志到文件
2. fluentd 根据tail解析日志文件， 发送到elesticsearch

### 实现
附上一个ansible-playbook, 需要指定目标机器，和配置文件


``` yaml

- hosts: "{{ host }}"
  tasks:
    - name: Install fluentd
      get_url: url=https://toolbelt.treasuredata.com/sh/install-redhat-td-agent2.sh dest=/root/fluentd.sh

    - name: chmod sh
      file:
        path: /root/fluentd.sh
        owner: root
        group: root
        mode: 01777

    - name: exec
      shell: /root/fluentd.sh

    - name: clean
      file:
        state: absent
        path: /root/fluentd.sh

    - lineinfile:
        path: /etc/init.d/td-agent
        regexp: '^TD_AGENT_USER='
        line: 'TD_AGENT_USER=root'

    - lineinfile:
        path: /etc/init.d/td-agent
        regexp: '^TD_AGENT_GROUP='
        line: 'TD_AGENT_GROUP=root'

    - name: touch pos file
      file:
        path: /etc/td-agent/data.pos
        state: touch

    - name: Install deps
      shell: /usr/sbin/td-agent-gem install fluent-plugin-elasticsearch

    - name: cp conf
      copy:
        src:  "{{ conf }}"
        dest: /etc/td-agent/td-agent.conf

    - name: start service
      shell: /etc/init.d/td-agent restart
```

```
#example.conf


<source>
    @type tail
    format json
    time_key time
    time_format %Y-%m-%dT%H:%M:%S,%N
    keep_time_key true
    path /opt/bigdata/druid/var/sv/historical/current
    tag historical.log
    pos_file /etc/td-agent/data.pos
</source>
<match historical.*>
    @type elasticsearch
    host buguelasticsearch.myun.tv
    port 9200
    include_tag_key true
    tag_key @level
    logstash_format true
    logstash_prefix druid.historical
    flush_interval 3s
</match>
```

