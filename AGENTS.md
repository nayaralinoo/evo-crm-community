# Evo CRM Community - Guia para Agentes

## Build do Frontend

**Sempre usar URLs de produção no build:**
```bash
VITE_API_URL=https://api.cicloia.com.br \
VITE_AUTH_API_URL=https://api.cicloia.com.br \
npm run build
```

Nunca rode `npm run build` sem definir as vars — o `.env` local tem `http://localhost:3000` que quebra o login em produção (CSP bloqueia HTTP, só permite HTTPS).

**Após o build, copiar para o nginx:**
```bash
docker cp dist/. evo-crm-community-evo-frontend-1:/usr/share/nginx/html/
```

Se o `nginx.conf` foi alterado, copiar e recarregar:
```bash
docker cp nginx.conf evo-crm-community-evo-frontend-1:/etc/nginx/conf.d/default.conf
docker exec evo-crm-community-evo-frontend-1 nginx -s reload
```

## Deploy de Containers (docker-compose.yml)

### Traefik Routing — evo-auth vs evo-crm

O traefik roteia paths da API baseado em `PathPrefix`. A rule do evo-auth é mais específica (priority 100) e a do evo-crm é genérica (priority 50).

**Todo path de API que pertence ao evo-auth DEVE estar listado na rule do evo-auth.**

Paths comuns do evo-auth que o frontend consome:
- `/api/v1/auth/*` — login, refresh, logout
- `/api/v1/permissions` — permissões do usuário
- `/api/v1/resource_actions` — actions disponíveis
- `/api/v1/roles` — gerenciamento de perfis
- `/api/v1/users` — gerenciamento de usuários
- `/api/v1/oauth_applications` — apps OAuth
- `/api/v1/profile` — dados do perfil do usuário
- `/api/v1/account` — dados da conta
- `/api/v1/access_tokens` — tokens de acesso
- `/api/v1/mfa` — autenticação dois fatores
- `/api/v1/data_privacy` — LGPD/GDPR
- `/api/v1/setup_survey` — survey de setup
- `/api/v1/dynamic_oauth` — OAuth dinâmico
- `/api/v1/features`, `/api/v1/plans`, `/api/v1/user_tours`
- `/oauth/*` — OAuth flow
- `/setup/*` — bootstrap inicial

Se um path novo do auth não for adicionado, a requisição cai no evo-crm e retorna 404/vazio, fazendo o frontend mostrar "Access Denied" ou página em branco.

## Resolução de Problemas Comuns

### "Access Denied" após login
1. Verificar se `/api/v1/permissions` retorna permissões: `curl -H "Authorization: Bearer $TOKEN" https://api.cicloia.com.br/api/v1/permissions`
2. Se retornar vazio ou 404 → traefik routing errado (requisição indo pro evo-crm)
3. Se retornar 401 → token inválido/expirado

### "Página em branco" (ex: Meu Perfil)
1. Verificar se a API de profile retorna dados: `curl -H "Authorization: Bearer $TOKEN" https://api.cicloia.com.br/api/v1/profile`
2. Se retornar 404 ou erro → path `/api/v1/profile` faltando na rule do evo-auth
3. Se retornar dados corretos → problema de cache do navegador (hard refresh)

### "Network Error" no login
1. Verificar se o build do frontend usou URLs de produção (`https://api.cicloia.com.br`) — o CSP bloqueia `http://localhost`
2. Verificar o console do navegador por violações de CSP
3. Adicionar domínios ausentes no `script-src` do `nginx.conf`

### CSP — adicionar novo domínio
Editar `nginx.conf` (linha do `Content-Security-Policy`) e adicionar no directive apropriado:
- Scripts externos: `script-src`
- Conexões (fetch/WS): `connect-src`
- Imagens: `img-src`

## Banco de Dados

### Roles/Permissões via SQL
```sql
INSERT INTO user_roles (user_id, role_id, granted_at, created_at, updated_at)
SELECT u.id, r.id, NOW(), NOW(), NOW()
FROM users u, roles r
WHERE u.email = 'email@example.com' AND r.key = 'super_admin';
```

### Verificar permissões
```sql
SELECT u.email, r.key, r.name
FROM user_roles ur
JOIN users u ON u.id = ur.user_id
JOIN roles r ON r.id = ur.role_id
WHERE u.email = 'email@example.com';
```

## Comunicação
- Responder sempre em português (PT-BR)
