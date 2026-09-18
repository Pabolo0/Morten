<script>
  import { onMount } from 'svelte';
  import { supabase } from '$lib/supabase';

  const sections = {
    luxury: { title: 'Luxo', subtitle: 'Máquinas que ultrapassam expectativas.', colors: 'from-red-950 via-red-900 to-yellow-950' },
    middle: { title: 'Premium', subtitle: 'Tecnologia, conforto e presença.', colors: 'from-green-950 via-blue-950 to-slate-950' },
    economy: { title: 'Essenciais', subtitle: 'Mobilidade inteligente para o dia a dia.', colors: 'from-amber-950 via-stone-900 to-orange-950' }
  };

  let cars = [];
  let loading = true;
  let dark = false;
  let search = '';

  async function loadCars() {
    if (!supabase) { loading = false; return; }
    const { data, error } = await supabase
      .from('cars')
      .select('id, model, year, price, description, image_url, featured, brands(name, category, country, flag_emoji)')
      .eq('available', true)
      .order('featured', { ascending: false })
      .order('price', { ascending: false });
    if (!error) cars = data ?? [];
    loading = false;
  }

  function toggleTheme() {
    dark = !dark;
    document.documentElement.classList.toggle('dark', dark);
  }

  function money(value) {
    return new Intl.NumberFormat('pt-BR', { style: 'currency', currency: 'BRL', maximumFractionDigits: 0 }).format(value);
  }

  function filtered(category) {
    return cars.filter((car) =>
      car.brands?.category === category &&
      `${car.brands?.name} ${car.model}`.toLowerCase().includes(search.toLowerCase())
    );
  }

  onMount(loadCars);
</script>

<svelte:head>
  <title>Morten — Veículos selecionados</title>
  <meta name="description" content="Morten: uma experiência moderna para encontrar seu próximo veículo." />
</svelte:head>

<header class="sticky top-0 z-50 border-b border-black/10 bg-white/85 backdrop-blur-xl dark:border-white/10 dark:bg-black/80">
  <div class="mx-auto flex max-w-7xl items-center justify-between gap-4 px-5 py-4">
    <a href="/" class="text-2xl font-black tracking-[0.25em]">MORTEN</a>
    <div class="hidden items-center gap-6 text-sm font-semibold md:flex">
      <a href="#luxury">Luxo</a><a href="#middle">Premium</a><a href="#economy">Essenciais</a>
    </div>
    <button onclick={toggleTheme} class="rounded-full border border-black/10 px-4 py-2 text-sm dark:border-white/15">
      {dark ? '☀️ Claro' : '🌙 Escuro'}
    </button>
  </div>
</header>

<main>
  <section class="relative overflow-hidden bg-neutral-950 px-5 py-24 text-white">
    <div class="mx-auto max-w-7xl">
      <p class="mb-4 text-sm font-bold uppercase tracking-[0.35em] text-white/60">Morten Automotive</p>
      <h1 class="max-w-4xl text-5xl font-black leading-none md:text-8xl">Seu próximo carro começa aqui.</h1>
      <p class="mt-7 max-w-2xl text-lg text-white/65">Um catálogo criado para separar o extraordinário do comum.</p>
      <div class="mt-10 max-w-xl">
        <input bind:value={search} placeholder="Buscar marca ou modelo..." class="w-full rounded-2xl border border-white/15 bg-white/10 px-5 py-4 outline-none placeholder:text-white/45 focus:border-white/40" />
      </div>
    </div>
  </section>

  {#if loading}
    <div class="mx-auto max-w-7xl px-5 py-16 text-center text-neutral-500">Carregando veículos...</div>
  {:else if !supabase}
    <div class="mx-auto max-w-7xl px-5 py-16 text-center text-neutral-500">Configure as variáveis do Supabase para carregar o catálogo.</div>
  {:else}
    {#each Object.entries(sections) as [category, section]}
      <section id={category} class={`bg-gradient-to-br ${section.colors} px-5 py-16 text-white`}>
        <div class="mx-auto max-w-7xl">
          <div class="mb-10">
            <p class="text-sm font-bold uppercase tracking-[0.3em] text-white/55">Morten</p>
            <h2 class="mt-2 text-4xl font-black md:text-6xl">{section.title}</h2>
            <p class="mt-3 text-white/65">{section.subtitle}</p>
          </div>

          <div class="grid gap-6 md:grid-cols-2 lg:grid-cols-3">
            {#each filtered(category) as car}
              <article class="overflow-hidden rounded-3xl border border-white/10 bg-black/25 shadow-2xl backdrop-blur">
                <div class="flex h-56 items-center justify-center bg-white/10">
                  {#if car.image_url}
                    <img src={car.image_url} alt={car.model} class="h-full w-full object-cover" />
                  {:else}
                    <span class="text-7xl opacity-30">🚘</span>
                  {/if}
                </div>
                <div class="p-6">
                  <div class="flex items-center justify-between gap-3">
                    <span class="text-sm font-bold text-white/70">{car.brands?.flag_emoji} {car.brands?.name}</span>
                    {#if car.featured}<span class="rounded-full bg-white/15 px-3 py-1 text-xs font-bold">DESTAQUE</span>{/if}
                  </div>
                  <h3 class="mt-3 text-2xl font-black">{car.model}</h3>
                  <p class="mt-1 text-sm text-white/55">{car.year} · {car.brands?.country}</p>
                  <p class="mt-4 min-h-10 text-sm text-white/70">{car.description}</p>
                  <div class="mt-6 flex items-end justify-between gap-4">
                    <div><p class="text-xs uppercase tracking-wider text-white/45">A partir de</p><p class="text-xl font-black">{money(car.price)}</p></div>
                    <button class="rounded-xl bg-white px-4 py-3 text-sm font-black text-black">Ver detalhes</button>
                  </div>
                </div>
              </article>
            {:else}
              <p class="text-white/60">Nenhum veículo encontrado.</p>
            {/each}
          </div>
        </div>
      </section>
    {/each}
  {/if}
</main>

<footer class="bg-black px-5 py-10 text-white/50">
  <div class="mx-auto flex max-w-7xl flex-col justify-between gap-4 text-sm md:flex-row">
    <span class="font-black tracking-[0.25em] text-white">MORTEN</span>
    <span>© 2026 Morten Automotive.</span>
  </div>
</footer>