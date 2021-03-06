<script>
  import { AppContent, Scrim } from "@smui/drawer";
  import Snackbar, { Actions, Label as SnackbarLabel } from "@smui/snackbar";
  import IconButton from "@smui/icon-button";
  import Button from "@smui/button";
  import { mdiClose } from "@mdi/js";
  import { onDestroy } from "svelte";
  import TopBar from "./TopBar.svelte";
  import Drawer from "./Drawer.svelte";
  import Filters from "./Filters.svelte";
  import Headers from "./Headers.svelte";
  import {
    selectedProfile,
    isPaused,
    addHeader,
    removeHeader,
    addUrlReplacement,
    removeUrlReplacement,
    commitChange,
    undo,
    save,
    init
  } from "../js/datasource";
  import MdiIcon from "./MdiIcon.svelte";
  import { toastMessage, undoable } from "../js/toast";
  import {
    KNOWN_REQUEST_HEADERS,
    KNOWN_RESPONSE_HEADERS
  } from "../js/constants";
  import lodashClone from "lodash/clone";

  let snackbar;
  let snackbarMessage;

  window.addEventListener("unload", save);

  const unsubscribeToastMessage = toastMessage.subscribe(message => {
    if (snackbar) {
      snackbarMessage = message;
      if (message.length > 0) {
        snackbar.open();
      } else {
        snackbar.close();
      }
    }
  });

  onDestroy(unsubscribeToastMessage);
</script>

<style lang="scss">
  :global(html),
  :global(body) {
    height: 460px;
    /** Fix FF popup disappearance on long window. */
    width: 620px !important;
    position: relative !important;
    margin: 0;
  }

  :global(button::-moz-focus-inner) {
    border: 0;
  }

  :global(.extra-gap) {
    margin: 2px;
  }

  :global(.small-icon-button) {
    width: 32px;
    height: 32px;
    padding: 5px;
  }

  :global(.data-table) {
    border-color: #bbb;
    width: calc(100% - 4px);
  }

  :global(.data-table-title) {
    height: 20px;
    margin: 0;
    padding: 0;
    width: 120px;
  }

  :global(.data-table-checkbox-cell) {
    width: 32px;
  }

  :global(.data-table-title-cell) {
    width: 140px;
  }

  :global(.data-table-row) {
    height: 20px;
    margin: 0;
    padding: 0;
    border-top-color: #eee;
  }

  :global(.data-table-cell) {
    padding-left: 2px;
    padding-right: 2px;
  }

  :global(.autocomplete-input) {
    border: none;
    border-bottom: 1px solid #ddd;
    height: 32px;
    width: 100%;
    background: none;
    margin: 0;
    outline: none;
    padding: 0;
  }

  :global(.dialog-close-button) {
    float: right;
    top: 4px;
  }

  :global(.hidden) {
    display: none !important;
  }

  :global(.app-content) {
    margin-left: 0 !important;
    position: absolute;
    left: 36px;
  }

  :global(.disabled) {
    opacity: 0.5;
    pointer-events: none;
  }

  :global(.large-textarea) {
    width: 100%;
    height: 50px;
  }

  :global(.extra-large-textarea) {
    width: 400px;
    height: 250px;
    overflow-x: hidden;
    overflow-y: auto;
  }

  :global(.ml-small) {
    margin-left: 5px;
  }

  .top-app-bar-container {
    height: 48px;
  }
</style>

{#await init() then initResult}
  <Drawer />

  <AppContent class="app-content">
    <div class="top-app-bar-container">
      <TopBar />
    </div>
    <div class={$isPaused ? 'disabled' : ''}>
      <Headers
        headers={$selectedProfile.headers}
        class="extra-gap"
        title="Request headers"
        autocompleteNames={KNOWN_REQUEST_HEADERS}
        on:add={() => {
          addHeader($selectedProfile.headers);
          commitChange({ headers: $selectedProfile.headers });
        }}
        on:remove={event => {
          removeHeader($selectedProfile.headers, event.detail);
          commitChange({ headers: $selectedProfile.headers });
        }}
        on:refresh={event => {
          commitChange({ headers: event.detail });
        }} />
      <Headers
        headers={$selectedProfile.respHeaders}
        class="extra-gap"
        title="Response headers"
        autocompleteNames={KNOWN_RESPONSE_HEADERS}
        profile={selectedProfile}
        on:add={() => {
          addHeader($selectedProfile.respHeaders);
          commitChange({ respHeaders: $selectedProfile.respHeaders });
        }}
        on:remove={event => {
          removeHeader($selectedProfile.respHeaders, event.detail);
          commitChange({ respHeaders: $selectedProfile.respHeaders });
        }}
        on:refresh={event => {
          commitChange({ respHeaders: event.detail });
        }} />
      <Headers
        headers={$selectedProfile.urlReplacements}
        class="extra-gap"
        title="Redirect URLs"
        autocompleteNames={[]}
        nameLabel="Original URL"
        valueLabel="Redirect URL"
        profile={$selectedProfile}
        on:add={async () => {
          await addUrlReplacement($selectedProfile.urlReplacements);
          commitChange({ urlReplacements: $selectedProfile.urlReplacements });
        }}
        on:remove={event => {
          removeUrlReplacement($selectedProfile.urlReplacements, event.detail);
          commitChange({ urlReplacements: $selectedProfile.urlReplacements });
        }}
        on:refresh={event => {
          commitChange({ urlReplacements: event.detail });
        }} />
      <Filters class="extra-gap" />
    </div>
  </AppContent>

  <Snackbar timeoutMs={4000} bind:this={snackbar} labelText={snackbarMessage}>
    <SnackbarLabel />
    <Actions>
      {#if $undoable}
        <Button on:click={() => undo()}>Undo</Button>
      {/if}
      <IconButton dense on:click={() => snackbar.close()} title="Dismiss">
        <MdiIcon size="24" icon={mdiClose} color="white" />
      </IconButton>
    </Actions>
  </Snackbar>
{/await}
