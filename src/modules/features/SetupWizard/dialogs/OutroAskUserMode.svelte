<script lang="ts">
    import DialogHeader from "@/lib/src/UI/components/DialogHeader.svelte";
    import Guidance from "@/lib/src/UI/components/Guidance.svelte";
    import Decision from "@/lib/src/UI/components/Decision.svelte";
    import Question from "@/lib/src/UI/components/Question.svelte";
    import Option from "@/lib/src/UI/components/Option.svelte";
    import Instruction from "@/lib/src/UI/components/Instruction.svelte";
    import UserDecisions from "@/lib/src/UI/components/UserDecisions.svelte";
    import InfoNote from "@/lib/src/UI/components/InfoNote.svelte";
    import { $msg as msg } from "@/lib/src/common/i18n";
    const TYPE_EXISTING = "existing-user";
    const TYPE_NEW = "new-user";
    const TYPE_COMPATIBLE_EXISTING = "compatible-existing-user";
    const TYPE_CANCELLED = "cancelled";
    type ResultType = typeof TYPE_EXISTING | typeof TYPE_NEW | typeof TYPE_COMPATIBLE_EXISTING | typeof TYPE_CANCELLED;
    type Props = {
        setResult: (result: ResultType) => void;
    };
    const { setResult }: Props = $props();
    let userType = $state<ResultType>(TYPE_CANCELLED);
    const canProceed = $derived.by(() => {
        return userType === TYPE_EXISTING || userType === TYPE_NEW || userType === TYPE_COMPATIBLE_EXISTING;
    });
    const proceedMessage = $derived.by(() => {
        if (userType === TYPE_NEW) {
            return msg("setup.outroAskUserMode.proceedMessage");
        } else if (userType === TYPE_EXISTING) {
            return msg("setup.outroAskUserMode.proceedMessage");
        } else if (userType === TYPE_COMPATIBLE_EXISTING) {
            return msg("setup.outroAskUserMode.proceedMessageApply");
        } else {
            return msg("setup.outroAskUserMode.proceedSelect");
        }
    });
</script>

<DialogHeader title={msg("setup.outroAskUserMode.title")} />
<Guidance>
    {@html msg("setup.outroAskUserMode.guidance")}
</Guidance>
<Instruction>
    <Question>{msg("setup.outroAskUserMode.question")}</Question>
    <Option title={msg("setup.outroAskUserMode.optionNewTitle")} bind:value={userType} selectedValue={TYPE_NEW}>
        <InfoNote>
            {msg("setup.outroAskUserMode.optionNewDesc")}
        </InfoNote>
    </Option>
    <Option
        title={msg("setup.outroAskUserMode.optionExistingTitle")}
        bind:value={userType}
        selectedValue={TYPE_EXISTING}
    >
        <InfoNote>
            {msg("setup.outroAskUserMode.optionExistingDesc")}
        </InfoNote>
    </Option>
    <Option
        title={msg("setup.outroAskUserMode.optionCompatibleTitle")}
        bind:value={userType}
        selectedValue={TYPE_COMPATIBLE_EXISTING}
    >
        <InfoNote warning>
            {msg("setup.outroAskUserMode.optionCompatibleDesc")}
        </InfoNote>
    </Option>
</Instruction>
<UserDecisions>
    <Decision title={proceedMessage} important={true} disabled={!canProceed} commit={() => setResult(userType)} />
    <Decision title={msg("setup.outroAskUserMode.btnTakeBack")} commit={() => setResult(TYPE_CANCELLED)} />
</UserDecisions>
