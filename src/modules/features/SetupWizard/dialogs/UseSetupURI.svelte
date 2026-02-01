<script lang="ts">
    import { configURIBase } from "../../../../common/types";
    import type { ObsidianLiveSyncSettings } from "../../../../lib/src/common/types";
    import DialogHeader from "@/lib/src/UI/components/DialogHeader.svelte";
    import Guidance from "@/lib/src/UI/components/Guidance.svelte";
    import Decision from "@/lib/src/UI/components/Decision.svelte";
    import UserDecisions from "@/lib/src/UI/components/UserDecisions.svelte";
    import InfoNote from "@/lib/src/UI/components/InfoNote.svelte";
    import InputRow from "@/lib/src/UI/components/InputRow.svelte";
    import Password from "@/lib/src/UI/components/Password.svelte";
    import { $msg as msg } from "@/lib/src/common/i18n";

    import { onMount } from "svelte";
    import { decryptString } from "../../../../lib/src/encryption/stringEncryption.ts";
    import type { GuestDialogProps } from "../../../../lib/src/UI/svelteDialog.ts";
    const TYPE_CANCELLED = "cancelled";
    type ResultType = typeof TYPE_CANCELLED | ObsidianLiveSyncSettings;
    type Props = GuestDialogProps<ResultType, string>;
    const { setResult, getInitialData }: Props = $props();

    let setupURI = $state("");
    let passphrase = $state("");
    let error = $state("");
    onMount(() => {
        if (getInitialData) {
            const initialURI = getInitialData();
            if (initialURI) {
                setupURI = initialURI;
            }
        }
    });

    const seemsValid = $derived.by(() => setupURI.startsWith(configURIBase));
    async function processSetupURI() {
        error = "";
        if (!seemsValid) return;
        if (!passphrase) {
            error = msg("setup.useSetupURI.errorPassphraseRequired");
            return;
        }
        try {
            const settingPieces = setupURI.substring(configURIBase.length);
            const encodedConfig = decodeURIComponent(settingPieces);
            const newConf = (await JSON.parse(
                await decryptString(encodedConfig, passphrase)
            )) as ObsidianLiveSyncSettings;
            setResult(newConf);
            // Logger("Settings imported successfully", LOG_LEVEL_NOTICE);
            return;
        } catch (e) {
            error = msg("setup.useSetupURI.errorParseFailed");
            return;
        }
    }
    async function canProceed() {
        return (await processSetupURI()) ?? false;
    }
</script>

<DialogHeader title={msg("setup.useSetupURI.title")} />
<Guidance
    >{msg("setup.useSetupURI.guidance")}</Guidance
>

<InputRow label={msg("setup.useSetupURI.labelSetupURI")}>
    <input
        type="text"
        placeholder={msg("setup.useSetupURI.placeholderSetupURI")}
        bind:value={setupURI}
        autocorrect="off"
        autocapitalize="off"
        spellcheck="false"
        required
    />
</InputRow>
<InfoNote visible={seemsValid}>{msg("setup.useSetupURI.noteValid")}</InfoNote>
<InfoNote warning visible={!seemsValid && setupURI.trim() != ""}>
    {msg("setup.useSetupURI.noteInvalid")}
</InfoNote>
<InputRow label={msg("setup.useSetupURI.labelPassphrase")}>
    <Password placeholder={msg("setup.useSetupURI.placeholderPassphrase")} bind:value={passphrase} required />
</InputRow>
<InfoNote error visible={error.trim() != ""}>
    {error}
</InfoNote>

<UserDecisions>
    <Decision
        title={msg("setup.useSetupURI.btnTestAndContinue")}
        important={true}
        disabled={!canProceed}
        commit={() => processSetupURI()}
    />
    <Decision title={msg("setup.useSetupURI.btnCancel")} commit={() => setResult(TYPE_CANCELLED)} />
</UserDecisions>
