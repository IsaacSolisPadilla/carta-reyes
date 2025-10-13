<script lang="ts">
    import { onMount } from "svelte";
    import { supabase } from "../lib/supabase";
    import GiftCard from "./GiftCard.svelte";

    export let who: string | null = null;

    type Category = "Todos" | "Reyes" | "PapaNoel";
    let category: Category = "Todos";

    // Guardamos TODO para totales globales
    let allGifts: any[] = [];
    let gifts: any[] = []; // visibles según filtro
    let loading = true;

    // Perfiles desde ENV (mismo formato que el ProfilePicker: "Nombre[:avatar];Nombre2[:avatar]")
    type Profile = { name: string };
    let profiles: Profile[] = [];

    onMount(async () => {
        const raw = (import.meta.env.PUBLIC_FAMILY_PROFILES ?? "").toString();
        profiles = raw
            .split(";")
            .map((s: string) => s.trim())
            .filter(Boolean)
            .map((entry: string) => {
                const [namePart] = entry.split(":");
                const name = (namePart ?? "").trim();
                return { name };
            }) as Profile[];

        await load();
    });

    async function load() {
        loading = true;
        const { data, error } = await supabase
            .from("gifts")
            .select("*")
            .order("created_at", { ascending: true });

        if (error) {
            alert(error.message);
            loading = false;
            return;
        }

        allGifts = data ?? [];
        applyFilter();
        loading = false;
    }

    function applyFilter() {
        if (category === "Todos") {
            gifts = allGifts;
        } else {
            gifts = allGifts.filter((g) => g.category === category);
        }
    }

    // Re-aplica filtro al cambiar categoría
    $: category, applyFilter();

    // ------------ Reservar / Quitar reserva (sin RPC) ------------
    async function claim(id: string) {
        if (!who) {
            alert("Elige tu perfil primero");
            return;
        }

        const { error } = await supabase
            .from("gifts")
            .update({ claimed_by: who })
            .eq("id", id)
            .is("claimed_by", null);

        if (error) alert(error.message);
        await load();
    }

    async function unclaim(id: string) {
        if (!who) {
            alert("Elige tu perfil primero");
            return;
        }

        const { error } = await supabase
            .from("gifts")
            .update({ claimed_by: null })
            .eq("id", id)
            .eq("claimed_by", who);

        if (error) alert(error.message);
        await load();
    }

    // ------------ Totales por persona ------------
    // Convierte cualquier formato "12", "12,5", "12.50", "12-15" a número aproximado.
    function parsePrice(v: any): number {
        if (v == null) return 0;
        if (typeof v === "number" && isFinite(v)) return v;
        const s = String(v);
        // toma el primer número (con , o .)
        const m = s.replace(",", ".").match(/(\d+(\.\d+)?)/);
        return m ? parseFloat(m[1]) : 0;
    }

    // Mapa de totales por claimed_by
    $: totalsByPerson = allGifts.reduce<Record<string, number>>((acc, g) => {
        const whoClaimed = (g.claimed_by ?? "").trim();
        if (!whoClaimed) return acc;
        const price = parsePrice(g.price);
        acc[whoClaimed] =
            (acc[whoClaimed] ?? 0) + (isFinite(price) ? price : 0);
        return acc;
    }, {});

    const fmt = (n: number) =>
        new Intl.NumberFormat("es-ES", {
            style: "currency",
            currency: "EUR",
            maximumFractionDigits: 0,
        }).format(Math.round(n));
</script>

<!-- Resumen de totales (“cesta” por perfil) -->
{#if profiles.length > 0}
    <div class="flex items-center justify-between flex-wrap gap-2 mt-2">
        <div class="text-sm font-medium opacity-80">Totales reservados</div>
        <div class="flex gap-2 flex-wrap">
            {#each profiles as p}
                {#if (totalsByPerson[p.name] ?? 0) > 0}
                    <span
                        class="inline-flex items-center gap-2 px-3 py-1 rounded-2xl border text-sm
                               bg-white/80"
                        class:border-amber-400={who &&
                            who.trim() === p.name.trim()}
                        class:ring-1={who && who.trim() === p.name.trim()}
                        class:ring-amber-300={who &&
                            who.trim() === p.name.trim()}
                        title={`Total reservado por ${p.name}`}
                    >
                        <span class="font-medium">{p.name}</span>
                        <span class="opacity-80"
                            >· {fmt(totalsByPerson[p.name])}</span
                        >
                    </span>
                {/if}
            {/each}
            <!-- Si el que navega no está en profiles, igual muestra su total -->
            {#if who && !profiles.some((pp) => pp.name.trim() === who.trim()) && (totalsByPerson[who] ?? 0) > 0}
                <span
                    class="inline-flex items-center gap-2 px-3 py-1 rounded-2xl border text-sm bg-white/80 border-amber-400 ring-1 ring-amber-300"
                >
                    <span class="font-medium">{who}</span>
                    <span class="opacity-80">· {fmt(totalsByPerson[who])}</span>
                </span>
            {/if}
        </div>
    </div>
{/if}

<!-- Filtro por categoría -->
<div class="flex items-center gap-2 my-4">
    <label class="text-sm opacity-80">Filtrar por:</label>
    <select
        bind:value={category}
        class="border rounded-xl px-3 py-2 text-sm bg-white/80"
        aria-label="Filtrar regalos por categoría"
    >
        <option value="Todos">Todos</option>
        <option value="Reyes">Reyes</option>
        <option value="PapaNoel">PapaNoel</option>
    </select>
</div>

<!-- Lista de regalos -->
{#if loading}
    <p class="animate-pulse text-sm opacity-70">Cargando…</p>
{:else if gifts.length === 0}
    <p>No hay regalos aún.</p>
{:else}
    <!-- 1 card por fila -->
    <div class="flex flex-col gap-4">
        {#each gifts as g (g.id)}
            <GiftCard gift={g} onClaim={claim} onUnclaim={unclaim} {who} />
        {/each}
    </div>
{/if}
