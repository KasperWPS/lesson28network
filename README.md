# Домашнее задание № 18 по теме: "Архитектура сетей". К курсу Administrator Linux. Professional

## Задание

### Теоретическая часть

- Найти свободные подсети
- Посчитать количество узлов в каждой подсети, включая свободные
- Указать Broadcast-адрес для каждой подсети
- Проверить, нет ли ошибок при разбиении


### Практическая часть

- Соединить офисы в сеть согласно схеме и настроить роутинг
- Интернет-трафик со всех серверов должен ходить через inetRouter
- Все сервера должны видеть друг друга (должен проходить ping)
- У всех новых серверов отключить дефолт на NAT (eth0), который vagrant поднимает для связи
- Добавить дополнительные сетевые интерфейсы, если потребуется


## Выполнение

- ОС: CentOS/8
- Vagrant 2.4.0
- Ansible 2.16.4

### Теоретическая часть

![Network topology](https://github.com/KasperWPS/lesson28network/blob/main/topology.svg)

Составить список используемых сетей:

- router-net
  - 192.168.255.0/30
    - mask: 255.255.255.252
    - min: 192.168.255.1
    - max: 192.168.255.2
    - broadcast: 192.168.255.3
    - Использовано (2/2):
      - **InetRouter** (192.168.255.1);
      - **CentralRouter** (192.168.255.2)

- dir-net
  - 192.168.0.0/28
    - mask: 255.255.255.240
    - min: 192.168.0.1
    - max: 192.168.0.14
    - broadcast: 192.168.0.15
    - Использовано (2/14):
      - **centralRouter** (192.168.0.1);
      - **CentralServer** (192.168.0.2)

- hw-net
  - 192.168.0.32/28
    - mask: 255.255.255.240
    - min: 192.168.0.33
    - max: 192.168.0.46
    - broadcast: 192.168.0.47
    - Использовано (1/14):
      - **CentralRouter** (192.168.0.33)

- mgt-net
  - 192.168.0.64/26
    - mask: 255.255.255.192
    - min: 192.168.0.65
    - max: 192.168.0.126
    - broadcast: 192.168.0.127
    - Использовано (1/62):
      - **CentralRouter** (192.168.0.65)

- office1-central
  - 192.168.255.8/30
    - mask: 255.255.255.252
    - min: 192.168.255.9
    - max: 192.168.255.10
    - broadcast: 192.168.255.11
    - Использовано (2/2):
      - **centralRouter** (192.168.255.9);
      - **office1Router** (192.168.255.10)

- office2-central
  - 192.168.255.4/30
    - mask: 255.255.255.252
    - min: 192.168.255.5
    - max: 192.168.255.6
    - broadcast: 192.168.255.7
    - Использовано (2/2):
      - **centralRouter** (192.168.255.5);
      - **office2Router** (192.168.255.6)

- dev1-net
  - 192.168.2.0/26
    - mask: 255.255.255.192
    - min: 192.168.2.1
    - max: 192.168.2.62
    - broadcast: 192.168.2.63
    - Использовано (1/62):
      - **office1Router** (192.168.2.1)

- test1-net
  - 192.168.2.64/26
    - mask: 255.255.2.192
    - min: 192.168.2.65
    - max: 192.168.2.126
    - broadcast: 192.168.2.127
    - Использовано (1/62):
      - **office1Router** (192.168.2.65)

- managers-net
  - 192.168.2.128/26
    - mask: 255.255.255.192
    - min: 192.168.2.129
    - max: 192.168.2.190
    - broadcast: 192.168.2.191
    - Использовано (2/62):
      - **office1Router** (192.168.2.129);
      - **office1Server** (192.168.2.130)

- office1-net
  - 192.168.2.192/26
    - mask: 255.255.255.292
    - min: 192.168.2.193
    - max: 192.168.2.254
    - broadcast: 192.168.2.255
    - Использовано (1/62):
      - **office1Router** (192.168.2.193)

- dev2-net
  - 192.168.1.0/25
    - mask: 255.255.255.128
    - min: 192.168.1.1
    - max: 192.168.1.126
    - broadcast: 192.168.1.127
    - Использовано (2/126):
      - **office2Router** (192.168.1.1);
      - **office2Server** (192.168.1.2)

- test2-net
  - 192.168.1.128/26
    - mask: 255.255.255.192
    - min: 192.168.1.129
    - max: 192.168.1.190
    - broadcast: 192.168.1.191
    - Использовано (1/62):
      - **office2Router** (192.168.1.129)

- office2-net
  - 192.168.1.192/26
    - mask: 255.255.255.192
    - min: 192.168.1.193
    - max: 192.168.1.254
    - broadcast: 192.168.1.255
    - Использовано (1/62):
      - **office2Router** (192.168.1.193)

*Ошибок в задании нет, но в методических рекомендациях, в таблице подсетей, отсутствуют две подсети, в связи с чем скорректирован список свободных адресов ниже:*
- 192.168.255.8/30
- 192.168.255.4/30

### Свободные адресные пространства в использованных подсетях:
- 192.168.255.218/25
- 192.168.255.64/26
- 192.168.255.32/27
- 192.168.255.16/28
- 192.168.255.12/30
- 192.168.0.16/28
- 192.168.0.48/28
- 192.168.0.128/25


Схема сети:

![Network topology](https://github.com/KasperWPS/lesson28network/blob/main/topology2.svg)

## Практическая часть

Конфигурация сетевой лаборатории (**config.json**):

```json
[
  {
    "name": "inetRouter",
    "cpus": 1,
    "gui": false,
    "box": "centos/8",
    "private_network":
    [
      { "ip": "192.168.255.1", "adapter": 2, "netmask": "255.255.255.252", "virtualbox__intnet": "router-net" },
      { "ip": "192.168.56.10", "adapter": 3, "netmask": "255.255.255.0" }
    ],
    "memory": "640",
    "no_share": true
  },
  {
    "name": "centralRouter",
    "cpus": 1,
    "gui": false,
    "box": "centos/8",
    "private_network":
    [
      { "ip": "192.168.255.2", "adapter": 2, "netmask": "255.255.255.252", "virtualbox__intnet": "router-net"      },
      { "ip": "192.168.0.1",   "adapter": 3, "netmask": "255.255.255.240", "virtualbox__intnet": "dir-net"         },
      { "ip": "192.168.0.33",  "adapter": 4, "netmask": "255.255.255.240", "virtualbox__intnet": "hw-net"          },
      { "ip": "192.168.0.65",  "adapter": 5, "netmask": "255.255.255.192", "virtualbox__intnet": "mgt-net"         },
      { "ip": "192.168.255.9", "adapter": 6, "netmask": "255.255.255.252", "virtualbox__intnet": "office1-central" },
      { "ip": "192.168.255.5", "adapter": 7, "netmask": "255.255.255.252", "virtualbox__intnet": "office2-central" },
      { "ip": "192.168.56.11", "adapter": 8, "netmask": "255.255.255.0" }
    ],
    "memory": 640,
    "no_share": true
  },
  {
    "name": "centralServer",
    "cpus": 1,
    "gui": false,
    "box": "centos/8",
    "private_network":
    [
      { "ip": "192.168.0.2",   "adapter": 2, "netmask": "255.255.255.240", "virtualbox__intnet": "dir-net" },
      { "ip": "192.168.56.12", "adapter": 3, "netmask": "255.255.255.0" }
    ],
    "memory": "640",
    "no_share": true
  },
  {
    "name": "office1Router",
    "cpus": 1,
    "gui": false,
    "box": "centos/8",
    "private_network":
    [
      { "ip": "192.168.255.10", "adapter": 2, "netmask": "255.255.255.252", "virtualbox__intnet": "office1-central" },
      { "ip": "192.168.2.1",    "adapter": 3, "netmask": "255.255.255.192", "virtualbox__intnet": "dev1-net"        },
      { "ip": "192.168.2.65",   "adapter": 4, "netmask": "255.255.255.192", "virtualbox__intnet": "test1-net"       },
      { "ip": "192.168.2.129",  "adapter": 5, "netmask": "255.255.255.192", "virtualbox__intnet": "managers-net"    },
      { "ip": "192.168.2.193",  "adapter": 6, "netmask": "255.255.255.192", "virtualbox__intnet": "office1-net"     },
      { "ip": "192.168.56.20",  "adapter": 7, "netmask": "255.255.255.0" }
    ],
    "memory": 640,
    "no_share": true
  },
  {
    "name": "office1Server",
    "cpus": 1,
    "gui": false,
    "box": "centos/8",
    "private_network":
    [
      { "ip": "192.168.2.130", "adapter": 2, "netmask": "255.255.255.192", "virtualbox__intnet": "managers-net" },
      { "ip": "192.168.56.21", "adapter": 3, "netmask": "255.255.255.0" }
    ],
    "memory": "640",
    "no_share": true
  },
  {
    "name": "office2Router",
    "cpus": 1,
    "gui": false,
    "box": "centos/8",
    "private_network":
    [
      { "ip": "192.168.255.6", "adapter": 2, "netmask": "255.255.255.252", "virtualbox__intnet": "office2-central" },
      { "ip": "192.168.1.1",   "adapter": 3, "netmask": "255.255.255.128", "virtualbox__intnet": "dev2-net"        },
      { "ip": "192.168.1.129", "adapter": 4, "netmask": "255.255.255.192", "virtualbox__intnet": "test2-net"       },
      { "ip": "192.168.1.193", "adapter": 5, "netmask": "255.255.255.192", "virtualbox__intnet": "office2-net"     },
      { "ip": "192.168.56.30", "adapter": 6, "netmask": "255.255.255.0" }
    ],
    "memory": 640,
    "no_share": true
  },
  {
    "name": "office2Server",
    "cpus": 1,
    "gui": false,
    "box": "centos/8",
    "private_network":
    [
      { "ip": "192.168.1.2",   "adapter": 2, "netmask": "255.255.255.128", "virtualbox__intnet": "dev2-net" },
      { "ip": "192.168.56.31", "adapter": 3, "netmask": "255.255.255.0" }
    ],
    "memory": "640",
    "no_share": true
  }
]
```

Vagrantfile:

```ruby
# -*- mode: ruby -*- 
# vi: set ft=ruby : vsa
Vagrant.require_version ">= 2.2.17"

# networ ожидает хэш, поэтому преобразуем строки в ключи хэша
class Hash
  def rekey
  t = self.dup
  self.clear
  t.each_pair{|k, v| self[k.to_sym] = v}
    self
  end
end

require 'json'

f = JSON.parse(File.read(File.join(File.dirname(__FILE__), 'config.json')))
# Локальная переменная PATH_SRC для монтирования
$PathSrc = ENV['PATH_SRC'] || "."

Vagrant.configure(2) do |config|
  if Vagrant.has_plugin?("vagrant-vbguest")
    config.vbguest.auto_update = false
  end

  # включить переадресацию агента ssh
  config.ssh.forward_agent = true
  # использовать стандартный для vagrant ключ ssh
  config.ssh.insert_key = false

  last_vm = f[(f.length)-1]['name']

  f.each do |g|

    config.vm.define g['name'] do |s|
      s.vm.box = g['box']
      s.vm.hostname = g['name']

      if g['private_network']
        g['private_network'].each do |ni|
          s.vm.network "private_network", **ni.rekey
        end
      end

      s.vm.synced_folder $PathSrc, "/vagrant", disabled: g['no_share']

      s.vm.provider :virtualbox do |virtualbox|
        virtualbox.customize [
          "modifyvm",             :id,
          "--audio",              "none",
          "--cpus",               g['cpus'],
          "--memory",             g['memory'],
          "--graphicscontroller", "VMSVGA",
          "--vram",               "64"
        ]

        attachController = false

        if g['disks']
          g['disks'].each do |dname, dconf|
            unless File.exist? (dconf['dfile'])
              attachController = true
              virtualbox.customize [
                'createhd',
                '--filename', dconf['dfile'],
                '--variant',  'Fixed',
                '--size',     dconf['size']
              ]
            end
          end
          if attachController == true
            virtualbox.customize [
              "storagectl", :id,
              "--name",     "SAS Controller",
              "--add",      "sas"
            ]
          end
          g['disks'].each do |dname, dconf|
            virtualbox.customize [
              'storageattach', :id,
              '--storagectl',  'SAS Controller',
              '--port',        dconf['port'],
              '--device',      0,
              '--type',        'hdd',
              '--medium',      dconf['dfile']
            ]
          end
        end
        virtualbox.gui = g['gui']
        virtualbox.name = g['name']
      end
      if g['name'] == last_vm
        s.vm.provision "ansible" do |ansible|
          ansible.playbook = "provisioning/playbook.yml"
          ansible.inventory_path = "provisioning/hosts"
          ansible.host_key_checking = "false"
          ansible.become = "true"
          ansible.limit = "all"
        end
      end
    end
  end
end
```

### Настройка NAT.

Вместо рекомендованного iptables (netfilter) применены nftables, т.к. в данной версии ОС заложен прозрачный переход на новую технологию, убедится в этом можно
```bash
vagrant ssh centralServer -c 'iptables -V'
```
```
iptables v1.8.4 (nf_tables)
```

nftables.conf (inetRouter):

```
table inet filter {
        chain input {
                type filter hook input priority filter; policy drop;
                ct state established,related accept
                iif "lo" accept
                ct state new tcp dport 22 accept
                icmp type echo-request accept
        }
}
table ip nat {
        chain postrouting {
                type nat hook postrouting priority srcnat; policy accept;
                ip daddr != 192.168.0.0/16 iif "eth1" oif "eth0" masquerade
        }
}

```

### Настройка маршрутов

Произведена с использованием nmcli и модулем ansible community.general.nmcli - этот модуль работает только с объектом connection, для применения внесённых изменений в конфигурацию сетевого соединения необходимо выполнять команду
```bash
nmcli connection up 'System eth0'
```
для этого написан handler, он исполняется только при внесении изменений, в связи с чем принципы идемпотентности сохранены. В данной части задания совершён значительный отступ от методических рекомендаций, однако, стоит принять во внимание, прежде всего, то, что данная технология перспективна и универсальна - данные методы можно применить в других ОС.

### Ansible provisioner

```yaml
---
- hosts: all
  become: true
  gather_facts: true

  tasks:
  - name: Accept login with password from sshd
    ansible.builtin.lineinfile:
      path: /etc/ssh/sshd_config
      regexp: '^PasswordAuthentication no$'
      line: 'PasswordAuthentication yes'
      state: present
    notify:
      - Restart sshd

  - name: Set timezone
    community.general.timezone:
      name: Europe/Moscow

  - name: List all files in directory /etc/yum.repos.d/*.repo
    find:
      paths: "/etc/yum.repos.d/"
      patterns: "*.repo"
    register: repos

  - name: Comment mirrorlist /etc/yum.repos.d/CentOS-*
    ansible.builtin.lineinfile:
      backrefs: true
      path: "{{ item.path }}"
      regexp: '^(mirrorlist=.+)'
      line: '#\1'
    with_items: "{{ repos.files }}"

  - name: Replace baseurl
    ansible.builtin.lineinfile:
      backrefs: true
      path: "{{ item.path }}"
      regexp: '^#baseurl=http:\/\/mirror.centos.org(.+)'
      line: 'baseurl=http://vault.centos.org\1'
    with_items: "{{ repos.files }}"

  - name: set up forward packages across routers
    sysctl:
      name: net.ipv4.conf.all.forwarding
      value: '1'
      state: present
    when: "'routers' in group_names"

  - name: Install epel-release
    ansible.builtin.yum:
      name: epel-release
      state: present

  - name: Install soft
    ansible.builtin.yum:
      name:
        - vim
        - tcpdump
        - traceroute
      state: present

  - name: Disable default route on eth0 interface
    community.general.nmcli:
      conn_name: System eth0
      type: ethernet
      ifname: eth0
      gw4_ignore_auto: true
      state: present
    when: (ansible_hostname != "inetRouter")
    notify:
      - Eth0 connect

  - name: Copy nftables config on inetRouter
    ansible.builtin.copy:
      src: files/inetRouter-nftables.conf
      dest: /etc/nftables.conf
    notify:
      - Configure nftables
    when: (ansible_hostname == "inetRouter")

  - name: Gateway configure on centralServer
    community.general.nmcli:
      conn_name: System eth1
      type: ethernet
      ifname: eth1
      gw4: "192.168.0.1"
      state: present
    notify:
      - Eth1 connect
    when: (ansible_hostname == "centralServer")

  - name: Gateway configure on office2Server
    community.general.nmcli:
      conn_name: System eth1
      type: ethernet
      ifname: eth1
      gw4: "192.168.1.1"
      state: present
    notify:
      - Eth1 connect
    when: (ansible_hostname == "office2Server")

  - name: Gateway configure on office1Server
    community.general.nmcli:
      conn_name: System eth1
      type: ethernet
      ifname: eth1
      gw4: "192.168.2.129"
      state: present
    notify:
      - Eth1 connect
    when: (ansible_hostname == "office1Server")

  - name: Gateway configure on office2Router
    community.general.nmcli:
      conn_name: System eth1
      type: ethernet
      ifname: eth1
      gw4: "192.168.255.5"
      state: present
    notify:
      - Eth1 connect
    when: (ansible_hostname == "office2Router")

  - name: Gateway configure on office1Router
    community.general.nmcli:
      conn_name: System eth1
      type: ethernet
      ifname: eth1
      gw4: "192.168.255.9"
      state: present
    notify:
      - Eth1 connect
    when: (ansible_hostname == "office1Router")

  - name: Gateway configure on centralRouter
    community.general.nmcli:
      conn_name: System eth1
      type: ethernet
      ifname: eth1
      gw4: "192.168.255.1"
      state: present
    notify:
      - Eth1 connect
    when: (ansible_hostname == "centralRouter")

  - name: Add route for 192.168.2.0/24 via 192.168.255.10 on centralRouter
    community.general.nmcli:
      conn_name: System eth5
      type: ethernet
      ifname: eth5
      routes4:
        - 192.168.2.0/24 192.168.255.10
      state: present
    notify:
      - Eth5 connect
    when: (ansible_hostname == "centralRouter")

  - name: Add route for 192.168.1.0/24 via 192.168.255.6 on centralRouter
    community.general.nmcli:
      conn_name: System eth6
      type: ethernet
      ifname: eth6
      routes4:
        - 192.168.1.0/24 192.168.255.6
      state: present
    notify:
      - Eth6 connect
    when: (ansible_hostname == "centralRouter")

  - name: Add routes on inetRouter
    community.general.nmcli:
      conn_name: System eth1
      type: ethernet
      ifname: eth1
      routes4:
        - 192.168.2.0/24 192.168.255.2
        - 192.168.1.0/24 192.168.255.2
        - 192.168.0.0/28 192.168.255.2
        - 192.168.0.32/28 192.168.255.2
        - 192.168.0.64/26 192.168.255.2
      state: present
    notify:
      - Eth1 connect
    when: (ansible_hostname == "inetRouter")

  handlers:

  - name: Eth0 connect
    command: nmcli connection up 'System eth0'

  - name: Eth1 connect
    command: nmcli connection up 'System eth1'

  - name: Eth5 connect
    command: nmcli connection up 'System eth5'

  - name: Eth6 connect
    command: nmcli connection up 'System eth6'

  - name: Restart sshd
    ansible.builtin.service:
      name: sshd
      state: restarted

  - name: Configure nftables
    ansible.builtin.lineinfile:
      path: /etc/sysconfig/nftables.conf
      line: 'include "/etc/nftables.conf"'
      state: present
    notify:
      - Restart nftables service

  - name: Restart nftables service
    ansible.builtin.service:
      name: nftables
      enabled: true
      state: restarted
```

### Inventory

```ini
[routers]
inetRouter ansible_host=192.168.56.10 ansible_user=vagrant ansible_ssh_private_key_file=.vagrant/machines/inetRouter/virtualbox/private_key
centralRouter ansible_host=192.168.56.11 ansible_user=vagrant ansible_ssh_private_key_file=.vagrant/machines/centralRouter/virtualbox/private_key
office1Router ansible_host=192.168.56.20 ansible_user=vagrant ansible_ssh_private_key_file=.vagrant/machines/office1Router/virtualbox/private_key
office2Router ansible_host=192.168.56.30 ansible_user=vagrant ansible_ssh_private_key_file=.vagrant/machines/office2Router/virtualbox/private_key

[servers]
office1Server ansible_host=192.168.56.21 ansible_user=vagrant ansible_ssh_private_key_file=.vagrant/machines/office1Server/virtualbox/private_key
office2Server ansible_host=192.168.56.31 ansible_user=vagrant ansible_ssh_private_key_file=.vagrant/machines/office2Server/virtualbox/private_key
centralServer ansible_host=192.168.56.12 ansible_user=vagrant ansible_ssh_private_key_file=.vagrant/machines/centralServer/virtualbox/private_key
```

### Проверка правильности выполнения

**office2Server**

```bash
vagrant ssh office2Server -c 'ping 8.8.8.8 -c4'
```
```
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=57 time=29.4 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=57 time=31.4 ms
64 bytes from 8.8.8.8: icmp_seq=3 ttl=57 time=31.1 ms
64 bytes from 8.8.8.8: icmp_seq=4 ttl=57 time=31.2 ms

--- 8.8.8.8 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 8ms
rtt min/avg/max/mdev = 29.378/30.768/31.352/0.806 ms
```

```bash
vagrant ssh office2Server -c 'traceroute -m3 8.8.8.8'
```
```
traceroute to 8.8.8.8 (8.8.8.8), 3 hops max, 60 byte packets
 1  _gateway (192.168.1.1)  0.739 ms  0.669 ms  0.579 ms
 2  192.168.255.5 (192.168.255.5)  30.392 ms  30.282 ms  30.175 ms
 3  192.168.255.1 (192.168.255.1)  30.083 ms  0.760 ms  0.694 ms
```

**office1Server**

```bash
vagrant ssh office1Server -c 'ping 8.8.8.8 -c4'
```
```
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=57 time=29.1 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=57 time=31.4 ms
64 bytes from 8.8.8.8: icmp_seq=3 ttl=57 time=31.5 ms
64 bytes from 8.8.8.8: icmp_seq=4 ttl=57 time=31.4 ms
```

```bash
vagrant ssh office1Server -c 'traceroute -m3 8.8.8.8'
```
```
 1  _gateway (192.168.2.129)  0.731 ms  0.661 ms  0.573 ms
 2  192.168.255.9 (192.168.255.9)  28.777 ms  28.642 ms  28.603 ms
 3  192.168.255.1 (192.168.255.1)  28.485 ms  28.428 ms  28.376 ms
```

**centralServer**

```bash
vagrant ssh centralServer -c 'ping 8.8.8.8 -c4'
```
```
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=59 time=27.3 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=59 time=28.5 ms
64 bytes from 8.8.8.8: icmp_seq=3 ttl=59 time=28.5 ms
64 bytes from 8.8.8.8: icmp_seq=4 ttl=59 time=28.4 ms

--- 8.8.8.8 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 8ms
rtt min/avg/max/mdev = 27.260/28.183/28.528/0.560 ms
```

```bash
vagrant ssh centralServer -c 'traceroute -m2 8.8.8.8'
```
```
traceroute to 8.8.8.8 (8.8.8.8), 2 hops max, 60 byte packets
 1  _gateway (192.168.0.1)  0.633 ms  0.567 ms  0.489 ms
 2  192.168.255.1 (192.168.255.1)  0.558 ms  0.491 ms  0.970 ms
```

### Доступность между хостами

```bash
vagrant ssh centralServer -c 'ping 192.168.1.2 -c1' # office2Server
```
```
PING 192.168.1.2 (192.168.1.2) 56(84) bytes of data.
64 bytes from 192.168.1.2: icmp_seq=1 ttl=62 time=0.960 ms

--- 192.168.1.2 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 0.960/0.960/0.960/0.000 ms
```

```bash
vagrant ssh centralServer -c 'ping 192.168.2.130 -c1' # office1Server
```
```
PING 192.168.2.130 (192.168.2.130) 56(84) bytes of data.
64 bytes from 192.168.2.130: icmp_seq=1 ttl=62 time=0.958 ms

--- 192.168.2.130 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 0.958/0.958/0.958/0.000 ms
```

```bash
vagrant ssh office2Server -c 'ping 192.168.2.130 -c1' # office1Server
```
```
PING 192.168.2.130 (192.168.2.130) 56(84) bytes of data.
64 bytes from 192.168.2.130: icmp_seq=1 ttl=61 time=1.20 ms

--- 192.168.2.130 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 1.199/1.199/1.199/0.000 ms
```










