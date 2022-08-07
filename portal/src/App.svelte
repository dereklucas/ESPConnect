<script>
  import { onMount } from "svelte";
  import Connect from "./components/Connect.svelte";
  import SelectScan from "./components/SelectScan.svelte";
  import Status from "./components/Status.svelte";

  let data = {
    loading: true,
    connectStatus: {
      sent: false,
      success: true,
    },
    selection: {
      direct_connect: false,
      selected: false,
      ssid: "",
    },
    access_points: [],
    id: "",
  };

  function setConnectSuccess() {
    data.connectStatus.sent = true;
    data.connectStatus.success = true;
  }

  function setConnectError() {
    data.connectStatus.sent = true;
    data.connectStatus.success = false;
  }

  function clearConnectionStatus() {
    clearSelection();

    data.connectStatus.sent = false;
    data.connectStatus.success = true;
  }

  function clearSelection() {
    data.selection.selected = false;
    data.selection.direct_connect = false;
    data.selection.ssid = "";
  }

  function selectAccessPoint(event) {
    data.selection.ssid = event.detail.ssid;
    if (event.detail.open) {
      data.selection.direct_connect = true;
    }
    data.selection.selected = true;
  }

  async function refresh() {
    data.loading = true;
    await updateAccessPoints();
  }

  async function updateAccessPoints() {
    const res = await fetch(`/espconnect/scan`);
    // const res = await fetch(`/points.json`); // dev data
    if (res.status === 200) {
      data.access_points = await res.json();
      data.loading = false;
      data.id = res.headers.get("x-id");
    } else if (res.status === 202) {
      setTimeout(updateAccessPoints, 2000);
    }
    return res;
  }

  onMount(async () => {
    try {
      await updateAccessPoints();
    } catch (err) {
      console.log(err);
    }
  });
</script>

<div class="container main-container">
  <div class="row">
    <div class="column text-center">
      <h3>Set up your card's WiFi</h3>
    </div>
  </div>
  <div class="row mb-2">
    <div class="column">
      <div class="card">
        {#if data.loading}
          <div class="container">
            <div class="row h-100">
              <div class="column text-center d-flex h-100">
                <div class="loader">
                  <svg
                    xmlns="http://www.w3.org/2000/svg"
                    width="36"
                    height="36"
                    viewBox="0 0 24 24"
                    style="fill: currentColor"
                    ><path
                      d="M12,22c5.421,0,10-4.579,10-10h-2c0,4.337-3.663,8-8,8s-8-3.663-8-8c0-4.336,3.663-8,8-8V2C6.579,2,2,6.58,2,12 C2,17.421,6.579,22,12,22z"
                    /></svg
                  >
                </div>
              </div>
            </div>
          </div>
        {:else if !data.connectStatus.sent}
          {#if !data.selection.selected}
            <SelectScan
              access_points={data.access_points}
              on:refresh={refresh}
              on:select={selectAccessPoint}
            />
          {:else}
            <Connect
              ssid={data.selection.ssid}
              direct_connect={data.selection.direct_connect}
              on:back={clearSelection}
              on:success={setConnectSuccess}
              on:error={setConnectError}
            />
          {/if}
        {:else}
          <Status
            ssid={data.selection.ssid}
            success={data.connectStatus.success}
            on:back={clearConnectionStatus}
          />
        {/if}
      </div>
    </div>
  </div>
  <div class="row">
    <div class="column text-sm text-muted">
      <p class="text-center">
        <a class="link" href="https://cards.vgood.science" target="_blank"
          >What is this thing?</a
        >
      </p>
    </div>
  </div>
  {#if data.id}
    <div class="id">[{data.id}]</div>
  {/if}
</div>

<style type="text/scss" global>
  $black: #555555;
  $white: #e3e3e3;

  $color-primary: $black;
  $color-secondary: $black; //#202327;
  $color-muted: #626770;
  $color-quinary: #869ab8;

  $loader-color: $white;
  $loader-size: 8px;
  $loader-height: 4px;
  $loader-border-size: 4px;
  $loader-gap: 6px;
  $loader-animation-duration: 1s;

  @import "../node_modules/milligram/src/milligram.sass";
  @import "../node_modules/spinthatshit/src/variables.scss";
  @import "../node_modules/spinthatshit/src/animations.scss";
  @import "../node_modules/spinthatshit/src/loaders/_loader10.scss";

  html,
  body {
    margin: 0;
    font-family: "Mont", system-ui, "Helvetica Neue", "Helvetica", "Arial",
      sans-serif;
    background: $white;
  }

  .id {
    position: fixed;
    bottom: 0;
    left: 0;
    padding: 1rem;
    font-weight: 500;
    letter-spacing: 0.1rem;
  }

  .link {
    text-decoration: underline;
  }

  .text-muted {
    color: $color-muted !important;
  }

  .text-center {
    text-align: center;
  }

  .w-100 {
    width: 100%;
  }

  .h-100 {
    height: 100%;
  }

  .d-flex {
    display: flex !important;
  }

  .flex-columns {
    flex-direction: column;
  }

  .flex-rows {
    flex-direction: row;
  }

  .mb-2 {
    margin-bottom: 2rem;
  }

  .pr-1 {
    padding-right: 1rem;
  }

  .w-auto {
    width: auto !important;
  }

  .text-sm {
    font-size: 12px;
  }

  .clickable-row {
    padding: 1rem 0rem;
    border-bottom: 0.1rem solid lighten($black, 50%);
    transition: background-color 0.5s cubic-bezier(0.215, 0.61, 0.355, 1),
      box-shadow 0.5s cubic-bezier(0.215, 0.61, 0.355, 1);
    border-radius: 0.5rem;
    cursor: pointer;
    word-break: break-word;

    &:hover {
      box-shadow: rgba(99, 99, 99, 0.1) 0px 2px 8px 2px;
    }
  }

  input[type="text"],
  input[type="password"] {
    padding: 2.5rem 2rem !important;
    box-shadow: rgba(0, 0, 0, 0.1) 0px 3px 6px 1px inset,
      #e3e3e3 0px 0px 6px 6px inset;
    border: none;
    font-size: 16px !important;
  }

  input:disabled {
    background-color: rgba(47, 63, 82, 0.05);
    border: none;
  }

  .logo {
    margin: 5rem 1rem;
    fill: currentColor;
  }

  .main-container {
    max-width: 42rem !important;
    margin: 3rem auto;
  }

  .card {
    display: flex;
    position: relative;
    padding: 1rem;
    border-radius: 1rem;
    width: 100%;
    box-shadow: rgba(149, 157, 165, 0.15) 0px 8px 24px;
    min-height: 290px;

    .container {
      padding: 1rem 2rem;
    }
  }

  .button {
    border-radius: 0.5rem;
    transition: background-color 0.5s linear;
    box-shadow: rgba(149, 157, 165, 0.15) 0px 8px 24px !important;
    &[disabled] {
      opacity: 0.9;
    }
  }

  .btn-light {
    background-color: rgb(113, 121, 129) !important;
    border-color: rgb(113, 121, 129) !important;
  }

  .loader {
    margin: auto;
    svg {
      animation-name: spin;
      animation-duration: 1500ms;
      animation-iteration-count: infinite;
      animation-timing-function: linear;
    }
  }

  @keyframes spin {
    from {
      transform: rotate(0deg);
    }
    to {
      transform: rotate(360deg);
    }
  }

  .btn-loader {
    margin: auto;
    @include loader10;
  }
</style>
