<script lang="ts">
    import DialogHeader from "@/lib/src/UI/components/DialogHeader.svelte";
    import Guidance from "@/lib/src/UI/components/Guidance.svelte";
    import Decision from "@/lib/src/UI/components/Decision.svelte";
    import Question from "@/lib/src/UI/components/Question.svelte";
    import Instruction from "@/lib/src/UI/components/Instruction.svelte";
    import UserDecisions from "@/lib/src/UI/components/UserDecisions.svelte";
    import { $msg as msg } from "@/lib/src/common/i18n";
    const TYPE_APPLY = "apply";
    const TYPE_CANCELLED = "cancelled";
    type ResultType = typeof TYPE_APPLY | typeof TYPE_CANCELLED;
    type Props = {
        setResult: (result: ResultType) => void;
    };
    const { setResult }: Props = $props();
</script>

<DialogHeader title={msg("setup.outroNewUser.title")} />
<Guidance>
    <p>
        {@html msg("setup.outroNewUser.guidanceMain")}
    </p>
    <p>
        {@html msg("setup.outroNewUser.guidanceImportant")}
    </p>
</Guidance>
<Instruction>
    <Question>{msg("setup.outroNewUser.question")}</Question>
</Instruction>
<UserDecisions>
    <Decision title={msg("setup.outroNewUser.btnRestart")} important={true} commit={() => setResult(TYPE_APPLY)} />
    <Decision title={msg("setup.outroNewUser.btnTakeBack")} commit={() => setResult(TYPE_CANCELLED)} />
</UserDecisions>
