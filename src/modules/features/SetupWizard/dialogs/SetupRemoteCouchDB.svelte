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
        PREFERRED_SETTING_CLOUDANT,
        PREFERRED_SETTING_SELF_HOSTED,
        RemoteTypes,
        type CouchDBConnection,
        type ObsidianLiveSyncSettings,
    } from "../../../../lib/src/common/types";
    import { isCloudantURI } from "../../../../lib/src/pouchdb/utils_couchdb";

    import { getObsidianDialogContext } from "../ObsidianSvelteDialog";
    import { onMount } from "svelte";
    import type { GuestDialogProps } from "../../../../lib/src/UI/svelteDialog";
    import { copyTo, pickCouchDBSyncSettings } from "../../../../lib/src/common/utils";
    import PanelCouchDBCheck from "./PanelCouchDBCheck.svelte";

    const default_setting = pickCouchDBSyncSettings(DEFAULT_SETTINGS);

    let syncSetting = $state<CouchDBConnection>({ ...default_setting });
    type ResultType = typeof TYPE_CANCELLED | CouchDBConnection;
    const TYPE_CANCELLED = "cancelled";
    type Props = GuestDialogProps<ResultType, CouchDBConnection>;
    const { setResult, getInitialData }: Props = $props();
    onMount(() => {
        if (getInitialData) {
            const initialData = getInitialData();
            if (initialData) {
                copyTo(initialData, syncSetting);
            }
        }
    });

    let error = $state("");
    const context = getObsidianDialogContext();

    function generateSetting() {
        const connSetting: CouchDBConnection = {
            ...syncSetting,
        };
        const trialSettings: CouchDBConnection = {
            ...connSetting,
            // ...encryptionSettings,
        };
        const preferredSetting = isCloudantURI(syncSetting.couchDB_URI)
            ? PREFERRED_SETTING_CLOUDANT
            : PREFERRED_SETTING_SELF_HOSTED;
        const trialRemoteSetting: ObsidianLiveSyncSettings = {
            ...DEFAULT_SETTINGS,
            ...preferredSetting,
            remoteType: RemoteTypes.REMOTE_COUCHDB,
            ...trialSettings,
        };
        return trialRemoteSetting;
    }
    let processing = $state(false);
    async function checkConnection() {
        try {
            processing = true;
            const trialRemoteSetting = generateSetting();
            const replicator = await context.services.replicator.getNewReplicator(trialRemoteSetting);
            if (!replicator) {
                return "Failed to create replicator instance.";
            }
            try {
                const result = await replicator.tryConnectRemote(trialRemoteSetting, false);
                if (result) {
                    return "";
                } else {
                    return "Failed to connect to the server. Please check your settings.";
                }
            } catch (e) {
                return `Failed to connect to the server: ${e}`;
            }
        } finally {
            processing = false;
        }
    }

    async function checkAndCommit() {
        error = "";
        try {
            error = (await checkConnection()) || "";
            if (!error) {
                const setting = generateSetting();
                setResult(pickCouchDBSyncSettings(setting));
                return;
            }
        } catch (e) {
            error = `Error during connection test: ${e}`;
            return;
        }
    }
    function commit() {
        const setting = pickCouchDBSyncSettings(generateSetting());
        setResult(setting);
    }
    function cancel() {
        setResult(TYPE_CANCELLED);
    }

    // const isURICloudant = $derived.by(() => {
    //     return syncSetting.couchDB_URI && isCloudantURI(syncSetting.couchDB_URI);
    // });
    // const isURISelfHosted = $derived.by(() => {
    //     return syncSetting.couchDB_URI && !isCloudantURI(syncSetting.couchDB_URI);
    // });
    // const isURISecure = $derived.by(() => {
    //     return syncSetting.couchDB_URI && syncSetting.couchDB_URI.startsWith("https://");
    // });
    const isURIInsecure = $derived.by(() => {
        return !!(syncSetting.couchDB_URI && syncSetting.couchDB_URI.startsWith("http://"));
    });
    const isUseJWT = $derived.by(() => {
        return syncSetting.useJWT;
    });
    const canProceed = $derived.by(() => {
        return (
            syncSetting.couchDB_URI.trim().length > 0 &&
            syncSetting.couchDB_USER.trim().length > 0 &&
            syncSetting.couchDB_PASSWORD.trim().length > 0 &&
            syncSetting.couchDB_DBNAME.trim().length > 0 &&
            (isUseJWT ? syncSetting.jwtKey.trim().length > 0 : true)
        );
    });
    const testSettings = $derived.by(() => {
        return generateSetting();
    });
</script>

<DialogHeader title={msg("setup.setupRemoteCouchDB.title")} />
<Guidance>{msg("setup.setupRemoteCouchDB.guidance")}</Guidance>
<InputRow label={msg("setup.setupRemoteCouchDB.labelURL")}>
    <input
        type="text"
        name="couchdb-url"
        placeholder={msg("setup.setupRemoteCouchDB.placeholderURL")}
        autocorrect="off"
        autocapitalize="off"
        spellcheck="false"
        bind:value={syncSetting.couchDB_URI}
        required
        pattern="^https?://.+"
    />
</InputRow>
<InfoNote warning visible={isURIInsecure}>{msg("setup.setupRemoteCouchDB.noteInsecure")}</InfoNote>
<InputRow label={msg("setup.setupRemoteCouchDB.labelUsername")}>
    <input
        type="text"
        name="couchdb-username"
        placeholder={msg("setup.setupRemoteCouchDB.placeholderUsername")}
        autocorrect="off"
        autocapitalize="off"
        spellcheck="false"
        required
        bind:value={syncSetting.couchDB_USER}
    />
</InputRow>
<InputRow label={msg("setup.setupRemoteCouchDB.labelPassword")}>
    <Password
        name="couchdb-password"
        placeholder={msg("setup.setupRemoteCouchDB.placeholderPassword")}
        bind:value={syncSetting.couchDB_PASSWORD}
        required
    />
</InputRow>

<InputRow label={msg("setup.setupRemoteCouchDB.labelDatabaseName")}>
    <input
        type="text"
        name="couchdb-database"
        placeholder={msg("setup.setupRemoteCouchDB.placeholderDatabaseName")}
        autocorrect="off"
        autocapitalize="off"
        spellcheck="false"
        required
        pattern="^[a-z0-9][a-z0-9_]*$"
        bind:value={syncSetting.couchDB_DBNAME}
    />
</InputRow>
<InfoNote>
    {msg("setup.setupRemoteCouchDB.noteDatabaseName")}
</InfoNote>
<InputRow label={msg("setup.setupRemoteCouchDB.labelUseInternalAPI")}>
    <input type="checkbox" name="couchdb-use-internal-api" bind:checked={syncSetting.useRequestAPI} />
</InputRow>
<InfoNote>
    {msg("setup.setupRemoteCouchDB.noteUseInternalAPI")}
</InfoNote>

<ExtraItems title={msg("setup.setupRemoteCouchDB.titleAdvancedSettings")}>
    <InputRow label={msg("setup.setupRemoteCouchDB.labelCustomHeaders")}>
        <textarea
            name="couchdb-custom-headers"
            placeholder={msg("setup.setupRemoteCouchDB.placeholderCustomHeaders")}
            bind:value={syncSetting.couchDB_CustomHeaders}
            autocapitalize="off"
            spellcheck="false"
            rows="4"
        ></textarea>
    </InputRow>
</ExtraItems>
<ExtraItems title={msg("setup.setupRemoteCouchDB.titleExperimentalSettings")}>
    <InputRow label={msg("setup.setupRemoteCouchDB.labelUseJWT")}>
        <input type="checkbox" name="couchdb-use-jwt" bind:checked={syncSetting.useJWT} />
    </InputRow>
    <InputRow label={msg("setup.setupRemoteCouchDB.labelJWTAlgorithm")}>
        <select bind:value={syncSetting.jwtAlgorithm} disabled={!isUseJWT}>
            <option value="HS256">HS256</option>
            <option value="HS512">HS512</option>
            <option value="ES256">ES256</option>
            <option value="ES512">ES512</option>
        </select>
    </InputRow>
    <InputRow label={msg("setup.setupRemoteCouchDB.labelJWTExpDuration")}>
        <input
            type="text"
            name="couchdb-jwt-exp-duration"
            placeholder={msg("setup.setupRemoteCouchDB.placeholderJWTExpDuration")}
            bind:value={() => `${syncSetting.jwtExpDuration}`, (v) => (syncSetting.jwtExpDuration = parseInt(v) || 0)}
            disabled={!isUseJWT}
        />
    </InputRow>
    <InputRow label={msg("setup.setupRemoteCouchDB.labelJWTKey")}>
        <textarea
            name="couchdb-jwt-key"
            rows="5"
            autocapitalize="off"
            spellcheck="false"
            placeholder={msg("setup.setupRemoteCouchDB.placeholderJWTKey")}
            bind:value={syncSetting.jwtKey}
            disabled={!isUseJWT}
        ></textarea>
    </InputRow>
    <InfoNote>
        {msg("setup.setupRemoteCouchDB.noteJWTKey")}
    </InfoNote>
    <InputRow label={msg("setup.setupRemoteCouchDB.labelJWTKeyID")}>
        <input
            type="text"
            name="couchdb-jwt-kid"
            placeholder={msg("setup.setupRemoteCouchDB.placeholderJWTKeyID")}
            bind:value={syncSetting.jwtKid}
            disabled={!isUseJWT}
        />
    </InputRow>
    <InputRow label={msg("setup.setupRemoteCouchDB.labelJWTSubject")}>
        <input
            type="text"
            name="couchdb-jwt-sub"
            placeholder={msg("setup.setupRemoteCouchDB.placeholderJWTSubject")}
            bind:value={syncSetting.jwtSub}
            disabled={!isUseJWT}
        />
    </InputRow>
    <InfoNote warning>
        {msg("setup.setupRemoteCouchDB.noteJWTWarning")}
    </InfoNote>
</ExtraItems>

<PanelCouchDBCheck trialRemoteSetting={testSettings}></PanelCouchDBCheck>
<hr />

<InfoNote error visible={error !== ""}>
    {error}
</InfoNote>

{#if processing}
    {msg("setup.setupRemoteCouchDB.btnChecking")}
{:else}
    <UserDecisions>
        <Decision title={msg("setup.setupRemoteCouchDB.btnTestAndContinue")} important disabled={!canProceed} commit={() => checkAndCommit()} />
        <Decision title={msg("setup.setupRemoteCouchDB.btnContinueAnyway")} commit={() => commit()} />
        <Decision title={msg("setup.setupRemoteCouchDB.btnCancel")} commit={() => cancel()} />
    </UserDecisions>
{/if}
