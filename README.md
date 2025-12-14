<p align="center">
  <img src="https://upload.wikimedia.org/wikipedia/commons/thumb/0/05/Ansible_Logo.png/800px-Ansible_Logo.png" alt="Banner" width="40%">
</p>

## ![Lesson](https://img.shields.io/badge/Lesson-otus__ansible-0A84FF?style=for-the-badge&logo=ansible&logoColor=white&labelColor=111827)![Author](https://img.shields.io/badge/Author-Kamil-10B981?style=for-the-badge&logo=github&logoColor=white&labelColor=111827)![Date](https://img.shields.io/badge/Date-14.12.2025-F59E0B?style=for-the-badge&logo=calendar&logoColor=white&labelColor=111827)

### 📌 Задание
Подготовить стенд и используя Ansible развернуть nginx со следующими условиями:
- [ ] Необходимо использовать модуль `yum/apt`;
- [ ] Конфигурационные файлы должны быть взяты из шаблона `jinja2` с переменными;
- [ ] После установки nginx должен быть в режиме `enabled` в systemd;
- [ ] Должен быть использован `notify` для старта nginx после установки;
- [ ] Сайт должен слушать на нестандартном порту - `8080`, для этого использовать переменные в Ansible.

### ✅ Результат
- [x] Плейбук написан, все условия выполнены.
- [x] Nginx успешно работает на порту 8080 и отдает кастомную страницу. Результат см. на скриншоте 🖼️ ["evidence.png"](evidence.png)

### 🧭 Оглавление
- [🧰 Шаг 1 - Список хостов](#one)
- [🧰 Шаг 2 - Шаблон конфигурации](#two)
- [🧰 Шаг 3 - Плейбук (Playbook)](#three)
- [🧰 Шаг 4 - Проверка работы](#four)

---

<a id="one"></a>
## 🧰 Шаг 1 - Список хостов

Работаем локально (`ansible_connection=local`).

```bash
[webservers]
localhost ansible_connection=local
<a id="two"></a>
```

🧰 Шаг 2 - Шаблон конфигурации

```bash
Nginx
server {
    listen {{ nginx_port }};
    root /var/www/html;
    index index.html;
    server_name _;

    location / {
        try_files $uri $uri/ =404;
    }
}
<a id="three"></a>
```

🧰 Шаг 3 - Плейбук (Playbook)

```bash
- name: Setup Nginx on custom port
  hosts: webservers
  become: yes
  vars:
    nginx_port: 8080

  tasks:
    - name: Install nginx
      apt:
        name: nginx
        state: latest
        update_cache: yes

    - name: Update config
      template:
        src: templates/nginx.conf.j2
        dest: /etc/nginx/sites-available/default
      notify: Reload nginx

    - name: Create index.html
      copy:
        content: "<h1>Otus Ansible {{ nginx_port }}</h1>"
        dest: /var/www/html/index.html
        mode: '0644'

    - name: Enable service
      service:
        name: nginx
        state: started
        enabled: yes

  handlers:
    - name: Reload nginx
      service:
        name: nginx
        state: reloaded
<a id="four"></a>
```

🧰 Шаг 4 - Проверка работы

```bash
ansible-playbook -i hosts.ini site.yml
Проверка доступности порта:
curl -I http://localhost:8080
Вывод консоли:

Plaintext

HTTP/1.1 200 OK
Server: nginx/1.24.0 (Ubuntu)
Date: Sun, 14 Dec 2025 13:13:05 GMT
Content-Type: text/html
Content-Length: 26
Last-Modified: Sun, 14 Dec 2025 13:13:01 GMT
Connection: keep-alive
