<script lang="ts">
    import ProfilePicker from "./ProfilePicker.svelte";
    import GiftList from "./GiftList.svelte";
    let who: string | null = null;

    // ⬇️ Comprobación temporal
    console.log("VITE_SUPABASE_URL", import.meta.env.VITE_SUPABASE_URL);
    console.log(
        "VITE_SUPABASE_ANON_KEY presente?",
        !!import.meta.env.VITE_SUPABASE_ANON_KEY,
    );
</script>

<div class="rounded-2xl bg-white/70 backdrop-blur border p-4 shadow-sm">
    <ProfilePicker current={who} onPick={(n) => (who = n)} />
    <div class="mt-3 text-sm opacity-80">
        {who ? `Estás como: ${who}` : "Elige tu perfil"}
    </div>
    <GiftList {who} />
</div>
