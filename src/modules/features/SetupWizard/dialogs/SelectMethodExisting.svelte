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
    const TYPE_USE_SETUP_URI = "use-setup-uri";
    const TYPE_SCAN_QR_CODE = "scan-qr-code";
    const TYPE_CONFIGURE_MANUALLY = "configure-manually";
    const TYPE_CANCELLED = "cancelled";
    type ResultType = typeof TYPE_USE_SETUP_URI | typeof TYPE_SCAN_QR_CODE | typeof TYPE_CONFIGURE_MANUALLY | typeof TYPE_CANCELLED;
    type Props = {
        setResult: (result: ResultType) => void;
    };
    const { setResult }: Props = $props();
    let userType = $state<ResultType>(TYPE_CANCELLED);
    let proceedTitle = $derived.by(() => {
        if (userType === TYPE_USE_SETUP_URI) {
            return msg("setup.selectMethodExisting.proceedSetupURI");
        } else if (userType === TYPE_CONFIGURE_MANUALLY) {
            return msg("setup.selectMethodExisting.proceedManual");
        } else if (userType === TYPE_SCAN_QR_CODE) {
            return msg("setup.selectMethodExisting.proceedQRCode");
        } else {
            return msg("setup.selectMethodExisting.proceedSelect");
        }
    });
    const canProceed = $derived.by(() => {
        return userType === TYPE_USE_SETUP_URI || userType === TYPE_CONFIGURE_MANUALLY || userType === TYPE_SCAN_QR_CODE;
    });
</script>

<DialogHeader title={msg("setup.selectMethodExisting.title")} />
<Guidance>{msg("setup.selectMethodExisting.guidance")}</Guidance>
<Instruction>
    <Question>{msg("setup.selectMethodExisting.question")}</Question>
    <Options>
        <Option selectedValue={TYPE_USE_SETUP_URI} title={msg("setup.selectMethodExisting.optionSetupURITitle")} bind:value={userType}>
            {msg("setup.selectMethodExisting.optionSetupURIDesc")}
        </Option>
        <Option selectedValue={TYPE_SCAN_QR_CODE} title={msg("setup.selectMethodExisting.optionQRCodeTitle")} bind:value={userType}>
            {msg("setup.selectMethodExisting.optionQRCodeDesc")}
        </Option>
        <Option
            selectedValue={TYPE_CONFIGURE_MANUALLY}
            title={msg("setup.selectMethodExisting.optionManualTitle")}
            bind:value={userType}
        >
            {msg("setup.selectMethodExisting.optionManualDesc")}
        </Option>
    </Options>
</Instruction>
<UserDecisions>
    <Decision title={proceedTitle} important={canProceed} disabled={!canProceed} commit={() => setResult(userType)} />
    <Decision title={msg("setup.selectMethodExisting.cancel")} commit={() => setResult(TYPE_CANCELLED)} />
</UserDecisions>
