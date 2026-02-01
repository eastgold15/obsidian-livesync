<script lang="ts">
    import DialogHeader from "@/lib/src/UI/components/DialogHeader.svelte";
    import Guidance from "@/lib/src/UI/components/Guidance.svelte";
    import Decision from "@/lib/src/UI/components/Decision.svelte";
    import Instruction from "@/lib/src/UI/components/Instruction.svelte";
    import UserDecisions from "@/lib/src/UI/components/UserDecisions.svelte";
    import { $msg as msg } from "@/lib/src/common/i18n";
    const TYPE_CLOSE = "close";
    type ResultType = typeof TYPE_CLOSE;
    type Props = {
        setResult: (result: ResultType) => void;
    };
    const { setResult }: Props = $props();
</script>

<DialogHeader title={msg("setup.scanQRCode.title")} />
<Guidance>{msg("setup.scanQRCode.guidance")}</Guidance>
<Instruction>
    <!-- <Question>How would you like to configure the connection to your server?</Question> -->
    <ol>
        <li>{msg("setup.scanQRCode.step1")}</li>
        <li>{msg("setup.scanQRCode.step2")}</li>
        <li>{msg("setup.scanQRCode.step3")}</li>
        <li>{msg("setup.scanQRCode.step4")}</li>
    </ol>
</Instruction>
<UserDecisions>
    <Decision title={msg("setup.scanQRCode.btnClose")} important={true} commit={() => setResult(TYPE_CLOSE)} />
</UserDecisions>
