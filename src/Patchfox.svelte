<script>
  import { onMount, onDestroy } from "svelte";
  import {
    connected,
    route,
    navigate,
    currentView,
    connect,
    reconnect
  } from "./utils.js";
  import Navigation from "./Navigation.svelte";

  window.ssb = false;

  let interval;

  onMount(() => {
    connect();

    interval = setInterval(() => {
      hermiebox.sbot.whoami((err, v) => {
        if (err) {
          console.error("can't call whoami", err);
          reconnect().catch(n => {
            console.error("can't reconnect")
            clearInterval(interval)
            navigate("/error", {error: n})
          });
        }
      });
    }, 5000);
  });

  onDestroy(() => clearInterval(interval));

  const popState = event => {
    if (event.state !== null) {
      console.dir("pop", event.state);
      let { location, data } = event.state;
      route.set({ location, data });
    }
  };

  const handleUncaughtException = event => {
    console.error("Uncaught exception", event);
    navigate("/error", { error: event.message });
  };

  const hashChange = event => {
    console.dir("hash change", event);
  };
</script>

<svelte:window
  on:popstate={popState}
  on:error={handleUncaughtException}
  on:hashchange={hashChange} />
<div class="container bg-gray">
  <Navigation />
  <svelte:component this={$currentView} />
</div>
