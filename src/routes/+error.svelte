<script>
  import { page } from '$app/state';
</script>

<svelte:head>
  <title>Página não encontrada — Morten</title>
  <meta name="description" content="A página que você procura não foi encontrada." />
</svelte:head>

<main class="flex min-h-screen items-center justify-center bg-neutral-950 px-5 text-white">
  <section class="max-w-xl text-center">
    <p class="text-sm font-black uppercase tracking-[0.4em] text-white/40">Morten</p>
    <h1 class="mt-5 text-8xl font-black">{page.status}</h1>
    <h2 class="mt-3 text-3xl font-black">Página não encontrada</h2>
    <p class="mt-4 text-white/50">O endereço pode ter sido alterado ou não existe mais.</p>
    <a href="/" class="mt-8 inline-flex rounded-2xl bg-white px-6 py-4 font-black text-black transition hover:scale-105">Voltar para o início</a>
  </section>
</main>