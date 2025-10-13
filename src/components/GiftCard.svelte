<script lang="ts">
    import { fade } from "svelte/transition";
    export let gift: any;
    export let onClaim: (id: string) => Promise<void>;
    export let onUnclaim: (id: string) => Promise<void>;
    export let who: string | null = null;

    let busy = false;

    async function handleClaim() {
        if (gift.claimed_by || busy) return;
        busy = true;
        try {
            await onClaim(gift.id);
        } finally {
            busy = false;
        }
    }
    async function handleUnclaim() {
        if (!who || busy) return;
        busy = true;
        try {
            await onUnclaim(gift.id);
        } finally {
            busy = false;
        }
    }

    // Normaliza comparación
    const norm = (s?: string | null) => (s ?? "").trim();

    // 🎨 Colores por persona
    const colors: Record<string, { card: string; badge: string }> = {
        Ainara: {
            card: "border-pink-300 bg-pink-50",
            badge: "bg-pink-50 text-pink-800 border-pink-200",
        },
        Conchi: {
            card: "border-emerald-300 bg-emerald-50",
            badge: "bg-emerald-50 text-emerald-800 border-emerald-200",
        },
        "Celia del Carmen": {
            card: "border-sky-300 bg-sky-50",
            badge: "bg-sky-50 text-sky-800 border-sky-200",
        },
    };

    function cardClass() {
        const owner = norm(gift.claimed_by);
        if (!owner) return "border-zinc-200 bg-white";
        if (owner === norm(who)) return "border-amber-400 bg-amber-50"; // 💛 mis reservas
        return colors[owner]?.card ?? "border-zinc-300 bg-zinc-50";
    }
    function badgeClass() {
        const owner = norm(gift.claimed_by);
        if (owner === norm(who))
            return "bg-amber-50 text-amber-800 border-amber-200";
        return (
            colors[owner]?.badge ?? "bg-zinc-50 text-zinc-700 border-zinc-200"
        );
    }
</script>

<!-- MOBILE-FIRST: apila, y en md+ pasa a fila con controles a la derecha -->
<div
    in:fade
    class={`w-full rounded-3xl border p-4 md:p-5 backdrop-blur
            flex flex-col md:flex-row gap-3 md:gap-4 transition-all
            hover:shadow-lg hover:-translate-y-0.5 ${cardClass()}`}
>
    <!-- Bloque superior: imagen + controles móviles debajo -->
    <div class="flex items-start gap-3">
        {#if gift.image_url}
            <img
                src={gift.image_url}
                alt={gift.title}
                class="w-24 h-24 md:w-28 md:h-28 object-cover rounded-2xl border shrink-0"
            />
        {/if}

        <!-- Controles SOLO en móvil (debajo de la imagen visualmente) -->
        <div class="md:hidden flex flex-col items-start gap-2">
            {#if gift.claimed_by}
                <span
                    class={`text-[11px] px-2 py-1 rounded-full border ${badgeClass()}`}
                >
                    Reservado por <strong>{gift.claimed_by}</strong>
                </span>
            {/if}

            {#if gift.claimed_by}
                {#if who && norm(who) === norm(gift.claimed_by)}
                    <button
                        class="px-3 py-2 rounded-xl text-sm font-medium bg-white border hover:bg-zinc-50 disabled:opacity-60"
                        on:click={handleUnclaim}
                        disabled={busy}>Quitar reserva</button
                    >
                {:else}
                    <button
                        class="px-3 py-2 rounded-xl text-sm font-medium bg-zinc-300 text-zinc-700 cursor-not-allowed"
                        disabled>Reservado</button
                    >
                {/if}
            {:else}
                <button
                    class="px-3 py-2 rounded-xl text-sm font-medium bg-zinc-900 text-white hover:bg-zinc-800 disabled:opacity-60"
                    on:click={handleClaim}
                    disabled={busy}>{busy ? "Reservando…" : "Reservar"}</button
                >
            {/if}
        </div>
    </div>

    <!-- Contenido principal: TÍTULO primero (ya no se tapa en móvil) -->
    <div class="flex-1 min-w-0">
        <h3 class="font-semibold text-lg md:text-xl leading-tight line-clamp-2">
            {gift.title}
        </h3>

        {#if gift.notes}
            <p class="text-sm opacity-80 mt-1 line-clamp-3">{gift.notes}</p>
        {/if}

        <div class="flex flex-wrap gap-2 mt-2 text-xs md:text-sm opacity-80">
            {#if gift.price}
                <span
                    class="inline-flex items-center gap-1 px-2 py-0.5 rounded-full bg-zinc-100 border"
                >
                    ~ {gift.price}€
                </span>
            {/if}
            {#if gift.url}
                <a
                    href={gift.url}
                    target="_blank"
                    rel="noreferrer"
                    class="underline">Ver enlace</a
                >
            {/if}

            <!-- Varias categorías o una sola -->
            {#if gift.category && Array.isArray(gift.category)}
                {#each gift.category as c}
                    <span
                        class="inline-flex items-center gap-1 px-2 py-0.5 rounded-full bg-amber-50 border border-amber-200"
                        >{c}</span
                    >
                {/each}
            {:else if gift.category}
                <span
                    class="inline-flex items-center gap-1 px-2 py-0.5 rounded-full bg-amber-50 border border-amber-200"
                    >{gift.category}</span
                >
            {/if}
        </div>
    </div>

    <!-- Controles SOLO en escritorio -->
    <div class="hidden md:flex flex-col items-end gap-2">
        {#if gift.claimed_by}
            <span
                class={`text-[11px] md:text-xs px-2 py-1 rounded-full border ${badgeClass()}`}
            >
                Reservado por <strong>{gift.claimed_by}</strong>
            </span>
        {/if}

        {#if gift.claimed_by}
            {#if who && norm(who) === norm(gift.claimed_by)}
                <button
                    class="px-3 py-2 rounded-xl text-sm font-medium bg-white border hover:bg-zinc-50 disabled:opacity-60"
                    on:click={handleUnclaim}
                    disabled={busy}>Quitar reserva</button
                >
            {:else}
                <button
                    class="px-3 py-2 rounded-xl text-sm font-medium bg-zinc-300 text-zinc-700 cursor-not-allowed"
                    disabled>Reservado</button
                >
            {/if}
        {:else}
            <button
                class="px-3 py-2 rounded-xl text-sm font-medium bg-zinc-900 text-white hover:bg-zinc-800 disabled:opacity-60"
                on:click={handleClaim}
                disabled={busy}>{busy ? "Reservando…" : "Reservar"}</button
            >
        {/if}
    </div>
</div>
