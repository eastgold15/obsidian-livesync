<script lang="ts">
    import DialogHeader from "@/lib/src/UI/components/DialogHeader.svelte";
    import Guidance from "@/lib/src/UI/components/Guidance.svelte";
    import Decision from "@/lib/src/UI/components/Decision.svelte";
    import UserDecisions from "@/lib/src/UI/components/UserDecisions.svelte";
    import InfoNote from "@/lib/src/UI/components/InfoNote.svelte";
    import ExtraItems from "@/lib/src/UI/components/ExtraItems.svelte";
    import InputRow from "@/lib/src/UI/components/InputRow.svelte";
    import Password from "@/lib/src/UI/components/Password.svelte";
    import { $msg as msg } from "@/lib/src/common/i18n";
    import {
        DEFAULT_SETTINGS,
        E2EEAlgorithmNames,
        E2EEAlgorithms,
        type EncryptionSettings,
    } from "../../../../lib/src/common/types";
    import { onMount } from "svelte";
    import type { GuestDialogProps } from "../../../../lib/src/UI/svelteDialog";
    import { copyTo, pickEncryptionSettings } from "../../../../lib/src/common/utils";
    const TYPE_CANCELLED = "cancelled";
    type ResultType = typeof TYPE_CANCELLED | EncryptionSettings;
    type Props = GuestDialogProps<ResultType, EncryptionSettings>;
    const { setResult, getInitialData }: Props = $props();
    let default_encryption: EncryptionSettings = {
        encrypt: true,
        passphrase: "",
        E2EEAlgorithm: DEFAULT_SETTINGS.E2EEAlgorithm,
        usePathObfuscation: true,
    } as EncryptionSettings;

    let encryptionSettings = $state<EncryptionSettings>({ ...default_encryption });

    onMount(() => {
        if (getInitialData) {
            const initialData = getInitialData();
            if (initialData) {
                copyTo(initialData, encryptionSettings);
            }
        }
    });
    let e2eeValid = $derived.by(() => {
        if (!encryptionSettings.encrypt) return true;
        return encryptionSettings.passphrase.trim().length >= 1;
    });

    function commit() {
        setResult(pickEncryptionSettings(encryptionSettings));
    }
</script>

<DialogHeader title={msg("setup.setupRemoteE2EE.title")} />
<Guidance>{msg("setup.setupRemoteE2EE.guidance")}</Guidance>
<InputRow label={msg("setup.setupRemoteE2EE.labelE2EE")}>
    <input type="checkbox" bind:checked={encryptionSettings.encrypt} />
    <Password
        name="e2ee-passphrase"
        placeholder={msg("setup.setupRemoteE2EE.placeholderPassphrase")}
        bind:value={encryptionSettings.passphrase}
        disabled={!encryptionSettings.encrypt}
        required={encryptionSettings.encrypt}
    />
</InputRow>
<InfoNote title="Strongly Recommended">
    {@html msg("setup.setupRemoteE2EE.noteStronglyRecommended")}
</InfoNote>
<InfoNote warning>
    {msg("setup.setupRemoteE2EE.noteMustBeSame")}
</InfoNote>
<InputRow label={msg("setup.setupRemoteE2EE.labelObfuscateProperties")}>
    <input
        type="checkbox"
        bind:checked={encryptionSettings.usePathObfuscation}
        disabled={!encryptionSettings.encrypt}
    />
</InputRow>

<InfoNote>
    {msg("setup.setupRemoteE2EE.noteObfuscateProperties")}
</InfoNote>

<ExtraItems title={msg("setup.setupRemoteE2EE.titleAdvanced")}>
    <InputRow label={msg("setup.setupRemoteE2EE.labelEncryptionAlgorithm")}>
        <select bind:value={encryptionSettings.E2EEAlgorithm} disabled={!encryptionSettings.encrypt}>
            {#each Object.values(E2EEAlgorithms) as alg}
                <option value={alg}>{E2EEAlgorithmNames[alg] ?? alg}</option>
            {/each}
        </select>
    </InputRow>
    <InfoNote>
        {@html msg("setup.setupRemoteE2EE.noteEncryptionAlgorithm", { defaultAlgorithm: E2EEAlgorithmNames[DEFAULT_SETTINGS.E2EEAlgorithm] })}
    </InfoNote>
    <InfoNote warning>
        {msg("setup.setupRemoteE2EE.noteAlgorithmWarning")}
    </InfoNote>
</ExtraItems>

<InfoNote warning>
    {@html msg("setup.setupRemoteE2EE.notePassphraseValidation")}
</InfoNote>

<UserDecisions>
    <Decision title={msg("setup.setupRemoteE2EE.btnProceed")} important disabled={!e2eeValid} commit={() => commit()} />
    <Decision title={msg("setup.setupRemoteE2EE.btnCancel")} commit={() => setResult(TYPE_CANCELLED)} />
</UserDecisions>
