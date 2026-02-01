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
    const TYPE_CONFIGURE_MANUALLY = "configure-manually";
    const TYPE_CANCELLED = "cancelled";
    type ResultType = typeof TYPE_USE_SETUP_URI | typeof TYPE_CONFIGURE_MANUALLY | typeof TYPE_CANCELLED;
    type Props = {
        setResult: (result: ResultType) => void;
    };
    const { setResult }: Props = $props();
    let userType = $state<ResultType>(TYPE_CANCELLED);
    let proceedTitle = $derived.by(() => {
        if (userType === TYPE_USE_SETUP_URI) {
            return msg("setup.selectMethodNewUser.proceedSetupURI");
        } else if (userType === TYPE_CONFIGURE_MANUALLY) {
            return msg("setup.selectMethodNewUser.proceedManual");
        } else {
            return msg("setup.selectMethodNewUser.proceedSelect");
        }
    });
    const canProceed = $derived.by(() => {
        return userType === TYPE_USE_SETUP_URI || userType === TYPE_CONFIGURE_MANUALLY;
    });
</script>

<DialogHeader title={msg("setup.selectMethodNewUser.title")} />
<Guidance>{msg("setup.selectMethodNewUser.guidance")}</Guidance>
<Instruction>
    <Question>{msg("setup.selectMethodNewUser.question")}</Question>
    <Options>
        <Option selectedValue={TYPE_USE_SETUP_URI} title={msg("setup.selectMethodNewUser.optionSetupURITitle")} bind:value={userType}>
            {msg("setup.selectMethodNewUser.optionSetupURIDesc")}
        </Option>
        <Option
            selectedValue={TYPE_CONFIGURE_MANUALLY}
            title={msg("setup.selectMethodNewUser.optionManualTitle")}
            bind:value={userType}
        >
            {msg("setup.selectMethodNewUser.optionManualDesc")}
        </Option>
    </Options>
</Instruction>
<UserDecisions>
    <Decision title={proceedTitle} important={canProceed} disabled={!canProceed} commit={() => setResult(userType)} />
    <Decision title={msg("setup.selectMethodNewUser.cancel")} commit={() => setResult(TYPE_CANCELLED)} />
</UserDecisions>
