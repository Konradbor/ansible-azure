---
theme: gaia
_class: lead
paginate: true
# backgroundColor: #fff
# backgroundImage: url('https://marp.app/assets/hero-background.jpg')
marp: true
---

<img  width="300" src=https://upload.wikimedia.org/wikipedia/commons/thumb/2/24/Ansible_logo.svg/1200px-Ansible_logo.svg.png></img>

<!-- ![bg left:40% 80%](https://marp.app/assets/marp.svg) -->

![bg opacity:.1 brightness:.8](https://cdn.pixabay.com/photo/2019/04/06/07/08/dubrovnik-4106788_1280.jpg)
<!-- # **Ansible** -->

Simplest way to automate apps and IT infrastructure

<style>
p, li{
  text-align:justify;  
  hyphens: auto;
  }
</style>

---

# Co to jest?

- Ansible to narzędzie do wdrażania oprogramowania i zarządzania konfiguracją typu open source, które umożliwia użycie techniki infrastruktura-jako-kod (_Infrastructure as Code - IaC_).
- Zamiast ręcznie przygotowywać system do działania aplikacji, tworzysz skrypt, który zrobi to automatycznie.

---

# Demo

Repozytorium: https://github.com/Konradbor/ansible-azure

```shell
git clone https://github.com/Konradbor/ansible-azure.git
cd ansible-azure
ansible-playbook 04-create-vm-with-nginx.yaml
```

---

### Imperatywny  (☞ﾟヮﾟ)☞  ☜(ﾟヮﾟ☜) Deklaratywny

W systemach zarządzających można wyróżnić dwa podejścia:
- Imperatywne - zrób to, potem to, jak się nie powiedzie, to zrób coś innego. Np. "zainstaluj pakiet `nginx`, a potem skopiuj plik `nginx.conf`".

- Deklaratywne - nie interesuje mnie jak to zrobisz, chcę mieć dany efekt. Np. "ta maszyna ma mieć pakiet `nginx` i plik `nginx.conf`.

Przykłady deklaratywnych systemów: Puppet i DSC.

---

### To jaki jest Ansible?

W zasadzie jest mieszanką tych dwóch typów.

<div class="container"><div class="flex-col" data-markdown> 

Podejście deklaratywne:
```yaml
- name: Install nginx
  yum:
    name: nginx
    state: present
```

</div> <div class="flex-col" data-markdown>

Podejście imperatywne:
```yaml
- name: Copy app
  synchronize:
    src: .net-app/
    dest: /app
```

</div></div>

Zaletą podejścia imperatywnego jest prostota tworzenia konfiguracji.

<style scoped>
.container{
    display: flex;
}
.flex-col{
    flex: 1;
    margin-left: 10px;
}
</style>

---

# Dwa tryby - Ad-hoc



Polecenia ad-hoc sprawdzają się w przypadku zadań, które są rzadko powtarzane. Na przykład, jeśli chcesz wyłączyć wszystkie maszyny na święta, możesz wykonać polecenie ad-hoc bez pisania playbooka.

```
$ ansible atlanta -a "/sbin/reboot" -f 10
$ ansible atlanta -m ansible.builtin.copy -a "src=/etc/hosts dest=/tmp/hosts"
```

---

# Dwa tryby - Playbook

<div class="container"><div class="flex-col" data-markdown> 

Playbook jest uruchamiany w kolejności od góry do dołu.
W każdym playbooku są zdefiniowane maszyny (lub grupy maszyn) na których zostanie uruchomiony.
W nim znajdują się zadania, wykonywane od góry do dołu. Playbooki można łączyć w moduły.


</div> <div class="flex-col" data-markdown>

```yaml
- name: update web servers
  hosts: webservers
  remote_user: root

  tasks:
  - name: ensure apache is at the latest version
    yum:
      name: httpd
      state: latest
  - name: write the apache config file
    template:
      src: /srv/httpd.j2
      dest: /etc/httpd.conf
```

</div></div>


<style scoped>
.container{
    display: flex;
}
.flex-col{
    flex: 1;
    margin-left: 10px;
}
</style>

---

# Główne założenie:

Przed wykonaniem każdego polecenia sprawdzany jest stan. Jeżeli nie różni się od oczekiwanego, to akcja nie jest wykonywana.

```
TASK [Install .NET SDK] **********************
ok: [51.145.134.117]

TASK [Copy app] ******************************
changed: [51.145.134.117]

TASK [Publish app] ***************************
changed: [51.145.134.117]

TASK [Copy nginx config] *********************
ok: [51.145.134.117]
```

---

# Uruchamianie playbooka

<div class="container"><div class="flex-col" data-markdown> 

Normalne uruchomienie
```bash
$ ansible-playbook nazwa_playbooka.yaml 
```

Nadpisanie zmiennych

```
$ ansible-playbook nazwa_playbooka.yaml 
--extra-vars "name=test location=westeurope"
```

</div> <div class="flex-col" data-markdown>

Wyświetlanie dodatkowych informacji
```
$ ansible-playbook nazwa_playbooka.yaml -vvv
```

Uruchamianie każdego zadania krok po kroku
```
$ ansible-playbook nazwa_playbooka.yaml --step
```

</div></div>


<style scoped>
.container{
    display: flex;
}
.flex-col{
    flex: 1;
    margin-left: 10px;
}
</style>

---

# Windows

- Can Ansible run on Windows?

No, Ansible can only manage Windows hosts. Ansible cannot run on a Windows host natively, though it can run under the Windows Subsystem for Linux (WSL).

> :warning: The Windows Subsystem for Linux is not supported by Ansible and should not be used for production systems.

---

# Windows

Ale za to można zarządzać maszynami z Windowsem.

```yaml
- name: Install all critical and security updates
  win_updates:
    category_names:
    - CriticalUpdates
    - SecurityUpdates
    state: installed
  register: update_result

- name: Reboot host if required
  win_reboot:
  when: update_result.reboot_required

```
