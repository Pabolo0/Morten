<script>
  import { onMount } from 'svelte';
  import { supabase } from '$lib/supabase';

  let session = null, loading = true, authorized = false;
  let email = '', password = '', authError = '', message = '';
  let cars = [], brands = [], inquiries = [], editing = null;
  let form = { brand_id: '', model: '', year: 2026, price: '', description: '', image_url: '', featured: false, available: true };

  async function checkUser() {
    if (!supabase) { loading = false; return; }
    const { data } = await supabase.auth.getSession();
    session = data.session;
    if (session) {
      const { data: admin } = await supabase.from('admins').select('user_id').eq('user_id', session.user.id).maybeSingle();
      authorized = !!admin;
      if (authorized) await loadAll();
    }
    loading = false;
  }

  async function login() {
    authError = '';
    const { data, error } = await supabase.auth.signInWithPassword({ email, password });
    if (error) { authError = error.message; return; }
    session = data.session;
    await checkUser();
  }

  async function logout() {
    await supabase.auth.signOut();
    session = null;
    authorized = false;
  }

  async function loadAll() {
    const [c, b, i] = await Promise.all([
      supabase.from('cars').select('id, brand_id, model, year, price, description, image_url, featured, available, brands(name, category, flag_emoji)').order('created_at', { ascending: false }),
      supabase.from('brands').select('id, name, category, flag_emoji').order('category').order('name'),
      supabase.from('inquiries').select('id, car_id, name, email, phone, message, status, created_at, cars(model)').order('created_at', { ascending: false })
    ]);
    cars = c.data ?? [];
    brands = b.data ?? [];
    inquiries = i.data ?? [];
  }

  function resetForm() {
    editing = null;
    form = { brand_id: brands[0]?.id ?? '', model: '', year: 2026, price: '', description: '', image_url: '', featured: false, available: true };
  }

  function editCar(car) {
    editing = car.id;
    form = { brand_id: car.brand_id, model: car.model, year: car.year, price: car.price, description: car.description ?? '', image_url: car.image_url ?? '', featured: car.featured, available: car.available };
    window.scrollTo({ top: 0, behavior: 'smooth' });
  }

  async function saveCar() {
    message = '';
    const payload = { ...form, brand_id: Number(form.brand_id), year: Number(form.year), price: Number(form.price) };
    const result = editing ? await supabase.from('cars').update(payload).eq('id', editing) : await supabase.from('cars').insert(payload);
    if (result.error) { message = result.error.message; return; }
    message = editing ? 'Veículo atualizado.' : 'Veículo cadastrado.';
    resetForm();
    await loadAll();
  }

  async function deleteCar(id) {
    if (!confirm('Excluir este veículo?')) return;
    const { error } = await supabase.from('cars').delete().eq('id', id);
    if (error) message = error.message; else await loadAll();
  }

  async function updateInquiry(id, status) {
    await supabase.from('inquiries').update({ status }).eq('id', id);
    await loadAll();
  }

  onMount(checkUser);
</script>

<svelte:head><title>Morten Admin</title></svelte:head>

<div class="min-h-screen bg-neutral-100 text-neutral-950 dark:bg-neutral-950 dark:text-white">
  <header class="border-b border-black/10 bg-white/80 px-5 py-5 backdrop-blur dark:border-white/10 dark:bg-black/70">
    <div class="mx-auto flex max-w-7xl items-center justify-between">
      <a href="/" class="text-2xl font-black tracking-[0.25em]">MORTEN</a>
      {#if session}<button onclick={logout} class="rounded-xl border px-4 py-2 text-sm font-bold">Sair</button>{/if}
    </div>
  </header>

  <main class="mx-auto max-w-7xl px-5 py-10">
    {#if loading}
      <p>Carregando painel...</p>
    {:else if !session}
      <section class="mx-auto max-w-md rounded-3xl bg-white p-7 shadow-xl dark:bg-neutral-900">
        <p class="text-xs font-bold uppercase tracking-[0.3em] text-neutral-500">Morten</p>
        <h1 class="mt-2 text-4xl font-black">Painel administrativo</h1>
        <p class="mt-3 text-sm text-neutral-500">Entre com uma conta autorizada.</p>
        <form onsubmit={(e) => { e.preventDefault(); login(); }} class="mt-7 space-y-4">
          <input required type="email" bind:value={email} placeholder="E-mail" class="w-full rounded-xl border p-3 dark:border-white/10 dark:bg-white/5" />
          <input required type="password" bind:value={password} placeholder="Senha" class="w-full rounded-xl border p-3 dark:border-white/10 dark:bg-white/5" />
          {#if authError}<p class="rounded-xl bg-red-100 p-3 text-sm text-red-700">{authError}</p>{/if}
          <button class="w-full rounded-xl bg-black px-4 py-3 font-black text-white dark:bg-white dark:text-black">Entrar</button>
        </form>
      </section>
    {:else if !authorized}
      <section class="mx-auto max-w-xl rounded-3xl bg-white p-8 shadow-xl dark:bg-neutral-900">
        <h1 class="text-3xl font-black">Acesso não autorizado</h1>
        <p class="mt-3 text-neutral-500">Sua conta está autenticada, mas ainda não foi cadastrada como administradora.</p>
      </section>
    {:else}
      <div class="grid gap-8 lg:grid-cols-[380px_1fr]">
        <section class="rounded-3xl bg-white p-6 shadow-xl dark:bg-neutral-900">
          <div class="flex items-center justify-between">
            <h1 class="text-2xl font-black">{editing ? 'Editar veículo' : 'Novo veículo'}</h1>
            {#if editing}<button onclick={resetForm} class="text-sm font-bold text-neutral-500">Cancelar</button>{/if}
          </div>
          <form onsubmit={(e) => { e.preventDefault(); saveCar(); }} class="mt-6 space-y-3">
            <select required bind:value={form.brand_id} class="w-full rounded-xl border p-3 dark:border-white/10 dark:bg-white/5">
              <option value="">Marca</option>
              {#each brands as brand}<option value={brand.id}>{brand.flag_emoji} {brand.name} · {brand.category}</option>{/each}
            </select>
            <input required bind:value={form.model} placeholder="Modelo" class="w-full rounded-xl border p-3 dark:border-white/10 dark:bg-white/5" />
            <div class="grid grid-cols-2 gap-3">
              <input required type="number" bind:value={form.year} placeholder="Ano" class="rounded-xl border p-3 dark:border-white/10 dark:bg-white/5" />
              <input required type="number" min="0" step="0.01" bind:value={form.price} placeholder="Preço" class="rounded-xl border p-3 dark:border-white/10 dark:bg-white/5" />
            </div>
            <textarea required bind:value={form.description} rows="4" placeholder="Descrição" class="w-full rounded-xl border p-3 dark:border-white/10 dark:bg-white/5"></textarea>
            <input bind:value={form.image_url} placeholder="URL da foto" class="w-full rounded-xl border p-3 dark:border-white/10 dark:bg-white/5" />
            <label class="flex items-center gap-2 text-sm"><input type="checkbox" bind:checked={form.featured} /> Destacar veículo</label>
            <label class="flex items-center gap-2 text-sm"><input type="checkbox" bind:checked={form.available} /> Disponível no catálogo</label>
            <button class="w-full rounded-xl bg-black px-4 py-3 font-black text-white dark:bg-white dark:text-black">{editing ? 'Salvar alterações' : 'Cadastrar veículo'}</button>
          </form>
          {#if message}<p class="mt-4 rounded-xl bg-neutral-100 p-3 text-sm dark:bg-white/5">{message}</p>{/if}
        </section>

        <div class="space-y-8">
          <section class="rounded-3xl bg-white p-6 shadow-xl dark:bg-neutral-900">
            <div class="flex items-center justify-between"><h2 class="text-2xl font-black">Veículos</h2><span class="text-sm text-neutral-500">{cars.length} cadastrados</span></div>
            <div class="mt-5 space-y-3">
              {#each cars as car}
                <article class="flex flex-col gap-4 rounded-2xl border border-black/10 p-4 md:flex-row md:items-center md:justify-between dark:border-white/10">
                  <div><p class="text-xs text-neutral-500">{car.brands?.flag_emoji} {car.brands?.name} · {car.year}</p><h3 class="text-lg font-black">{car.model}</h3><p class="text-sm font-bold">{new Intl.NumberFormat('pt-BR',{style:'currency',currency:'BRL'}).format(car.price)}</p></div>
                  <div class="flex gap-2"><button onclick={() => editCar(car)} class="rounded-xl border px-4 py-2 text-sm font-bold">Editar</button><button onclick={() => deleteCar(car.id)} class="rounded-xl bg-red-600 px-4 py-2 text-sm font-bold text-white">Excluir</button></div>
                </article>
              {/each}
            </div>
          </section>

          <section class="rounded-3xl bg-white p-6 shadow-xl dark:bg-neutral-900">
            <div class="flex items-center justify-between"><h2 class="text-2xl font-black">Interesses</h2><span class="text-sm text-neutral-500">{inquiries.length} recebidos</span></div>
            <div class="mt-5 space-y-3">
              {#each inquiries as item}
                <article class="rounded-2xl border border-black/10 p-4 dark:border-white/10">
                  <div class="flex flex-wrap items-center justify-between gap-2"><h3 class="font-black">{item.name}</h3><select value={item.status} onchange={(e) => updateInquiry(item.id, e.currentTarget.value)} class="rounded-lg border px-2 py-1 text-xs dark:border-white/10 dark:bg-white/5"><option value="new">Novo</option><option value="contacted">Contatado</option><option value="closed">Fechado</option></select></div>
                  <p class="mt-1 text-sm text-neutral-500">{item.email}{item.phone ? ' · ' + item.phone : ''}</p>
                  <p class="mt-2 text-sm">{item.cars?.model ?? 'Veículo não informado'}</p>
                  <p class="mt-2 text-sm leading-6 text-neutral-600 dark:text-white/60">{item.message}</p>
                </article>
              {:else}<p class="text-sm text-neutral-500">Nenhum interesse recebido ainda.</p>{/each}
            </div>
          </section>
        </div>
      </div>
    {/if}
  </main>
</div>