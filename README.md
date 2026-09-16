# Planos & Beneficiários

API REST + Angular para gestão de planos e beneficiários (.NET 10 / PostgreSQL 17).

> **Contexto:** este projeto começou como um desafio técnico.
> A arquitetura inicial foi fornecida; o desenvolvimento do módulo de
> Beneficiários, testes e frontend é meu.

---

## Stack

| Camada   | Tecnologia                          |
|----------|-------------------------------------|
| Backend  | .NET 10 (C#), Entity Framework Core |
| Frontend | Angular 20 (TypeScript)             |
| Banco    | PostgreSQL 17                       |
| Infra    | Docker, Docker Compose              |

---

## Como rodar

**Pré-requisitos:** Docker e Docker Compose instalados.

```bash
docker compose up --build
```

| Serviço | URL                    |
|---------|------------------------|
| API     | http://localhost:9999  |
| Web     | http://localhost:4200  |

---

## O que eu implementei

### Backend

- **Módulo de Beneficiários completo**: endpoints de CRUD com paginação e filtros combináveis (`status`, `plano_id`, `nome`), seguindo o padrão do módulo de Planos.
- **Domínio `Beneficiario`**: validação separada para criação e atualização, CPF imutável, exclusão lógica com `ExcluidoEm`.
- **Regra de inativo congelado**: beneficiário com status INATIVO não pode ter dados alterados — retorna 409.
- **Unicidade de CPF real**: índice único no banco + captura de `DbUpdateException` para retornar 409 mesmo sob concorrência.
- **Correções no código base**: filtro `plano_id` em snake_case, conversor de `DateOnly` com timezone UTC, exclusão lógica no lugar de delete físico, DTOs separados para POST e PUT.
- **Testes**: ajustados os desalinhados com a spec e adicionados novos para validação, filtros e reativação.

### Frontend

- **Roteamento e menu** entre Planos e Beneficiários.
- **Listagem de Beneficiários** com filtros combináveis, paginação, loading e tratamento de erro.
- **Formulário** de cadastro e edição com validações no cliente, CPF desabilitado na edição e mensagens específicas para erros 400, 409 e 422.

### O que ficou de fora

- Testes automatizados do frontend (não exigidos no escopo).
- Bibliotecas de componentes (mantive o padrão visual do módulo de Planos).
- Funcionalidades além da especificação.

---

## Decisões técnicas

- **Padrão de erro e camadas**: segui exatamente o modelo do módulo de Planos (referência da casa) — DTOs, serviços, tratamento de exceção e nomenclatura.
- **Concorrência de CPF**: garantia real via índice único no PostgreSQL; a consulta prévia no serviço serve apenas para resposta rápida em casos normais.
- **Exclusão lógica**: mantém histórico e o CPF ocupado, conforme especificação.
- **CPF imutável**: DTOs separados (`BeneficiarioRequest` com CPF no POST, `BeneficiarioUpdateRequest` sem CPF no PUT).
- **Ordenação padrão**: `NomeCompleto` + `Id` como desempate, garantindo paginação estável.
- **Frontend reativo**: uso de **Signals** para estado e `Subject` + `switchMap` para cancelar requisições antigas ao trocar filtros/página (evita o erro `NG0203` do `takeUntilDestroyed` fora do construtor).
- **Componentes standalone**: seguindo o padrão já usado em Planos, sem `NgModule`.

### Pontos que a spec não definia e decidi

- **Filtros manuais** (botão "Filtrar") em vez de automáticos — evita requisições desnecessárias.
- **Exclusão de inativo** permitida — a spec não proíbe e exclusão lógica é remoção, não mudança de status.
- **Tamanho padrão da página**: 10, com seletor no frontend (5, 10, 20, 50).

### Inconsistências entre spec e testes

- Teste esperava 20 itens por página, spec define 10 → ajustei o teste.
- Teste esperava 200 ao alterar beneficiário inativo, spec define 409 → ajustei o teste.

---

## Origem

Projeto desenvolvido a partir do código base de um desafio técnico público
da 4Tech. A arquitetura inicial
— módulo de Planos, scaffolding do .NET e do Angular, setup do Docker — foi
fornecida como ponto de partida. A implementação do módulo de Beneficiários,
as correções e a finalização da interface são de minha autoria.