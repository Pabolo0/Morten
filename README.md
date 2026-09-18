# Morten

Site de catálogo automotivo da Morten, feito com SvelteKit, Svelte, Tailwind CSS e Supabase.

## Stack

- SvelteKit + Svelte 5
- Tailwind CSS 4
- Supabase (Database + Auth)
- Vite

## Rodar localmente

Use **Node.js 22 ou superior**.

1. Clone o repositório.
2. Execute `npm install`.
3. Copie `.env.example` para `.env`.
4. Preencha:
   - `VITE_SUPABASE_URL`
   - `VITE_SUPABASE_PUBLISHABLE_KEY`
5. Execute `npm run dev`.

Nunca coloque uma chave **service_role/secret** no frontend.

## Supabase

O catálogo usa as tabelas `brands` e `cars`. Os formulários públicos criam registros em `inquiries`.

A área `/admin` usa Supabase Auth. Para autorizar uma conta, o usuário precisa existir no Auth e também ter seu UUID cadastrado em `public.admins`.

Exemplo, executado no SQL Editor do Supabase após criar a conta:

```sql
insert into public.admins (user_id)
values ('UUID_DO_USUARIO');
```

O RLS deve permanecer habilitado. A autorização administrativa é feita pela tabela `admins`, não por `user_metadata`.

## Funcionalidades

- Catálogo por categoria: Luxo, Premium e Essenciais.
- Busca por marca e modelo.
- Tema claro/escuro.
- Modal de detalhes do veículo.
- Formulário de interesse conectado ao Supabase.
- Painel administrativo para cadastrar, editar, excluir e ocultar veículos.
- Gerenciamento do status dos interesses.
- Favicon, página de erro e robots.txt.
- Sitemap preparado para produção.

## Veículos iniciais

- **Luxo:** Lamborghini, Rolls-Royce e Bugatti.
- **Premium:** Chevrolet, Toyota e Fiat.
- **Essenciais:** Renault, Citroën e Volkswagen.

## Build

```bash
npm install
npm run build
npm run preview
```

O projeto também deve ser validado em CI antes de uma publicação de produção.
