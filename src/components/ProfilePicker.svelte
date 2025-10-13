<script lang="ts">
    import { onMount } from "svelte";
    export let current: string | null = null;
    export let onPick: (name: string) => void;

    let profiles: { name: string; avatar?: string }[] = [];

    onMount(() => {
        const raw = import.meta.env.PUBLIC_FAMILY_PROFILES || "";
        profiles = raw
            .split(";")
            .filter(Boolean)
            .map((p: { split: (arg0: string) => [any, any] }) => {
                const [name] = p.split(":");
                return { name };
            });
    });
</script>

<div class="mb-2 text-sm font-medium opacity-80">¿Quién eres?</div>

<div
    class="flex gap-2 overflow-x-auto pb-2"
    role="tablist"
    aria-label="Selector de perfil"
>
    {#each profiles as p}
        <button
            type="button"
            role="tab"
            aria-selected={current === p.name}
            class="shrink-0 px-3 py-2 rounded-2xl border bg-white/80
                   flex items-center gap-2 transition-all
                   hover:shadow-sm hover:-translate-y-0.5
                   data-[active=true]:bg-gradient-to-r data-[active=true]:from-amber-100 data-[active=true]:to-pink-100
                   data-[active=true]:border-amber-300 data-[active=true]:shadow"
            data-active={current === p.name}
            on:click={() => onPick(p.name)}
        >
            {#if p.avatar}
                <img
                    src={p.avatar}
                    alt={p.name}
                    class="w-7 h-7 rounded-full ring-2"
                    class:ring-amber-300={current === p.name}
                    class:ring-transparent={current !== p.name}
                />
            {/if}
            <span class="text-sm font-medium">{p.name}</span>
        </button>
    {/each}
</div>
