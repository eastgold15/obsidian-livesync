<script lang="ts">
    import DialogHeader from "@/lib/src/UI/components/DialogHeader.svelte";
    import Guidance from "@/lib/src/UI/components/Guidance.svelte";
    import Decision from "@/lib/src/UI/components/Decision.svelte";
    import Question from "@/lib/src/UI/components/Question.svelte";
    import Option from "@/lib/src/UI/components/Option.svelte";
    import Options from "@/lib/src/UI/components/Options.svelte";
    import Instruction from "@/lib/src/UI/components/Instruction.svelte";
    import UserDecisions from "@/lib/src/UI/components/UserDecisions.svelte";
    import InfoNote from "@/lib/src/UI/components/InfoNote.svelte";
    import ExtraItems from "@/lib/src/UI/components/ExtraItems.svelte";
    import Check from "@/lib/src/UI/components/Check.svelte";
    import { $msg as msg } from "@/lib/src/common/i18n";
    const TYPE_IDENTICAL = "identical";
    const TYPE_INDEPENDENT = "independent";
    const TYPE_UNBALANCED = "unbalanced";
    const TYPE_CANCEL = "cancelled";

    const TYPE_BACKUP_DONE = "backup_done";
    const TYPE_BACKUP_SKIPPED = "backup_skipped";
    const TYPE_UNABLE_TO_BACKUP = "unable_to_backup";

    type ResultTypeVault =
        | typeof TYPE_IDENTICAL
        | typeof TYPE_INDEPENDENT
        | typeof TYPE_UNBALANCED
        | typeof TYPE_CANCEL;
    type ResultTypeBackup =
        | typeof TYPE_BACKUP_DONE
        | typeof TYPE_BACKUP_SKIPPED
        | typeof TYPE_UNABLE_TO_BACKUP
        | typeof TYPE_CANCEL;

    type ResultTypeExtra = {
        preventFetchingConfig: boolean;
    };
    type ResultType =
        | {
              vault: ResultTypeVault;
              backup: ResultTypeBackup;
              extra: ResultTypeExtra;
          }
        | typeof TYPE_CANCEL;
    type Props = {
        setResult: (result: ResultType) => void;
    };
    const { setResult }: Props = $props();
    let vaultType = $state<ResultTypeVault>(TYPE_CANCEL);
    let backupType = $state<ResultTypeBackup>(TYPE_CANCEL);
    const canProceed = $derived.by(() => {
        return (
            (vaultType === TYPE_IDENTICAL || vaultType === TYPE_INDEPENDENT || vaultType === TYPE_UNBALANCED) &&
            (backupType === TYPE_BACKUP_DONE || backupType === TYPE_BACKUP_SKIPPED)
        );
    });
    let preventFetchingConfig = $state(false);

    function commit() {
        setResult({
            vault: vaultType,
            backup: backupType,
            extra: {
                preventFetchingConfig,
            },
        });
    }
</script>

<DialogHeader title={msg("setup.fetchEverything.title")} />
<Guidance>{msg("setup.fetchEverything.guidance")}</Guidance>
<Guidance important title={msg("setup.fetchEverything.guidanceImportantTitle")}>
    {@html msg("setup.fetchEverything.guidanceImportant")}
</Guidance>
<hr />
<Instruction>
    <Question>{@html msg("setup.fetchEverything.questionMinimise")}</Question>
    <Options>
        <Option
            selectedValue={TYPE_IDENTICAL}
            title={msg("setup.fetchEverything.optionIdenticalTitle")}
            bind:value={vaultType}
        >
            {msg("setup.fetchEverything.optionIdenticalDesc")}
        </Option>
        <Option
            selectedValue={TYPE_INDEPENDENT}
            title={msg("setup.fetchEverything.optionIndependentTitle")}
            bind:value={vaultType}
        >
            {msg("setup.fetchEverything.optionIndependentDesc")}
        </Option>
        <Option
            selectedValue={TYPE_UNBALANCED}
            title={msg("setup.fetchEverything.optionUnbalancedTitle")}
            bind:value={vaultType}
        >
      
            {msg("setup.fetchEverything.optionUnbalancedDesc")}
            <InfoNote info>
                {msg("setup.fetchEverything.noteUnbalancedInfo")}
            </InfoNote>
        </Option>
    </Options>
</Instruction>
<hr />
<Instruction>
    <Question>{msg("setup.fetchEverything.questionBackup")}</Question>
    <InfoNote>
        {msg("setup.fetchEverything.noteBackup")}
    </InfoNote>
    <Options>
        <Option selectedValue={TYPE_BACKUP_DONE} title={msg("setup.fetchEverything.optionBackupDoneTitle")} bind:value={backupType} />
        <Option
            selectedValue={TYPE_BACKUP_SKIPPED}
            title={msg("setup.fetchEverything.optionBackupSkippedTitle")}
            bind:value={backupType}
        />
        <Option
            selectedValue={TYPE_UNABLE_TO_BACKUP}
            title={msg("setup.fetchEverything.optionUnableToBackupTitle")}
            bind:value={backupType}
        >
            <InfoNote error visible={backupType === TYPE_UNABLE_TO_BACKUP}>
                {msg("setup.fetchEverything.noteUnableToBackup")}
            </InfoNote>
        </Option>
    </Options>
</Instruction>
<Instruction>
    <ExtraItems title={msg("setup.fetchEverything.titleAdvanced")}>
        <Check title={msg("setup.fetchEverything.checkPreventFetchConfig")} bind:value={preventFetchingConfig} />
    </ExtraItems>
</Instruction>
<UserDecisions>
    <Decision title={msg("setup.fetchEverything.btnResetAndResume")} important disabled={!canProceed} commit={() => commit()} />
    <Decision title={msg("setup.fetchEverything.btnCancel")} commit={() => setResult(TYPE_CANCEL)} />
</UserDecisions>
