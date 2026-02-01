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
    const TYPE_CANCEL = "cancelled";

    const TYPE_BACKUP_DONE = "backup_done";
    const TYPE_BACKUP_SKIPPED = "backup_skipped";
    const TYPE_UNABLE_TO_BACKUP = "unable_to_backup";

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
              backup: ResultTypeBackup;
              extra: ResultTypeExtra;
          }
        | typeof TYPE_CANCEL;
    type Props = {
        setResult: (result: ResultType) => void;
    };
    const { setResult }: Props = $props();

    let backupType = $state<ResultTypeBackup>(TYPE_CANCEL);
    let confirmationCheck1 = $state(false);
    let confirmationCheck2 = $state(false);
    let confirmationCheck3 = $state(false);
    const canProceed = $derived.by(() => {
        return (
            (backupType === TYPE_BACKUP_DONE || backupType === TYPE_BACKUP_SKIPPED) &&
            confirmationCheck1 &&
            confirmationCheck2 &&
            confirmationCheck3
        );
    });
    let preventFetchingConfig = $state(false);

    function commit() {
        setResult({
            backup: backupType,
            extra: {
                preventFetchingConfig,
            },
        });
    }
</script>

<DialogHeader title={msg("setup.rebuildEverything.title")} />
<Guidance>{@html msg("setup.rebuildEverything.guidance")}</Guidance>
<InfoNote>
    {msg("setup.rebuildEverything.noteWhenToUse")}
</InfoNote>
<Guidance important title={msg("setup.rebuildEverything.guidanceConfirm")}>
    <Check
        title={msg("setup.rebuildEverything.check1Title")}
        bind:value={confirmationCheck1}
    >
        <InfoNote>{msg("setup.rebuildEverything.check1Note1")}</InfoNote>
        <InfoNote>{msg("setup.rebuildEverything.check1Note2")}</InfoNote>
    </Check>
    <Check
        title={msg("setup.rebuildEverything.check2Title")}
        bind:value={confirmationCheck2}
    >
        <InfoNote>{msg("setup.rebuildEverything.check2Note")}</InfoNote>
    </Check>
    <Check title={msg("setup.rebuildEverything.check3Title")} bind:value={confirmationCheck3} />
</Guidance>
<hr />
<Instruction>
    <Question>{msg("setup.rebuildEverything.questionBackup")}</Question>
    <InfoNote warning>
        {msg("setup.rebuildEverything.noteBackupWarning")}
    </InfoNote>
    <Options>
        <Option selectedValue={TYPE_BACKUP_DONE} title={msg("setup.rebuildEverything.optionBackupDoneTitle")} bind:value={backupType} />
        <Option
            selectedValue={TYPE_BACKUP_SKIPPED}
            title={msg("setup.rebuildEverything.optionBackupSkippedTitle")}
            bind:value={backupType}
        />
        <Option
            selectedValue={TYPE_UNABLE_TO_BACKUP}
            title={msg("setup.rebuildEverything.optionUnableToBackupTitle")}
            bind:value={backupType}
        >
            <InfoNote error visible={backupType === TYPE_UNABLE_TO_BACKUP}>
                {@html msg("setup.rebuildEverything.noteUnableToBackup")}
            </InfoNote>
        </Option>
    </Options>
</Instruction>
<Instruction>
    <ExtraItems title={msg("setup.rebuildEverything.titleAdvanced")}>
        <Check title={msg("setup.rebuildEverything.checkPreventFetchConfig")} bind:value={preventFetchingConfig} />
    </ExtraItems>
</Instruction>
<UserDecisions>
    <Decision title={msg("setup.rebuildEverything.btnOverwrite")} important disabled={!canProceed} commit={() => commit()} />
    <Decision title={msg("setup.rebuildEverything.btnCancel")} commit={() => setResult(TYPE_CANCEL)} />
</UserDecisions>
