# Morten

Site de catálogo automotivo da Morten, feito com SvelteKit, Tailwind CSS e Supabase.

## Rodar localmente

1. Instale Node.js.
2. Execute `npm install`.
3. Copie `.env.example` para `.env`.
4. Preencha `VITE_SUPABASE_PUBLISHABLE_KEY` com a chave publishable do projeto Supabase.
5. Execute `npm run dev`.

O catálogo lê os veículos diretamente da tabela `public.cars` e as marcas de `public.brands`.

## Estrutura

- Luxo: Lamborghini, Rolls-Royce e Bugatti.
- Premium: Chevrolet, Toyota e Fiat.
- Essenciais: Renault, Citroën e Volkswagen.
- Tema claro/escuro.
- Busca por marca/modelo.
- Campo preparado para fotos via URL/Storage.
