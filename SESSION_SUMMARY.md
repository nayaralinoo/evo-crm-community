# Sessão — 29 Jun 2026

## O que foi feito

### 1. Imagens (Google Drive → Deploy)
- Download das imagens do Google Drive (folder `12mwBIrD0CpKb4GE6M1eY7HFrj1Yly1c0`)
- Sincronizadas para:
  - `public/assets/crm.svg` (logo claro)
  - `public/assets/crm_light.svg` (logo escuro)
  - `public/favicon.svg` (cópia do crm.svg)
  - `public/assets/crm_hover.png` (hover do CRM)
- Backend Rails: `public/crm_hover.png` mantido

### 2. Paleta de Cores (Dark Mode)
Core 5 cores: `#7240da` / `#7782de` / `#7cc2e6` / `#141748` / `#edeffb`

| Nível | Token | Cor |
|---|---|---|
| 0 Canvas | `background` | `237 56% 14%` |
| 1 Sidebar | `sidebar` | `237 56% 18%` (#141748) |
| 2 Cards | `card` | `237 50% 22%` |
| Primary | `primary` | `260 68% 65%` (roxo #7240da) |
| Secondary | `secondary` | `234 60% 67%` (pervenche #7782de) |
| Accent | `accent` | `200 58% 50%` (azul #7cc2e6) |
| Foreground | `foreground` | `240 64% 96%` (#edeffb) |
| Muted text | `muted-foreground` | `240 30% 70%` |
| Destructive | `destructive` | `0 70% 50%` (vermelho extra) |
| Extras | `chart-4` | green, `chart-5` pink |

Arquivo: `src/styles/globals.css`

### 3. Login / Auth
- Senha resetada via `rails runner`: `nayara.lino@gmail.com` / `@nanoleT88`
- Build anterior quebrado por `VITE_API_URL` undefined (`baseURL:"undefined/api/v1"`)
- Corrigido: build manual com `VITE_API_URL=https://api.cicloia.com.br`

### 4. CSV Import — Google Contacts Fields
Backend: `app/services/data_import/contact_manager.rb`
- Mapeia todos os campos Google Contacts para o modelo Contact
- `Name Prefix`/`First Name`/`Middle Name`/`Last Name`/`Name Suffix` → `contact.name`
- `E-mail 1 - Value` → `contact.email`
- `Phone 1 - Value` → `contact.phone_number`
- Demais → `additional_attributes` ou `custom_attributes`
- Compatibilidade retroativa mantida

Sample CSV: `public/downloads/import-contacts-sample.csv`

### 5. Sidebar
- Copyright removido do `Sidebar.tsx`

### 6. Deploy
- `docker compose down -v` (limpeza total)
- `docker compose build --no-cache evo-frontend evo-crm evo-crm-sidekiq`
- `docker compose up -d`
- Todos serviços healthy em `crm.cicloia.com.br`

## Credenciais
- Email: `nayara.lino@gmail.com`
- Senha: `@nanoleT88`
- API: `https://api.cicloia.com.br`

## Arquivos Modificados
- `src/styles/globals.css` — tema dark mode
- `app/services/data_import/contact_manager.rb` — CSV import mappings
- `app/jobs/data_import_job.rb` — company linkage
- `public/downloads/import-contacts-sample.csv` — sample CSV
- `docker-compose.yml` — auth command simplificado
