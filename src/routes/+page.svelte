<script>
  import { onMount } from 'svelte';
  import { supabase } from '$lib/supabase';

  const sections = {
    luxury: { title: 'Luxo', subtitle: 'Máquinas que ultrapassam expectativas.', colors: 'from-red-950 via-red-900 to-yellow-950', accent: 'text-yellow-300' },
    middle: { title: 'Premium', subtitle: 'Tecnologia, conforto e presença.', colors: 'from-green-950 via-blue-950 to-slate-950', accent: 'text-cyan-300' },
    economy: { title: 'Essenciais', subtitle: 'Mobilidade inteligente para o dia a dia.', colors: 'from-amber-950 via-stone-900 to-orange-950', accent: 'text-amber-200' }
  };

  let cars = [];
  let loading = true;
  let dark = false;
  let search = '';
  let selectedCar = null;
  let contactOpen = false;
  let sent = false;
  let sending = false;
  let inquiryError = '';
  let form = { name: '', email: '', phone: '', message: '' };

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

  async function sendInquiry() {
    if (!supabase || !selectedCar) return;
    sending = true;
    inquiryError = '';
    const payload = {
      car_id: selectedCar.id,
      name: form.name.trim(),
      email: form.email.trim(),
      phone: form.phone.trim() || null,
      message: form.message.trim()
    };
    const { error } = await supabase.from('inquiries').insert(payload);
    sending = false;
    if (error) {
      inquiryError = 'Não foi possível enviar agora. Tente novamente em instantes.';
      return;
    }
    sent = true;
    form = { name: '', email: '', phone: '', message: '' };
  }

  function closeDetails() {
    selectedCar = null;
    contactOpen = false;
    sent = false;
    inquiryError = '';
  }

  function handleKeydown(event) {
    if (event.key === 'Escape' && selectedCar) closeDetails();
  }

  onMount(() => {
    loadCars();
    window.addEventListener('keydown', handleKeydown);
    return () => window.removeEventListener('keydown', handleKeydown);
  });
</script>

<svelte:head>
  <title>Morten — Veículos selecionados</title>
  <meta name="description" content="Morten Automotive: catálogo de veículos selecionados, com modelos de luxo, premium e essenciais." />
  <meta name="theme-color" content="#0a0a0a" />
  <meta property="og:title" content="Morten — Veículos selecionados" />
  <meta property="og:description" content="Explore veículos selecionados e encontre seu próximo carro na Morten Automotive." />
  <meta property="og:type" content="website" />
</svelte:head>

<header class="sticky top-0 z-50 border-b border-black/10 bg-white/90 backdrop-blur-xl dark:border-white/10 dark:bg-black/85">
  <div class="mx-auto flex max-w-7xl items-center justify-between gap-4 px-5 py-4">
    <a href="/" class="text-2xl font-black tracking-[0.25em]">MORTEN</a>
    <nav class="hidden items-center gap-7 text-sm font-semibold md:flex">
      <a href="#luxury" class="transition-opacity hover:opacity-60">Luxo</a>
      <a href="#middle" class="transition-opacity hover:opacity-60">Premium</a>
      <a href="#economy" class="transition-opacity hover:opacity-60">Essenciais</a>
    </nav>
    <button onclick={toggleTheme} aria-label="Alternar tema" class="rounded-full border border-black/10 px-4 py-2 text-sm font-bold transition hover:bg-black hover:text-white dark:border-white/15 dark:hover:bg-white dark:hover:text-black">
      {dark ? '☀️ Claro' : '🌙 Escuro'}
    </button>
  </div>
</header>

<main>
  <section class="relative overflow-hidden bg-neutral-950 px-5 py-24 text-white md:py-32">
    <div class="absolute -right-32 -top-32 h-96 w-96 rounded-full bg-white/10 blur-3xl"></div>
    <div class="mx-auto max-w-7xl">
      <p class="mb-4 text-sm font-bold uppercase tracking-[0.35em] text-white/50">Morten Automotive</p>
      <h1 class="max-w-5xl text-5xl font-black leading-[0.95] md:text-8xl">Seu próximo carro começa aqui.</h1>
      <p class="mt-7 max-w-2xl text-lg text-white/60">Explore veículos selecionados, compare categorias e encontre um modelo que combine com você.</p>
      <div class="mt-10 max-w-2xl">
        <div class="flex items-center gap-3 rounded-2xl border border-white/15 bg-white/10 px-5 py-4 focus-within:border-white/40">
          <span class="text-xl">⌕</span>
          <input bind:value={search} placeholder="Buscar marca ou modelo..." class="w-full bg-transparent outline-none placeholder:text-white/40" />
          {#if search}<button onclick={() => search = ''} class="text-white/50 hover:text-white">×</button>{/if}
        </div>
      </div>
      <div class="mt-8 flex flex-wrap gap-3 text-xs font-bold uppercase tracking-wider text-white/45">
        <span>{cars.length} veículos disponíveis</span><span>•</span><span>9 marcas</span><span>•</span><span>Compra segura</span>
      </div>
    </div>
  </section>

  {#if loading}
    <div class="mx-auto max-w-7xl px-5 py-20 text-center text-neutral-500">Carregando veículos...</div>
  {:else if !supabase}
    <div class="mx-auto max-w-7xl px-5 py-20 text-center text-neutral-500">Configure as variáveis do Supabase para carregar o catálogo.</div>
  {:else}
    {#each Object.entries(sections) as [category, section]}
      <section id={category} class={`bg-gradient-to-br ${section.colors} px-5 py-16 text-white md:py-20`}>
        <div class="mx-auto max-w-7xl">
          <div class="mb-10 flex items-end justify-between gap-6">
            <div>
              <p class={`text-sm font-bold uppercase tracking-[0.3em] ${section.accent} opacity-80`}>Morten</p>
              <h2 class="mt-2 text-4xl font-black md:text-6xl">{section.title}</h2>
              <p class="mt-3 text-white/55">{section.subtitle}</p>
            </div>
            <span class="hidden rounded-full border border-white/10 bg-white/5 px-4 py-2 text-sm font-bold sm:block">{filtered(category).length} modelos</span>
          </div>

          <div class="grid gap-6 md:grid-cols-2 lg:grid-cols-3">
            {#each filtered(category) as car}
              <article class="group overflow-hidden rounded-3xl border border-white/10 bg-black/25 shadow-2xl backdrop-blur transition duration-300 hover:-translate-y-1 hover:bg-black/35">
                <div class="relative flex h-56 items-center justify-center overflow-hidden bg-white/10">
                  {#if car.image_url}
                    <img src={car.image_url} alt={`${car.brands?.name ?? "Morten"} ${car.model}`} loading="lazy" class="h-full w-full object-cover transition duration-500 group-hover:scale-105" />
                  {:else}
                    <div class="text-center"><span class="text-7xl opacity-25">🚘</span><p class="mt-2 text-xs text-white/30">Foto em breve</p></div>
                  {/if}
                  {#if car.featured}
                    <span class="absolute left-4 top-4 rounded-full bg-white px-3 py-1 text-[10px] font-black tracking-wider text-black">DESTAQUE</span>
                  {/if}
                </div>
                <div class="p-6">
                  <div class="flex items-center justify-between gap-3">
                    <span class="text-sm font-bold text-white/65">{car.brands?.flag_emoji} {car.brands?.name}</span>
                    <span class="text-xs text-white/40">{car.year}</span>
                  </div>
                  <h3 class="mt-3 text-2xl font-black">{car.model}</h3>
                  <p class="mt-1 text-sm text-white/50">{car.brands?.country}</p>
                  <p class="mt-4 min-h-10 text-sm leading-6 text-white/65">{car.description}</p>
                  <div class="mt-6 flex items-end justify-between gap-4 border-t border-white/10 pt-5">
                    <div><p class="text-[10px] uppercase tracking-wider text-white/40">A partir de</p><p class="text-xl font-black">{money(car.price)}</p></div>
                    <button onclick={() => selectedCar = car} class="rounded-xl bg-white px-4 py-3 text-sm font-black text-black transition hover:scale-105">Ver detalhes</button>
                  </div>
                </div>
              </article>
            {:else}
              <p class="text-white/60">Nenhum veículo encontrado nesta categoria.</p>
            {/each}
          </div>
        </div>
      </section>
    {/each}
  {/if}
</main>

{#if selectedCar}
  <div class="fixed inset-0 z-[100] flex items-center justify-center bg-black/75 p-4 backdrop-blur-sm" role="presentation" aria-label="Detalhes do veículo" onclick={(e) => e.target === e.currentTarget && (selectedCar = null)}>
    <div class="max-h-[90vh] w-full max-w-2xl overflow-auto rounded-3xl bg-white text-neutral-950 shadow-2xl dark:bg-neutral-900 dark:text-white">
      <div class="relative flex h-64 items-center justify-center overflow-hidden bg-neutral-100 dark:bg-neutral-800">
        {#if selectedCar.image_url}<img src={selectedCar.image_url} alt={selectedCar.model} class="h-full w-full object-cover" />{:else}<span class="text-8xl opacity-20">🚘</span>{/if}
        <button onclick={closeDetails} aria-label="Fechar" class="absolute right-4 top-4 rounded-full bg-black/60 px-4 py-2 text-xl text-white">×</button>
      </div>
      <div class="p-7 md:p-9">
        <p class="text-sm font-bold text-neutral-500">{selectedCar.brands?.flag_emoji} {selectedCar.brands?.name} · {selectedCar.year}</p>
        <h2 class="mt-2 text-4xl font-black">{selectedCar.model}</h2>
        <p class="mt-4 leading-7 text-neutral-600 dark:text-white/60">{selectedCar.description}</p>
        <div class="mt-7 rounded-2xl bg-neutral-100 p-5 dark:bg-white/5">
          <p class="text-xs uppercase tracking-wider text-neutral-500">Preço</p>
          <p class="mt-1 text-3xl font-black">{money(selectedCar.price)}</p>
        </div>
        {#if !contactOpen}
          <button onclick={() => contactOpen = true} class="mt-6 w-full rounded-2xl bg-neutral-950 px-5 py-4 font-black text-white transition hover:opacity-85 dark:bg-white dark:text-black">Tenho interesse</button>
        {:else if sent}
          <div class="mt-6 rounded-2xl bg-emerald-100 p-5 text-emerald-900 dark:bg-emerald-950/40 dark:text-emerald-200">
            <p class="font-black">Interesse enviado!</p>
            <p class="mt-1 text-sm opacity-80">Recebemos sua mensagem sobre este veículo.</p>
          </div>
        {:else}
          <form onsubmit={(e) => { e.preventDefault(); sendInquiry(); }} class="mt-6 space-y-3">
            <input required bind:value={form.name} placeholder="Seu nome" class="w-full rounded-xl border border-black/10 bg-neutral-50 px-4 py-3 outline-none dark:border-white/10 dark:bg-white/5" />
            <input required type="email" bind:value={form.email} placeholder="Seu e-mail" class="w-full rounded-xl border border-black/10 bg-neutral-50 px-4 py-3 outline-none dark:border-white/10 dark:bg-white/5" />
            <input bind:value={form.phone} placeholder="Telefone (opcional)" class="w-full rounded-xl border border-black/10 bg-neutral-50 px-4 py-3 outline-none dark:border-white/10 dark:bg-white/5" />
            {#if inquiryError}<p class="rounded-xl bg-red-100 p-3 text-sm text-red-700 dark:bg-red-950/40 dark:text-red-200">{inquiryError}</p>{/if}
            <textarea required bind:value={form.message} rows="4" placeholder="Olá, tenho interesse neste veículo..." class="w-full resize-none rounded-xl border border-black/10 bg-neutral-50 px-4 py-3 outline-none dark:border-white/10 dark:bg-white/5"></textarea>
            <button disabled={sending} class="w-full rounded-xl bg-neutral-950 px-5 py-4 font-black text-white disabled:opacity-50 dark:bg-white dark:text-black">{sending ? 'Enviando...' : 'Enviar interesse'}</button>
          </form>
        {/if}
      </div>
    </div>
  </div>
{/if}

<footer class="bg-black px-5 py-10 text-white/50">
  <div class="mx-auto flex max-w-7xl flex-col justify-between gap-4 text-sm md:flex-row">
    <span class="font-black tracking-[0.25em] text-white">MORTEN</span>
    <span>© 2026 Morten Automotive.</span><a href="/admin" class="font-bold transition hover:text-white">Área administrativa</a>
  </div>
</footer>