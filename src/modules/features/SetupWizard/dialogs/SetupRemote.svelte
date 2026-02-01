<script lang="ts">
    import DialogHeader from "@/lib/src/UI/components/DialogHeader.svelte";
    import Decision from "@/lib/src/UI/components/Decision.svelte";
    import Question from "@/lib/src/UI/components/Question.svelte";
    import Option from "@/lib/src/UI/components/Option.svelte";
    import Options from "@/lib/src/UI/components/Options.svelte";
    import Instruction from "@/lib/src/UI/components/Instruction.svelte";
    import UserDecisions from "@/lib/src/UI/components/UserDecisions.svelte";
    import { $msg as msg } from "@/lib/src/common/i18n";
    const TYPE_COUCHDB = "couchdb";
    const TYPE_BUCKET = "bucket";
    const TYPE_P2P = "p2p";
    const TYPE_CANCELLED = "cancelled";
    type ResultType = typeof TYPE_COUCHDB | typeof TYPE_BUCKET | typeof TYPE_P2P | typeof TYPE_CANCELLED;
    type Props = {
        setResult: (result: ResultType) => void;
    };
    const { setResult }: Props = $props();
    let userType = $state<ResultType>(TYPE_CANCELLED);
    let proceedTitle = $derived.by(() => {
        if (userType === TYPE_COUCHDB) {
            return msg("setup.setupRemote.proceedCouchDB");
        } else if (userType === TYPE_BUCKET) {
            return msg("setup.setupRemote.proceedBucket");
        } else if (userType === TYPE_P2P) {
            return msg("setup.setupRemote.proceedP2P");
        } else {
            return msg("setup.setupRemote.proceedSelect");
        }
    });
    const canProceed = $derived.by(() => {
        return userType === TYPE_COUCHDB || userType === TYPE_BUCKET || userType === TYPE_P2P;
    });
</script>

<DialogHeader title={msg("setup.setupRemote.title")} />
<Instruction>
    <Question>{msg("setup.setupRemote.question")}</Question>
    <Options>
        <Option selectedValue={TYPE_COUCHDB} title={msg("setup.setupRemote.optionCouchDBTitle")} bind:value={userType}>
            {msg("setup.setupRemote.optionCouchDBDesc")}
        </Option>
        <Option selectedValue={TYPE_BUCKET} title={msg("setup.setupRemote.optionBucketTitle")} bind:value={userType}>
            {msg("setup.setupRemote.optionBucketDesc")}
        </Option>
        <Option selectedValue={TYPE_P2P} title={msg("setup.setupRemote.optionP2PTitle")} bind:value={userType}>
            {msg("setup.setupRemote.optionP2PDesc")}
        </Option>
    </Options>
</Instruction>
<UserDecisions>
    <Decision title={proceedTitle} important={canProceed} disabled={!canProceed} commit={() => setResult(userType)} />
    <Decision title={msg("setup.setupRemote.takeBack")} commit={() => setResult(TYPE_CANCELLED)} />
</UserDecisions>
