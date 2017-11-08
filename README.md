# ansible-sensu

# Setup

Install `cmacrae.sensu` from ansible-galaxy if needed.

```
$ ansible-galaxy install cmacrae.sensu
```

# Edit config

Edit `hosts`

Example
```
[rabbitmq_servers]
mq.cmacrae.sensu.com

[redis_servers]
redis.cmacrae.sensu.com

[sensu_masters]
cmacrae.sensu.com

[zones]
client.domain.com
```

@see `site.yml` , `zones.yml` and check `ssh login-user`

## Setup some parameters

@see `group_vars/sensu_masters.yml`

You may need to set these parameters.

- `sensu_api_host`
- `slack_webhook_url`
- `slack_channel`
- `handler_mail_from`
- `handler_mail_to`
- `contacts_infra_email_to`

# Install sensu_server, graphite and grafana
```
ansible-playbook -v -i hosts site.yml --private-key="~/.ssh/priv-key.pem"
```

- open uchiwa port 3000
- open api port 4567
- open mq port 5671

# Install sensu_client
```
ansible-playbook -v -i hosts zones.yml --private-key="~/.ssh/priv-key.pem"
```

# Browse

Uchiwa Dashboard

`http://[ip address]:3000/`

Grafana Dashboard

`https://[ip address]/`

## Play Grafana

- Access to Grafana
    - Launch a web browser
    - Access to url: ```https://xxx.xxx.xxx.xxx/``` (Your public IP address)
    - Accept untrusted certification
    - Log in with username=`admin`, password=`admin`
- Click the top-left menu icon
- Data Sources -> Add new
    - Name: `Graphite`
    - Default: On
    - Type: Graphite
    - Url: `http://localhost:9001`
    - Access: proxy
    - Basic Auth: Enable: Off
- Dashboards -> Home -> New
- Row menu (gree button) -> Add Panel -> Graph -> Click on the title -> edit
- Select the metrics (e.g. `carbon.agents.ip-xxx-xxx-xxx-xxx-a.memUsage`)

# Plugins

@see `group_vars/sensu_masters.yml`
@see `group_vars/zones.yml`
@see `data/static/sensu/definitions/*`


# Bug Fix

Ansible-sensu uses `cmacrae.sensu` from ansible-galaxy.
If you want to run this script under the AmazonLinux, you should fix some points.

```
diff --git a/tasks/Amazon/main.yml b/tasks/Amazon/main.yml
index 7e3857b..42db7ab 100644
--- a/tasks/Amazon/main.yml
+++ b/tasks/Amazon/main.yml
@@ -10,7 +10,7 @@
       content: |
         [sensu]
         name=sensu
-        baseurl=https://sensu.global.ssl.fastly.net/yum/$releasevar/$basearch/
+        baseurl=https://sensu.global.ssl.fastly.net/yum/6/$basearch/
         gpgcheck=0
         enabled=1
       owner: root
diff --git a/tasks/ssl_generate.yml b/tasks/ssl_generate.yml
index 80e8892..57d5f7c 100644
--- a/tasks/ssl_generate.yml
+++ b/tasks/ssl_generate.yml
@@ -23,7 +23,7 @@
         copy: no

     - name: Generate SSL certs
-      command: "_ {{ sensu_config_path }}/ssl_generation/sensu_ssl_tool/ssl_certs.sh generate"
+      command: "{{ sensu_config_path }}/ssl_generation/sensu_ssl_tool/ssl_certs.sh generate"
       args:
         chdir: "{{ sensu_config_path }}/ssl_generation/sensu_ssl_tool"
         creates: "{{ sensu_config_path }}/ssl_generation/sensu_ssl_tool/server"
```
