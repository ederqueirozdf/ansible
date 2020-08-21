
# Ansible
<hr>
Criação de usuário, grupo e permissão sshd via ansible

    ---
    - hosts: teste-sudoers
      become: yes
      vars:
        usr_group_name: localgroup
        usr_sudo_group: wheel
      tasks:
        - group:
           name: '{{ usr_group_name }}'
           state: present
        - name: Create user
          user:
           name: usuario1
           comment: Service user
           groups: '{{ usr_sudo_group }}','{{ usr_group_name }}'
           shell: /bin/bash
           password: '$1$Qfcoeo4N$7ER6zcQHqNuWd2Nr2Ttwn/'
           createhome: yes
        - name: Allow 'wheel' group to have passwordless sudo
          lineinfile:
            dest: /etc/sudoers
            state: present
            regexp: '^%wheel'
            line: '%wheel ALL=(ALL) NOPASSWD: ALL'
            validate: 'visudo -cf %s'
        - name: Add Group to AllowGroups
          lineinfile:
            dest: /etc/ssh/sshd_config
            state: present
            regexp: '^(AllowGroups(?!.*\b{{ usr_group_name }}\b).*)$'
            line: '\1 {{ usr_group_name }}'
            backup: True
            backrefs: True
