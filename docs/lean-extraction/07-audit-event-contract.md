# Contrato do AuditEvent — CloudEvents 1.0

O evento de auditoria segue o envelope [CloudEvents 1.0](https://cloudevents.io/).

## Estrutura

```json
{
    "specversion": "1.0.1",
    "type": "AuditEvent",
    "source": "https://www.example.com/",
    "id": "app-domain-key",
    "time": "2020-04-05T17:31:00Z",
    "contenttype": "application/json",
    "data": {
        "appInfo": {
            "appname": "example-app",
            "domain": "sales",
            "version": "1.0.0"
        },
        "user": {
            "id": "12345",
            "name": "John Doe",
            "role": "admin"
        },
        "operation": {
            "type": "insert",
            "entity": "users",
            "generatedAt": "2020-04-05T17:30:00Z",
            "transmittedAt": "2020-04-05T17:31:00Z"
        },
        "index": {
            "indexField1": "value1",
            "indexField2": "value2"
        },
        "data": {
            "id": "12345",
            "name": "John Doe",
            "email": "mail@to.com"
        }
    }
}
```

## Descrição dos campos

### Envelope CloudEvents

| Campo         | Tipo   | Descrição                                         |
|---------------|--------|---------------------------------------------------|
| `specversion` | string | Versão da spec CloudEvents (1.0.1)               |
| `type`        | string | Tipo do evento: sempre `AuditEvent`              |
| `source`      | string | URL da aplicação que gerou o evento              |
| `id`          | string | Identificador único: `app-domain-key`            |
| `time`        | string | Timestamp ISO 8601 do momento de transmissão     |
| `contenttype` | string | Sempre `application/json`                        |

### data.appInfo — Identificação da aplicação

| Campo     | Descrição                                          |
|-----------|----------------------------------------------------|
| `appname` | Nome da aplicação que gerou o evento              |
| `domain`  | Domínio de negócio (ex: `sales`, `financeiro`)    |
| `version` | Versão da aplicação                               |

### data.user — Autor da operação

| Campo  | Descrição                                  |
|--------|--------------------------------------------|
| `id`   | Identificador do usuário no sistema        |
| `name` | Nome do usuário                            |
| `role` | Papel/perfil do usuário                   |

### data.operation — Operação realizada

| Campo           | Descrição                                                   |
|-----------------|-------------------------------------------------------------|
| `type`          | Tipo da operação: `insert`, `update` ou `delete`           |
| `entity`        | Nome da entidade/tabela afetada                            |
| `generatedAt`   | Timestamp de quando a operação foi gerada no sistema       |
| `transmittedAt` | Timestamp de quando o evento foi enviado ao RavenLedger   |

### data.index — Campos de indexação

Mapa livre de campos chave-valor para facilitar pesquisas e localização do registro auditado. O sistema cliente define quais campos incluir como índice.

### data.data — Snapshot do dado

Snapshot do estado do objeto no momento da operação. Estrutura livre, definida pelo sistema cliente.

## Observações de design

- `id` usa o padrão `app-domain-key` para garantir unicidade e rastreabilidade por origem.
- A diferença entre `generatedAt` e `transmittedAt` permite detectar latência ou falhas na transmissão.
- O campo `index` é estratégico: permite que o RavenLedger filtre registros sem precisar fazer full scan no `data`.
- `data.data` é o payload completo — sua imutabilidade é garantida pelo RavenLedger após recepção.
