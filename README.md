# SecureCertAppDeploy
SecureCertAppDeploy  This repository contains an Ansible project for managing CA certificates and deploying a Python application. It includes roles for configuring standard and custom CA certificates, deploying a Python Flask application into a virtual environment, and managing its configuration securely.




ansible-vault encrypt roles/python_app/vars/*

password: manoj

ansible-vault decrypt roles/python_app/vars/*
ansible-playbook playbooks/main.yml --ask-vault-pass
