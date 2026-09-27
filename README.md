# API de Gerenciamento de Estoque — DDA Metalúrgica

## 📋 Sobre o projeto

API REST desenvolvida para o **gerenciamento de estoque de pastilhas industriais da DDA Metalúrgica**.

O sistema tem como objetivo controlar informações de fornecedores, pastilhas, estoque, movimentações de entrada e saída, alertas de estoque crítico e indicadores para acompanhamento gerencial.

A API foi planejada utilizando uma abordagem **API First**, com contrato definido em **OpenAPI 3.0**.

## 🏭 Contexto

A API será utilizada por um frontend web responsivo, acessado por gestores e operadores da empresa em computadores, tablets e celulares utilizados no chão de fábrica.

Entre os principais dados controlados estão:

* Cadastro de fornecedores;
* Cadastro de pastilhas industriais;
* Estoque mínimo;
* Saldo de estoque;
* Movimentações de entrada e saída;
* Histórico de movimentações;
* Alertas de estoque crítico;
* Indicadores gerenciais.

A integridade dessas informações é importante para evitar alterações indevidas no estoque e garantir que os alertas de ruptura sejam confiáveis.

## 🛠️ Tecnologias previstas

* **Java 17+**
* **Spring Boot 3.x**
* **Spring Security 6**
* **JWT (JSON Web Token)**
* **Spring Data JPA**
* **Bean Validation**
* **PostgreSQL/MySQL**
* **OpenAPI 3.0**

## 🔗 Base da API

```text
/api/v1
```

## 📚 Principais endpoints

### 🔐 Autenticação

```http
POST /api/v1/auth/login
```

Responsável pela autenticação do usuário e emissão do token JWT.

---

### 🏢 Fornecedores

```http
GET    /api/v1/fornecedores
POST   /api/v1/fornecedores
GET    /api/v1/fornecedores/{id}
PUT    /api/v1/fornecedores/{id}
DELETE /api/v1/fornecedores/{id}
```

Permite consultar e administrar os fornecedores cadastrados.

---

### 🔩 Pastilhas

```http
GET    /api/v1/pastilhas
POST   /api/v1/pastilhas
GET    /api/v1/pastilhas/{id}
PUT    /api/v1/pastilhas/{id}
DELETE /api/v1/pastilhas/{id}
GET    /api/v1/pastilhas/alertas
```

Permite consultar e administrar o catálogo de pastilhas e seus níveis mínimos de estoque.

---

### 📦 Movimentações

```http
GET  /api/v1/movimentacoes
POST /api/v1/movimentacoes
```

Responsável pelo registro das movimentações de:

* **ENTRADA**
* **SAÍDA**

As movimentações são mantidas como parte da trilha de auditoria e não possuem operações de alteração ou exclusão.

---

### 📊 Dashboard

```http
GET /api/v1/dashboard/resumo
```

Retorna indicadores consolidados do estoque para acompanhamento gerencial.

## 🔒 Segurança

A API utiliza autenticação **stateless baseada em JWT (Bearer Token)**.

O usuário realiza login através do endpoint:

```http
POST /api/v1/auth/login
```

Após a autenticação, o token deve ser enviado nas requisições protegidas através do header:

```http
Authorization: Bearer <token>
```

O token possui validade de **2 horas**.

A assinatura utiliza **HS256**, com uma chave de pelo menos 256 bits armazenada na variável de ambiente `JWT_SECRET`.

## 👥 Perfis de acesso

A API possui dois perfis principais:

### OPERADOR

Usuário responsável pelas atividades do chão de fábrica.

Pode:

* Consultar fornecedores;
* Consultar pastilhas;
* Consultar alertas de estoque;
* Registrar entradas;
* Registrar saídas;
* Consultar suas próprias movimentações;
* Consultar o resumo do dashboard.

Não pode:

* Cadastrar fornecedores;
* Alterar fornecedores;
* Excluir fornecedores;
* Alterar pastilhas;
* Alterar estoque mínimo;
* Excluir pastilhas;
* Consultar movimentações de outros operadores.

### ADMIN

Responsável pela gestão do estoque.

Pode:

* Consultar fornecedores;
* Cadastrar fornecedores;
* Alterar fornecedores;
* Inativar fornecedores;
* Consultar pastilhas;
* Cadastrar pastilhas;
* Alterar pastilhas;
* Definir estoque mínimo;
* Inativar pastilhas;
* Registrar movimentações;
* Consultar todo o histórico de movimentações;
* Consultar o dashboard.

## 🌐 Recursos públicos

Os seguintes recursos não exigem autenticação:

```text
POST /api/v1/auth/login
OPTIONS /**
GET /actuator/health
```

A documentação Swagger fica disponível somente no ambiente de desenvolvimento:

```text
/swagger-ui/**
/v3/api-docs/**
```

Em produção, esses recursos ficam desabilitados.

## 🛡️ Principais medidas de segurança

O projeto possui um plano de segurança que contempla diferentes ameaças e vulnerabilidades.

Entre as principais medidas estão:

### Senhas

As senhas não são armazenadas em texto puro.

É utilizado:

```text
BCryptPasswordEncoder
```

com fator 12.

Além disso, as senhas não são retornadas pelos DTOs nem armazenadas em logs.

### SQL Injection

O acesso aos dados será realizado através do **Spring Data JPA**, utilizando métodos derivados e consultas parametrizadas.

A concatenação de strings para montagem de consultas SQL é proibida.

### XSS

São utilizadas validações de entrada, respostas em JSON e headers de segurança.

Os dados exibidos pelo frontend devem ser tratados como texto, evitando a utilização de `innerHTML`.

### CSRF

A proteção CSRF fica desabilitada porque a API utiliza autenticação stateless através de token Bearer enviado no header `Authorization`.

Caso a autenticação passe a utilizar cookies, a proteção CSRF deverá ser reativada.

### CORS

O CORS utiliza uma lista explícita de origens permitidas.

Não é utilizado:

```text
*
```

Em desenvolvimento:

```text
http://localhost:5173
```

Em produção:

```text
https://estoque.ddametalurgica.com.br
```

### HTTPS

O HTTPS será obrigatório em produção para proteger senhas e tokens durante a comunicação.

Também será utilizado HSTS.

### Validação de dados

A API utiliza Bean Validation para validar os dados recebidos.

Entre as validações previstas estão:

```text
@NotBlank
@Size
@Email
@Pattern
@Min
```

Dados inválidos devem retornar HTTP `400`.

## 🚨 Códigos HTTP utilizados

| Código | Significado                               |
| ------ | ----------------------------------------- |
| `200`  | Operação realizada com sucesso            |
| `201`  | Recurso criado                            |
| `400`  | Dados enviados são inválidos              |
| `401`  | Usuário não autenticado ou token inválido |
| `403`  | Usuário autenticado, mas sem permissão    |
| `404`  | Recurso não encontrado                    |
| `409`  | Conflito de regra de negócio              |
| `500`  | Erro interno do servidor                  |

Os erros da API devem utilizar uma estrutura padronizada de resposta, evitando exposição de stack trace ou informações internas do sistema.

## 🔑 Autorização

A autorização é realizada em diferentes níveis.

### SecurityFilterChain

Controla o acesso aos endpoints de acordo com o método HTTP e o perfil do usuário.

### @PreAuthorize

As regras também podem ser aplicadas diretamente nos serviços, proporcionando uma segunda camada de proteção.

### Regra de negócio

Além do perfil, determinadas operações possuem regras específicas.

Por exemplo, o operador pode consultar somente suas próprias movimentações, enquanto o administrador pode consultar todo o histórico.

## 📦 Integridade do estoque

As movimentações de estoque são tratadas dentro de transações.

Para evitar problemas de concorrência, está previsto o uso de:

* Bloqueio de linha; ou
* Versionamento otimista.

Uma saída de estoque também deve verificar se existe saldo suficiente antes de realizar a operação.

A quantidade da movimentação deve ser maior que zero.

## 📝 Auditoria

As movimentações são consideradas imutáveis.

Não existem endpoints para alterar ou excluir uma movimentação.

Caso seja necessário corrigir uma operação, deverá ser realizada uma **movimentação de estorno**, preservando a trilha de auditoria.

Também estão previstos registros de:

* Login com sucesso;
* Tentativas de login com falha;
* Acessos negados;
* Alterações de cadastro;
* Usuário responsável;
* IP;
* Data e hora.

Dados sensíveis não devem ser registrados nos logs.

## 📖 Documentação da API

A API utiliza **OpenAPI 3.0** para definição do contrato.

Durante o desenvolvimento, a documentação poderá ser acessada através do Swagger UI:

```text
/swagger-ui/
```

A documentação OpenAPI:

```text
/v3/api-docs/
```

Esses recursos ficam disponíveis somente no ambiente de desenvolvimento.

## ⚙️ Variáveis de ambiente

Informações sensíveis não devem ser armazenadas diretamente no código-fonte.

Exemplo:

```text
JWT_SECRET
```

Também devem ser protegidas as credenciais utilizadas para acesso ao banco de dados.

Essas informações devem ficar fora do repositório Git.

## 📁 Estrutura conceitual

```text
API
│
├── Autenticação
│   └── JWT
│
├── Fornecedores
│   ├── Consulta
│   └── Administração
│
├── Pastilhas
│   ├── Catálogo
│   ├── Estoque mínimo
│   └── Alertas
│
├── Movimentações
│   ├── Entrada
│   ├── Saída
│   └── Auditoria
│
└── Dashboard
    └── Indicadores
```

## 🔐 Resumo das permissões

| Operação                         | Público | OPERADOR | ADMIN |
| -------------------------------- | :-----: | :------: | :---: |
| Login                            |    ✅    |     ✅    |   ✅   |
| Consultar fornecedores           |    ❌    |     ✅    |   ✅   |
| Cadastrar fornecedor             |    ❌    |     ❌    |   ✅   |
| Alterar fornecedor               |    ❌    |     ❌    |   ✅   |
| Excluir/inativar fornecedor      |    ❌    |     ❌    |   ✅   |
| Consultar pastilhas              |    ❌    |     ✅    |   ✅   |
| Cadastrar pastilha               |    ❌    |     ❌    |   ✅   |
| Alterar pastilha                 |    ❌    |     ❌    |   ✅   |
| Alterar estoque mínimo           |    ❌    |     ❌    |   ✅   |
| Consultar alertas                |    ❌    |     ✅    |   ✅   |
| Registrar entrada                |    ❌    |     ✅    |   ✅   |
| Registrar saída                  |    ❌    |     ✅    |   ✅   |
| Consultar próprias movimentações |    ❌    |     ✅    |   ✅   |
| Consultar todas as movimentações |    ❌    |     ❌    |   ✅   |
| Alterar movimentação             |    ❌    |     ❌    |   ❌   |
| Excluir movimentação             |    ❌    |     ❌    |   ❌   |
| Consultar dashboard              |    ❌    |     ✅    |   ✅   |

## 🎯 Objetivo da segurança

O plano de segurança busca proteger principalmente:

* Confidencialidade dos dados;
* Integridade do estoque;
* Autorização dos usuários;
* Rastreabilidade das movimentações;
* Disponibilidade da API;
* Proteção das credenciais;
* Segurança da comunicação entre frontend e API.

A implementação das regras de segurança deverá ser realizada posteriormente utilizando **Spring Security 6**, conforme as decisões definidas neste plano.

## 👨‍💻 Projeto

**Projeto:** Sistema de Gerenciamento de Estoque de Pastilhas Industriais
**Empresa:** DDA Metalúrgica
**Arquitetura:** API First
**Contrato:** OpenAPI 3.0
**Base URL:** `/api/v1`

**Repositório:**
https://github.com/rojerio/atividade-openapi

## 📚 Referências

MENEZES, Nilo Ney Coutinho. *Introdução à programação com Python – 4ª edição: Algoritmos e lógica de programação para iniciantes*. Novatec Editora, 2024.

WAZLAWICK, Raul Sidnei. *Introdução a Algoritmos e Programação com Python: Uma abordagem dirigida por testes*. Elsevier Editora Ltda., 2017.

PASQUALOTTI, Paulo Roberto. *Algoritmos e programação*. Florianópolis: SENAI/SC, 2022. 316 p.
