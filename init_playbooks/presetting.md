# Преднастройка серверов

## Общие для всех серверов

Все скрипты запускаем из корневой директории репозитория.

- Учетные записи

```bash
# Завести всех пользователей обозначенных в файле ansible/presetting_playbooks/add_user.playbook
ansible-playbook ansible/init_playbooks/add_user.yml -i ansible/hosts_tested.ini

# Завести пользователя support и coder c используя тег имени
ansible-playbook ansible/init_playbooks/add_user.yml -i ansible/hosts_tested.ini -t support

# Обновить ключи списки - одноименные c учётной записью файлы *.keys (ansible/presetting_playbooks/ssh_keys/iflexsupport.keys)
ansible-playbook ansible/init_playbooks/add_user.playbook -i ansible/hosts_tested.cfg -t update_key
```

> Важно! Когда необходимо выполнить на ограниченном наборе хостов какой либо play-book следует явно это указать. **для примера**:

```shell
~$ ansible-playbook playbook.yml --limit k8s #, где k8s группа хостов
~$ ansible-playbook playbook.yml --limit-hosts # вывести список хостов на которые будут применён playbook
```

## Добавление пользователя и ключей авторизации

add_user.yml - Добавляет пользователей и ключи авторизации из \*.keys

```yml
- name: Set authorized keys tacken from file
  ansible.posix.authorized_key:
    user: support_test
    state: present
    key: "{{ lookup('file', 'support_test.keys') }}"
    exclusive: True # параметр заставляет зачистить файл перед добавлением параметров
```

## Работа над ошибками

В случае, если часть проскочила проблема на каком либо хосте,

```shell
10.28.80.143               : ok=0    changed=0    unreachable=1    failed=0    skipped=0    rescued=0    ignored=0)
```

следует попробовать выполнить

```shell
ansible-playbook add_user.yml  --limit 10.28.80.143 -vvv # мы получим вывод с информацией для отладки

<10.28.80.143> Failed to connect to the host via ssh: mkdir: невозможно создать каталог «/home/support/.ansible/tmp/ansible-tmp-1613687607.4901924-10434-190101718825834»: На устройстве не осталось свободного места
fatal: [10.28.80.143]: UNREACHABLE! => {
```
