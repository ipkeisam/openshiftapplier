# OpenShift Applier for Labs

https://github.com/redhat-cop/openshift-applier

# Apply Application

Install Requirements if not already installed.

`[.openshift-applier]$ ansible-galaxy install -r requirements.yml --roles-path=roles`

Right now limited to using ansible on your localhost.

`[.openshift-applier]$ ansible-playbook apply.yml -i inventory/`

See the inventory for the filter tag options.

Example of `filter_tags`
`[.openshift-applier]$ ansible-playbook apply.yml -i inventory/ -e filter_tags=something-here`


#Apply Secrets

`[.openshift-applier]$ ansible-galaxy install -r requirements-secrets.yml --roles-path=roles`

`[.openshift-applier]$ ansible-playbook secrets.yml -i inventory/ --ask-vault-pass`


#Steps to deploy to separate namespace (usually your username)

##ci/cd
export NAMESPACE=$(whoami)
ansible-playbook secrets.yml --ask-vault-pass -e ci_cd_namespace=$NAMESPACE
ansible-playbook site.yml -e ci_cd_namespace=$NAMESPACE -e dev_namespace=$NAMESPACE -e "filter_tags=s2i"

##service openshift applier 
ansible-playbook secrets.yml -i inventory/ --ask-vault-pass -e dev_namespace=$NAMESPACE
ansible-playbook apply.yml -i inventory/ -e ci_cd_namespace=$NAMESPACE -e dev_namespace=$NAMESPACE -e filter_tags=dev

oc scale --replicas=0 dc jenkins

oc start-build -n $NAMESPACE vehicle-service --from-file=../target/vehicle-service-0.0.1-SNAPSHOT.jar