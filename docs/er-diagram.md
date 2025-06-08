# ER Diagram

It's er-diagram for LCL project

``` mermaid
erDiagram
  accounts {
    bigint id PK
    string email
    string password
  }
  
  users {
    string id PK
    bigint account_id
    string name
  }

  refresh_tokens {
    string token PK
    bigint account_id FK
    string user_id FK
    timestamp expired_at
  }

  users_groups {
    id string PK
    bigint account_id
    user_id string
    group_id string
  }
  
  groups {
    string id PK
    bigint account_id
    string name
  }
  
  roles {
    string id PK
    bigint account_id
    string name
  }
  
  users_roles {
    string id PK
    bigint account_id
    string user_id
    string role_id
  }
  
  groups_roles {
    string id PK
    bigint account_id
    string group_id
    string role_id
  }
  
  policies {
    string id PK
    bigint account_id
  }
  
  policy-MongoDB
  
  roles_policies {
    string id PK
    bigint account_id
    string roles_id
    string policy_id
  }
  
  m_services {
    string name PK
  }
  
  m_abilities {
    string name PK
    string service_id
  }
  
  accounts ||--o{ users : ""
  users ||--o{ users_groups : ""
  accounts ||--o| refresh_tokens : ""
  users ||--o| refresh_tokens: ""
  groups ||--o{ users_groups : ""
  users ||--o{ users_roles: ""
  roles ||--o{ users_roles: ""
  groups ||--o{ groups_roles: ""
  roles ||--o{ groups_roles: ""
  roles ||--o{ roles_policies: ""
  policies ||--o{ roles_policies: ""
  policies ||--|| policy-MongoDB: ""
  m_services ||--|{ m_abilities: ""
  m_abilities }|--|{ policy-MongoDB: ""
```
