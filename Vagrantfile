# -*- mode: ruby -*-
# vi: set ft=ruby :

VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|


 config.vm.define "worker0" do |worker0|
   worker0.vm.hostname = "worker0"
   worker0.vm.box = "centos-9-stream"
   worker0.vm.provision "shell", inline: <<-SHELL
     sudo dnf clean metadata
     sudo dnf install -y git python3-pip podman
     su - vagrant -c "pip3 install --upgrade pip --user"
     su - vagrant -c "pip3 install setuptools-rust ansible virtualenv tox molecule molecule-podman  --user"     
     git clone  https://opendev.org/opendev/base-jobs.git
     git clone https://opendev.org/zuul/zuul-jobs.git
     git clone https://opendev.org/openstack/tripleo-ansible.git
     echo "localhost ansible_connection=local" > /home/vagrant/inventory
     mkdir -p /home/vagrant/.ansible/roles
     cp -a /home/vagrant/zuul-jobs/roles/* /home/vagrant/.ansible/roles/
     cp -a /home/vagrant/base-jobs/roles/* /home/vagrant/.ansible/roles/
     mkdir -p /home/vagrant/.ansible/plugins/modules/
     cp -a /home/vagrant/tripleo-ansible/tripleo_ansible/ansible_plugins/* /home/vagrant/.ansible/plugins/
     sudo chown -R vagrant:vagrant /home/vagrant/*
     sudo chown -R vagrant:vagrant .ansible
     ssh-keygen  -t rsa -f /home/vagrant/.ssh/id_rsa  -P ''
     sudo chown -R vagrant:vagrant /home/vagrant/.ssh/*
     cat /home/vagrant/.ssh/id_rsa.pub >> /home/vagrant/.ssh/authorized_keys
     cat > /home/vagrant/0001-Vagrant-test.patch <<EOF
From b0e80f8b6cfa2b9477d4ee983cf37a32a2ee5e35 Mon Sep 17 00:00:00 2001
From: Your Name <you@example.com>
Date: Tue, 25 Oct 2022 08:19:53 +0000
Subject: [PATCH] Vagrant test

---
 zuul.d/playbooks/pre.yml | 6 +++---
 1 file changed, 3 insertions(+), 3 deletions(-)

diff --git a/zuul.d/playbooks/pre.yml b/zuul.d/playbooks/pre.yml
index ce712601..773b9060 100644
--- a/zuul.d/playbooks/pre.yml
+++ b/zuul.d/playbooks/pre.yml
@@ -6,7 +6,7 @@
       import_tasks: directories.yml
     - name: Set project path fact
       set_fact:
-        tripleo_ansible_project_path: "{{ ansible_user_dir }}/{{ zuul.projects['opendev.org/openstack/tripleo-ansible'].src_dir }}"
+        tripleo_ansible_project_path: "{{ ansible_user_dir }}/tripleo-ansible"
 
     - name: Ensure pip is available
       include_role:
@@ -85,7 +85,7 @@
       become: true
     - name: Get Ansible Galaxy roles
       command: >-
-        {{ ansible_user_dir }}/test-python/bin/ansible-galaxy install
+        ansible-galaxy install
         -fr
         {{ tripleo_ansible_project_path }}/tripleo_ansible/ansible-role-requirements.yml
       environment:
@@ -93,7 +93,7 @@
 
     - name: Get Ansible Galaxy collections
       command: >-
-        {{ ansible_user_dir }}/test-python/bin/ansible-galaxy collection install
+        ansible-galaxy collection install
         -fr
         {{ tripleo_ansible_project_path }}/tripleo_ansible/requirements.yml
       environment:
-- 
2.31.1

EOF
     cd /home/vagrant/tripleo-ansible
     git apply /home/vagrant/0001-Vagrant-test.patch
     cd ~
     sudo dnf install centos-release-openstack-yoga -y
     su - vagrant -c "ansible-playbook  -i /home/vagrant/inventory /home/vagrant/tripleo-ansible/tripleo_ansible/playbooks/prepare-test-host.yml -e ansible_user=vagrant -e ansible_user_dir=/home/vagrant/"
     su - vagrant -c "ansible-playbook  -i /home/vagrant/inventory /home/vagrant/tripleo-ansible/zuul.d/playbooks/pre.yml  -e ansible_user=vagrant -e ansible_user_dir=/home/vagrant"
     su - vagrant -c "echo 'source /home/vagrant/tripleo-ansible/ansible-test-env-podman.rc' >> /home/vagrant/.bashrc"
     su - vagrant -c "echo '' >> /home/vagrant/.bashrc"
     su - vagrant -c "echo 'source /home/vagrant/test-python/bin/activate' >> /home/vagrant/.bashrc"
     su - vagrant -c "echo 'sudo setenforce 0'"
  SHELL

  end

end
