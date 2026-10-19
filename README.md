# Corte Certo — Sistema de Agendamento de Serviços

Projeto Integrador Transdisciplinar em Engenharia de Software II (Cruzeiro do Sul Virtual).
**Aluno:** Eduardo Adelino Garcia Junior — **RGM:** 31703429

Aplicação web para agendar serviços de barbearia: o cliente cria conta, escolhe o serviço e o horário,
acompanha os agendamentos e vê um painel com resumo e gráfico.

| Item | Tecnologia |
|---|---|
| Back-end | Python 3.12 + Flask (padrão **MVC**) |
| Banco de dados | PostgreSQL (produção) · SQLite (testes/dev) |
| Front-end | HTML, CSS e JavaScript puros (responsivo) |
| Testes | `unittest` (compatível com `pytest`) |
| Hospedagem | Render (web service + PostgreSQL) |
| Modo de codificação | Tradicional |

## Estrutura do repositório

```
backend/    app/models (M) · app/controllers (C) · app/db.py · app/validators.py · tests/
frontend/   index.html (V) · style.css · app.js
database/   schema.sql (projeto físico: tabelas, restrições, índices, carga inicial)
docs/       situacao-1-planejamento · situacao-2-codificacao · situacao-3-testes
render.yaml configuração de deploy
```

MVC: **Model** (`backend/app/models`) concentra o SQL; **Controller** (`backend/app/controllers`) recebe as
requisições, valida e responde JSON; **View** é o front-end em `frontend/`, que consome a API.

## Como rodar localmente

```bash
cd backend
python -m venv .venv && source .venv/bin/activate      # Windows: .venv\Scripts\activate
pip install -r requirements.txt
python -m flask --app wsgi run --debug                  # http://127.0.0.1:5000 (usa SQLite local)
```

Para usar PostgreSQL localmente, defina `DATABASE_URL=postgresql://usuario:senha@localhost:5432/corte_certo`.

## Testes

```bash
cd backend
python -m unittest -v      # ou: pytest
```

Cobrem validações (data futura, horário comercial, senha, e-mail), autenticação, perfil, agendamentos,
paginação/filtro/ordenação, isolamento entre usuários e dashboard.

## Deploy no Render

1. Suba este repositório no GitHub.
2. No Render: **New → Blueprint**, selecione o repositório (o `render.yaml` cria o web service e o PostgreSQL).
3. Aguarde o build. A URL `https://corte-certo.onrender.com` (ou similar) é o link da solução em funcionamento.

> O plano gratuito do PostgreSQL do Render expira após um período. Se precisar de banco permanente grátis,
> use Neon ou Supabase e cole a connection string na variável `DATABASE_URL` do serviço.

## API (resumo)

| Método e rota | Descrição |
|---|---|
| `POST /api/auth/register` · `/login` · `/logout` · `GET /api/auth/me` | Conta e sessão |
| `PUT /api/perfil` · `PUT /api/perfil/senha` | Editar dados e trocar senha |
| `GET /api/servicos` | Lista de serviços |
| `GET /api/agendamentos?pagina&por_pagina&q&status&ordem&direcao` | Listagem paginada, filtrada e ordenada |
| `POST /api/agendamentos` | Criar (data futura, 08h–20h, horário livre) |
| `PATCH /api/agendamentos/<id>/status` · `DELETE /api/agendamentos/<id>` | Concluir/cancelar e excluir |
| `GET /api/dashboard` | Totais por status, por serviço e próximo horário |

## Correções aplicadas a partir do laudo

| Problema relatado | Correção |
|---|---|
| Erro 500 ao salvar e-mail no perfil | `UPDATE` tratado: e-mail repetido retorna 409 com mensagem clara |
| Tabela cortada no celular | Tabela vira lista de cartões em telas estreitas + rolagem horizontal de segurança |
| Datas retroativas aceitas | Validação no front-end (`min` do campo) e no controller (`validar_data_hora`) |
| Gráfico lento (> 5 s) | Índices em `agendamentos (usuario_id, data_hora)` e `(status)` |
| Registros duplicados com duplo clique | Botão desabilitado durante a requisição + índice único de horário ativo no banco |

Funcionalidades ainda não implementadas (registradas no laudo): recuperação de senha por e-mail,
upload de foto, exportação em PDF, notificações em tempo real, tema escuro manual e integração com pagamentos.
