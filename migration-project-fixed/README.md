# Enterprise Migration Framework — VMware Gen2 → OpenShift Virtualization Gen3

## Struttura

```
migration-project/
├── site.yml                          # Playbook principale
├── ansible.cfg                       # Config Ansible
├── inventory.yml                     # localhost connection:local
├── .gitignore
├── group_vars/
│   └── all.yml                       # Tutte le variabili
└── roles/
    ├── awx_setup/                    # Deploy AWX via Helm su gabvirt
    ├── infra_setup/                  # Deploy MinIO pod + service
    ├── data_ingestion/               # Upload immagine su MinIO
    ├── migration_cdi/                # Secret S3 + DataVolume CDI
    └── vm_provisioning/              # VirtualMachine KubeVirt
```

## Esecuzione

### 1. Deploy AWX (una volta sola)
```bash
ansible-playbook site.yml --tags "awx_setup"
```

### 2. Deploy MinIO (una volta sola)
```bash
ansible-playbook site.yml --tags "infra_setup"
```
Poi avvia il port-forward in un terminale separato:
```bash
oc port-forward svc/landing-zone-service 9000:9000 -n gabvirt
```

### 3. Migrazione completa
```bash
ansible-playbook site.yml --tags "data_ingestion,migration_cdi,vm_provisioning"
```

## AWX — Configurazione Job Template

Dopo il deploy di AWX accedi a:
```
https://awx-gabvirt.apps.migrationlab.devopstribe.it
```

Crea:
- **Organization**: Default
- **Project**: migration-project → `https://github.com/gabri-souce/migration-project.git`
- **Credential**: OpenShift API token
- **Job Template**: site.yml con tag `data_ingestion,migration_cdi,vm_provisioning`
