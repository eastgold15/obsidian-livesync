<script lang="ts">
    import DialogHeader from "@/lib/src/UI/components/DialogHeader.svelte";
    import Guidance from "@/lib/src/UI/components/Guidance.svelte";
    import Decision from "@/lib/src/UI/components/Decision.svelte";
    import Question from "@/lib/src/UI/components/Question.svelte";
    import Option from "@/lib/src/UI/components/Option.svelte";
    import Options from "@/lib/src/UI/components/Options.svelte";
    import Instruction from "@/lib/src/UI/components/Instruction.svelte";
    import UserDecisions from "@/lib/src/UI/components/UserDecisions.svelte";
    import { $msg as msg } from "@/lib/src/common/i18n";
    const TYPE_NEW_USER = "new-user";
    const TYPE_EXISTING_USER = "existing-user";
    const TYPE_CANCELLED = "cancelled";
    type ResultType = typeof TYPE_NEW_USER | typeof TYPE_EXISTING_USER | typeof TYPE_CANCELLED;
    type Props = {
        setResult: (result: ResultType) => void;
    };
    const { setResult }: Props = $props();
    let userType = $state<ResultType>(TYPE_CANCELLED);
    let proceedTitle = $derived.by(() => {
        if (userType === TYPE_NEW_USER) {
            return msg("setup.intro.proceedNewUser");
        } else if (userType === TYPE_EXISTING_USER) {
            return msg("setup.intro.proceedExistingUser");
        } else {
            return msg("setup.intro.proceedSelect");
        }
    });
    const canProceed = $derived.by(() => {
        return userType === TYPE_NEW_USER || userType === TYPE_EXISTING_USER;
    });
</script>

<DialogHeader title={msg("setup.intro.title")} />
<Guidance>{msg("setup.intro.guidance")}</Guidance>
<Instruction>
    <Question>{msg("setup.intro.question")}</Question>
    <Options>
        <Option selectedValue={TYPE_NEW_USER} title={msg("setup.intro.optionNewUserTitle")} bind:value={userType}>
            {msg("setup.intro.optionNewUserDesc")}
        </Option>
        <Option
            selectedValue={TYPE_EXISTING_USER}
            title={msg("setup.intro.optionExistingUserTitle")}
            bind:value={userType}
        >
            {msg("setup.intro.optionExistingUserDesc")}
        </Option>
    </Options>
</Instruction>
<UserDecisions>
    <Decision title={proceedTitle} important={canProceed} disabled={!canProceed} commit={() => setResult(userType)} />
    <Decision title={msg("setup.intro.takeBack")} commit={() => setResult(TYPE_CANCELLED)} />
</UserDecisions>
