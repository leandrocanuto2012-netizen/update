# Pacote v2 — módulos de gestão Assados Nobre ERP

## Conteúdo
- `03-modulos-gestao.sql`: tabelas, índices e políticas RLS para Supabase.
- `gestao-router.js`: API Express com autorização por módulo/ação, escopo de filial e usuário, trilha de auditoria, edição/exclusão controlada e prazos jurídicos.
- `modulos-gestao.html`: interface responsiva para celular com painel individual, dashboard das filiais, Contabilidade, EDRAS e Jurídico.
- `testes-basicos.js`: verificações estáticas de integridade dos arquivos e rotas.

## Instalação segura
1. Faça backup do projeto Supabase e do repositório.
2. Execute `03-modulos-gestao.sql` no SQL Editor do projeto Supabase correto.
3. Copie `gestao-router.js` para a raiz do servidor e `modulos-gestao.html` para a pasta pública do ERP (por exemplo, `public/modulos-gestao.html`).
4. Integre somente depois de fornecer middleware de sessão assinada e autorização reais. Exemplo de montagem:

```js
const createGestaoRouter = require('./gestao-router');
app.use('/api/gestao', createGestaoRouter({ supabase, authenticate, authorize }));
```

5. `authenticate` precisa validar uma sessão assinada real e definir `req.user` com `id` ou `matricula`, `nome`, `unidade_id`, `unidade_tipo` e `role`. Não use o token legado `tok-<id>` como prova de autenticação.
6. `authorize(modulo, acao)` deve devolver middleware Express e verificar a permissão correspondente. Permissões necessárias: `dashboard_usuario`, `dashboard_filiais`, `contabilidade` (ações `criar`, `editar`, `excluir`, `confirmar`), `edras` (ações `criar`, `editar`, `excluir`) e `juridico` (ações `criar`, `editar`).
7. Para consolidação da rede, o middleware deve definir `req.unidadesPermitidas` e `req.user.pode_ver_todas_filiais=true` apenas para perfis da matriz expressamente autorizados. Os usuários comuns ficam limitados à própria unidade.
8. Sirva `modulos-gestao.html` em rota autenticada. A interface chama `/api/gestao/...` na mesma origem.

## Regras e limites
- Contabilidade: lançamentos começam como `rascunho`; só podem ser editados/excluídos enquanto rascunho. Confirmação exige permissão específica. O módulo não substitui escrituração, apuração fiscal nem validação de contador.
- EDRAS: cada usuário consulta e altera apenas os próprios movimentos, dentro da própria unidade. É ferramenta de organização financeira, não recomendação de investimento/crédito.
- Jurídico: processos e prazos ficam restritos à unidade; sigilo precisa ser controlado por permissões. Upload de documentos ainda não está implementado; não use URLs públicas. O módulo é controle administrativo, não aconselhamento jurídico.
- Dashboard das filiais: a matriz só consolida todas as unidades se receber autorização explícita no contexto autenticado.
- A interface não contorna permissões: todos os dados passam pela API protegida.

## Verificação
Execute `node --check gestao-router.js` e `node testes-basicos.js`. Isso valida sintaxe e estrutura estática, não substitui teste integrado com Supabase e autenticação.

## Estado da publicação
Este pacote é uma entrega para integração manual. A tentativa anterior de criar branch no GitHub retornou HTTP 403 (`Resource not accessible by integration`); nenhum arquivo foi publicado automaticamente no repositório.
