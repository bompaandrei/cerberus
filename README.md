# cerberus - OS: Ubuntu 20.04 LTS
Holds ansible automation for Cerberus project.

https://www.cerberus.zone/running-a-validator.html

Create an inventory file that cointains the following:

```
[$NAME.cerberus]
$IP_ADDRESS

[$NAME.cerberus:vars]
ansible_connection=ssh
ansible_user=root
validator_owner=$NAME_OF_THE_VALIDATOR
moniker_name="$NAME_OF_THE_MONIKER"
```
^^^ all names can be the same ^^^

# How to run

### Step 1: 
## At this step linux is configured, firewall deployed and everything is prepared for Cerberus
```
ansible-playbook -i VALIDATOR.inventory playbooks/01_linux_config.yaml
ansible-playbook -i VALIDATOR.inventory playbooks/02_firewall.yaml
ansible-playbook -i VALIDATOR.inventory playbooks/03_cerberus_dependency_deployer.yaml
```

### Step 2:
## At this step Cerberus is installed, Moniker initiated and the wallet is created
```
cd /home/cerberusjah/; ./cerberus_installer.sh
```
```
cerberusd init "${MONIKER_NAME}" --chain-id cerberus-chain-1
```
```
cd /home/cerberusjah/; ./cerberus_config.sh
```
```
cerberusd keys add <key-name> --keyring-backend os
```

### Step 3:
## This will only create the service file for systemd and start the service
```
ansible-playbook -i VALIDATOR.inventory playbooks/04_cerberus_deployer.yaml
```

### Step 4:
## Just wait for the app to be in sync and then create the validator
`Wait for the validator to be in sync (cerberusd status) and then run the create-validator command:`

```
cerberusd tx staking create-validator \
      --from "<key-name>" \
      --amount "1000000000000ucrbrus" \
      --pubkey=$(cerberusd tendermint show-validator) \
      --commission-max-change-rate 0.01 \
      --commission-max-rate 0.20 \
      --commission-rate 0.05 \
      --min-self-delegation 1 \
      --chain-id "cerberus-chain-1" \
      --moniker $MONIKER_NAME \
      --details "Enter a description about your validation" \
      --security-contact "security contact email" \
      --website "www.SomeWebsite.com" -y
```