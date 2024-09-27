<script lang="ts">
	import { onDestroy, onMount } from "svelte";
	import { persisted } from "svelte-persisted-store";
	import { get } from "svelte/store";

    // Setup
    const tabs = persisted<{ [key: string]: number }>("tabs", {});
    const bc = new BroadcastChannel("main");

    const counter = persisted("counter", 0);

    const tabLifetime = 1000; //ms

    // Create unique tab id
    let id = parseInt(Math.random().toString().split(".")[1].split("").reverse().join(""));

    let x = 0;
    let y = 0;

    let isTop = true;

    // Main loop
    let start: number | null;
    function step(timestamp: number) {
        if (start == null) start = timestamp;
        const elapsed = timestamp - start;
        const now = Date.now();

        //----- Global state update -----//
        isTop = false;

        // Keep alive
        $tabs[id] = now;

        // Find master
        let keys = Object.keys($tabs).filter(k => now - $tabs[k] < tabLifetime).map(Number);
        // console.log("Id:", id);
        // console.log("Tabs:", $tabs);
        // console.log("Keys:", keys);
        const master = Math.max(...keys);
        console.info("Current master:", keys, "\n\n", master);
        if (master == id) {
            isTop = true;
            $counter++;

            // Purge dead tabs (older than 100ms)
            $tabs = Object.fromEntries(Object.entries($tabs).filter(([k,v]) => now - v < tabLifetime))

            x = Math.random() * 300;
            y = Math.random() * 100;
        }

        //----- Local state update -----//
        const targetX = 200 - window.screenX;
        const targetY = 200 - window.screenY;

        // x = x + (targetX - x) * 0.1;
        // y = y + (targetY - y) * 0.1;

        requestAnimationFrame(step);
    }

    onMount(() => {
        console.log("ID:", id);
        $tabs = { ...$tabs, [id]: Date.now() };
        // bc.onmessage = (e) => {
        //     console.log("RECEIVED:\n", e.data);
        //     const [command, data] = e.data.split(":");
        //     switch (command) {
        //         // case "tabQuit": refreshMaster(); break;
        //         // case "tabJoin": refreshMaster(); break;
        //     }
        // }

        // bc.postMessage("tabJoin:" + id);

        // setInterval(() => {
        //     console.log("Master:", $masterTab);
        // }, 1000);

        requestAnimationFrame(step);
    });

    // function onUnload() {
    //     if ($masterTab === id) { $masterTab = 0; }
    //     // Stop listening to messages
    //     bc.onmessage = null;

    //     bc.postMessage("tabQuit:" + id);
    //     bc.close();
    // };

    onDestroy(() => {
        bc.close();
    });

</script>

<div
    class="bb"
    style:top="{y}px"
    style:left="{x}px"
    style:border-color="{isTop ? 'green' : 'red'}"
/>
{id}
<br />
{$counter}

<style>
    .bb {
        position: absolute;

        width: 20px;
        height: 20px;
        border: 1px solid green;
    }
</style>